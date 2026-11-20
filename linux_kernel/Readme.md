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
---

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

---

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

---

**0004-kernel.mk-apply-missing-common-modules.patch** [fix(rtg_test): fix ioctl magic number sign mismatch on x86](https://github.com/openharmony/kernel_linux_build/pull/1):
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

---

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

---

**0006-test-rtg-use-unsigned-char-for-ioctl-magic.patch** [fix(rtg_test): fix ioctl magic number sign mismatch on x86](https://gitcode.com/openharmony/kernel_linux_build/pull/239):
Тест `rtg_test` объявляет магическое число ioctl как `const char RTG_SCHED_IPC_MAGIC = 0xAB`. Знаковость типа `char` зависит от архитектуры: на arm/arm64 он беззнаковый, а на x86 знаковый, и значение `0xAB` превращается в `-85`. При подстановке в `_IOWR()` происходит сдвиг отрицательного числа, и сборка падает с ошибкой `-Werror,-Wshift-negative-value`. Даже без `-Werror` номера ioctl на x86 не совпали бы с номерами драйвера RTG в ядре, и все вызовы в тесте завершались бы ошибкой. Патч объявляет константу как `unsigned char`, чтобы значение было одинаковым на всех архитектурах. На arm/arm64 поведение не меняется.
**Важно:** патч применяется к репозиторию `kernel/linux/build`. 

Применение:
```
cd /mnt/ohos/OpenHarmony7/kernel/linux/build
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0006-test-rtg-use-unsigned-char-for-ioctl-magic.patch
git apply --check $P
git apply $P
```

Откатить:
```
cd /mnt/ohos/OpenHarmony7/kernel/linux/build
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0006-test-rtg-use-unsigned-char-for-ioctl-magic.patch
git apply -R --check $P
git apply -R $P
```

---

**0007-kernel.mk-build-and-install-kernel-modules.patch** [Add module build and installation steps to kernel.mk](https://github.com/openharmony/kernel_linux_build/commit/9181c75b27641db8d794da64dde4abc27e9484f5)

Сборка ядра OpenHarmony (`kernel.mk`) вызывает только цель `$(KERNEL_IMAGE)` (`bzImage`), поэтому драйверы, отмеченные в конфигурации как модули (`=m`), не собираются: `modules_prepare` лишь готовит дерево к сборке модулей, но сами модули не создаёт. Встроенный (`=y`) драйвер для этого не подходит, если ему нужна прошивка с корневой файловой системы: он запрашивает её при запуске ядра, до монтирования корня. Так устроены, например, драйверы Wi-Fi (`ath10k` и др.).

Патч добавляет после сборки образа ядра два шага: `make modules` собирает модули, `make modules_install` устанавливает их без отладочных символов (`INSTALL_MOD_STRIP=1`) в `$(KERNEL_SRC_TMP_PATH)/../modules/lib/modules/<версия>/` вместе с `modules.dep` и `modules.alias`, которые нужны `modprobe` для зависимостей и подбора драйвера по железу. Если в конфигурации нет модулей, шаги ничего не делают. Не зависит от архитектуры.

**Важно:** патч применяется к репозиторию `kernel/linux/build`.

Применение:
```
cd /mnt/ohos/OpenHarmony7/kernel/linux/build
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0007-kernel.mk-build-and-install-kernel-modules.patch
git apply --check $P
git apply $P
```

Откатить:
```
cd /mnt/ohos/OpenHarmony7/kernel/linux/build
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0007-kernel.mk-build-and-install-kernel-modules.patch
git apply -R --check $P
git apply -R $P
```

Проверка после сборки ядра:
```
find /mnt/ohos/OpenHarmony7/out/kernel/OBJ/modules -name "*.ko" | head
```

---

**0008-blackbox-add-x86-system-adapter.patch**

Ядро blackbox OpenHarmony (`drivers/staging/blackbox`) — только реестр модулей: само на аварии не реагирует. Панику ловит системный адаптер платформы, а данные до перезагрузки хранит pstore. В OpenHarmony такой адаптер есть только для Rockchip: каталог `blackbox/rockchip` приходит из `kernel/linux/patches/linux-6.6/rk3568_patch/kernel.patch` и собирается только при `ARCH_ROCKCHIP`. В самом `kernel_linux_6.6` его нет.

Патч добавляет `drivers/staging/blackbox/x86/` — копию `rockchip/system_adapter.c` (вариант с pstore_ram) со своими `Kconfig` (`CONFIG_BLACKBOX_X86`) и `Makefile`. Код адаптера платформенно-независим, кроме сброса кэша данных перед перезагрузкой: `dcache_clean_inval_poc()`, `__flush_dcache_area()` и `__cpuc_flush_dcache_area()` есть только на ARM, на x86 используется `clflush_cache_range()`. Вариант «через память» (`system_adapter_by_memory.c`) не перенесён: он требует тёплой перезагрузки без стирания ОЗУ, на ноутбуке это ненадёжно.

Хуки стоят в начале `blackbox/Kconfig` и `blackbox/Makefile`, поэтому патч накладывается и на чистое ядро, и поверх `rk3568_patch` (тот дописывает свои строки в конец этих файлов). Порядок компоновки не важен: у ядра blackbox статический список модулей и `core_initcall`, у адаптера — `postcore_initcall`.

Нужны опции: `CONFIG_PSTORE`, `CONFIG_PSTORE_RAM`, `CONFIG_PSTORE_BLACKBOX`, `CONFIG_BLACKBOX_STORAGE_BY_PSTORE_RAM`, `CONFIG_BLACKBOX_X86`, а также пути логов `CONFIG_BLACKBOX_LOG_ROOT_PATH` и `CONFIG_BLACKBOX_LOG_PART_REPRESENTATIVE`. На стенде пока не проверено.

Применить (из корня дерева ядра):

```
cd /mnt/ohos/OpenHarmont7/kernel/linux/linux-6.6
P=/mnt/ohos/OpenHarmont7/oneir/oneir_patches/linux_kernel/0008-blackbox-add-x86-system-adapter.patch
git apply --check $P
git apply $P
```

Откатить:

```
cd /mnt/ohos/OpenHarmont7/kernel/linux/linux-6.6
P=/mnt/ohos/OpenHarmont7/oneir/oneir_patches/linux_kernel/0008-blackbox-add-x86-system-adapter.patch
git apply -R --check $P
git apply -R $P


---

**0009-blackbox-read-pstore-record-on-linux-6.6.patch**

На linux-6.6 blackbox OpenHarmony не читает запись pstore. Функция `get_log_by_pstore()` в `drivers/staging/blackbox/blackbox_storage.c` открывает `/sys/fs/pstore/blackbox-*`, пишет в лог `read ... is succ`, ставит флаг `g_blackbox_flag`, закрывает и удаляет файл, но содержимое в буфер так и не читает. Системный адаптер на 6.6 по этому флагу сообщает «сбой был», но `error_info` не заполняет. В итоге каждый лог сбоя в `/data/log/bbox` сохраняется с пустыми событием, модулем и временем, `last_kmsg` — 0 байт, а сама запись в pstore удаляется.

Запись пишет `pstore_blackbox_dump()` (`fs/pstore/platform.c`) ровно в том формате, который ждёт ветка адаптера для 5.10: заголовок `struct fault_log_info` (отметка `LOG_FLAG`, событие, модуль `PSTORE`, время, стек, длина), за ним хвост kmsg.

Патч:
- в `blackbox_storage.c` возвращает чтение записи (`kernel_read()`); флаг ставится и запись удаляется только после успешного чтения;
- в `x86/system_adapter.c` убирает ветку по версии ядра в `get_last_log_info()`, чтобы заголовок разбирался всегда.

Адаптер Rockchip (из `rk3568_patch`) содержит ту же ошибку и здесь не меняется.

**Требует `0008`.** На чистое ядро накладывать по порядку: `0008`, затем `0009`.

Применить (из корня дерева ядра):

```
cd /mnt/ohos/OpenHarmont7/kernel/linux/linux-6.6
P=/mnt/ohos/OpenHarmont7/oneir/oneir_patches/linux_kernel/0009-blackbox-read-pstore-record-on-linux-6.6.patch
git apply --check $P
git apply $P
```

Откатить:

```
cd /mnt/ohos/OpenHarmont7/kernel/linux/linux-6.6
P=/mnt/ohos/OpenHarmont7/oneir/oneir_patches/linux_kernel/0009-blackbox-read-pstore-record-on-linux-6.6.patch
git apply -R --check $P
git apply -R $P
```

Откатывать `0008` — только после отката `0009`.

После правки дерева ядра перед сборкой: `touch /mnt/ohos/OpenHarmont7/kernel/linux/linux-6.6`, иначе ninja пропустит сборку ядра.

Проверка (QEMU с ramoops, холодный старт → `reboot`):

```
dmesg | grep -i bbox                       # "read /sys/fs/pstore/blackbox-ramoops-0: N bytes", непустые event/module
ls -lR /data/log/bbox                      # каталог <время>-<номер> с непустым last_kmsg
head -c 300 /data/log/bbox/*/last_kmsg
```
