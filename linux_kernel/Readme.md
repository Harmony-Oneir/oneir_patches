**0001-x86-mm-pat-fix-track_pfn_copy-after-CVE-backport.patch**:
В ветку OpenHarmony-7.0-Release ядра linux-6.6 портировали исправление уязвимости CVE-2025-22090. Она касается учёта типа кэширования (PAT) для памяти устройств при неудачном fork(). Общий код ядра перевели на новую сигнатуру track_pfn_copy(dst, src, &pfn) и добавили вызов untrack_pfn_copy(), а x86-реализацию в memtype.c обновить забыли. Из-за этого сборка падает с ошибкой conflicting types for 'track_pfn_copy'. Патч приводит x86-часть к виду из upstream linux-6.6.y.

Применение (Из корня дерева ядра):
```
cd /mnt/ohos/OpenHarmony7/kernel/linux/linux-6.6
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0001-x86-mm-pat-fix-track_pfn_copy-after-CVE-backport.patch
git apply --check $P
git apply $P
```

Откатить:

```
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0001-x86-mm-pat-fix-track_pfn_copy-after-CVE-backport.patch
git apply -R $P
```

**0002-x86-add-missing-amd-fch-h.patch**:
В ветке OpenHarmony-7.0-Release ядра linux-6.6 в файл `arch/x86/kernel/cpu/amd.c` был перенесён патч «x86/CPU/AMD: вывод причины последнего сброса». Этот файл подключает заголовок `<linux/platform_data/x86/amd-fch.h>`, однако сам заголовок добавлен не был. Из-за этого сборка для x86 завершается ошибкой. Этот патч добавляет заголовок с определениями регистров управления питанием FCH (AMD Fusion Controller Hub), как в основной ветке проекта. Файл `amd.c` использует `FCH_PM_BASE` и `FCH_PM_S5_RESET_STATUS`, чтобы определить причину последнего сброса и вывести её в dmesg при загрузке. На работу системы это не влияет.
Проблема касается только x86: файл `amd.c` не компилируется для архитектур ARM, ARM64 и RISC-V.

Применить (из корня дерева ядра):

```
cd /mnt/ohos/OpenHarmony7/kernel/linux/linux-6.6
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0002-x86-add-missing-amd-fch-h.patch
git apply --check $P
git apply $P
```

Откатить:

```
cd /mnt/ohos/OpenHarmony7/kernel/linux/linux-6.6
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0002-x86-add-missing-amd-fch-h.patch
git apply -R --check $P
git apply -R $P
```

**0003-binfmt_misc-use-6.6-allow_write_access-API.patch**:
Перенесённое исправление в `fs/binfmt_misc.c` вызывает функцию `exe_file_allow_write_access()`, которая существует только в ядрах версии 6.14 и новее — в этих версиях она заменила `allow_write_access()`.

В linux-6.6 по-прежнему используется `allow_write_access()`, поэтому при включённом `CONFIG_BINFMT_MISC` сборка завершается ошибкой. Следует использовать название функции, принятое в версии 6.6, как это сделано в основной ветке linux-6.6.y.

Проблема не зависит от архитектуры: она затрагивает любую конфигурацию с включённым `CONFIG_BINFMT_MISC`.

Применение (из корня дерева ядра):
```
cd /mnt/ohos/OpenHarmony7/kernel/linux/linux-6.6
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0003-binfmt_misc-use-6.6-allow_write_access-API.patch
git apply --check $P
git apply $P
```

Откатить:
```
cd /mnt/ohos/OpenHarmony7/kernel/linux/linux-6.6
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0003-binfmt_misc-use-6.6-allow_write_access-API.patch
git apply -R --check $P
git apply -R $P
```


**0004-kernel.mk-apply-missing-common-modules.patch**:
Патч `rk3568_patch/kernel.patch` прописывает в `Makefile` и `Kconfig` каталоги `drivers/auth_ctl`, `drivers/tzdriver` и `net/newip`. Сами каталоги в дерево ядра не входят: их подкладывают скрипты `apply_qos_auth.sh`, `apply_tzdriver.sh` и `apply_newip.sh` из `kernel/linux/common_modules`, а `kernel.mk` эти скрипты не вызывал. Из-за этого сборка падает уже на `make distclean`. Патч добавляет вызов скриптов по тому же шаблону, что у остальных модулей `common_modules`. Сами функции включаются или выключаются опциями в defconfig; tzdriver (драйвер ARM TrustZone) на x86 остаётся выключенным. Проблема не зависит от архитектуры.

Важно: патч применяется к репозиторию `kernel/linux/build`, а не к дереву ядра.

Применение:
```
cd /mnt/ohos/OpenHarmony7/kernel/linux/build
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0004-kernel.mk-apply-missing-common-modules.patch
git apply --check $P
git apply $P
```

Откатить:
```
cd /mnt/ohos/OpenHarmony7/kernel/linux/build
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0004-kernel.mk-apply-missing-common-modules.patch
git apply -R --check $P
git apply -R $P
```

**0005-sched-walt-use-arch_scale_cpu_capacity.patch**:
Планировщик WALT OpenHarmony (`kernel/sched/walt.c`) определяет мощность кластера CPU через `topology_get_cpu_scale()`. Эта функция читает переменную cpu_scale, которая есть только на архитектурах с `GENERIC_ARCH_TOPOLOGY` (arm, arm64, riscv). На x86 её нет, и линковка падала с `undefined reference to 'cpu_scale'`. Патч заменяет вызов на общий для всех архитектур `arch_scale_cpu_capacity()`. На ARM поведение не меняется, потому что там это та же функция. На x86 все ядра получают одинаковую мощность `1024`, что соответствует симметричным процессорам. Затрагивает все архитектуры без `GENERIC_ARCH_TOPOLOGY`, включая x86.

Применить:

```
cd /mnt/ohos/OpenHarmony7/kernel/linux/linux-6.6
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0005-sched-walt-use-arch_scale_cpu_capacity.patch
git apply --check $P
git apply $P
```

Откатить:

```
cd /mnt/ohos/OpenHarmony7/kernel/linux/linux-6.6
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0005-sched-walt-use-arch_scale_cpu_capacity.patch
git apply -R --check $P
git apply -R $P
```
