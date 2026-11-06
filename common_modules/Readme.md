**0001-qos_auth-include-linux-compat.h-for-compat_ptr.patch**

Модуль `qos_auth/auth_ctl` (`auth_ctrl.c`, `qos_ctrl.c`) использует `compat_ptr` и `compat_uptr_t` в обработке `ioctl` от 32-битных программ, но не подключает `<linux/compat.h>`. На arm64 заголовок попадает косвенно через другие заголовки, на x86_64 нет, и сборка ядра с `CONFIG_AUTHORITY_CTRL` завершается ошибкой `call to undeclared function 'compat_ptr'`. Патч подключает заголовок явно. Не зависит от архитектуры.

**Важно:** патч применяется к репозиторию `kernel/linux/common_modules`. Правится исходный модуль, а не копия в `out/kernel/src_tmp`: сборка ядра копирует модуль в дерево ядра заново при каждом запуске.

Применение:
```
cd /mnt/ohos/OpenHarmony7/kernel/linux/common_modules
P=/mnt/ohos/OpenHarmony7/oneir_patches/common_modules/0001-qos_auth-include-linux-compat.h-for-compat_ptr.patch
git apply --check $P
git apply $P
```

Откатить:
```
cd /mnt/ohos/OpenHarmony7/kernel/linux/common_modules
P=/mnt/ohos/OpenHarmony7/oneir_patches/common_modules/0001-qos_auth-include-linux-compat.h-for-compat_ptr.patch
git apply -R --check $P
git apply -R $P
```

После применения или отката нужно пересобрать ядро.

---

**0002-proc-include-linux-compat.h-for-sched_qos_ctrl.patch**

Обработка `ioctl` узла `/proc/<pid>/sched_qos_ctrl`, добавленная OpenHarmony в `fs/proc/base.c` (включается
`CONFIG_QOS_CTRL`), использует `compat_ptr` и `compat_uptr_t` для вызовов от 32-битных программ, но файл не
подключает `<linux/compat.h>`. На arm64 заголовок попадает косвенно, на x86_64 нет, и сборка ядра с
`CONFIG_QOS_CTRL` завершается ошибкой `call to undeclared function 'compat_ptr'`. Патч подключает заголовок явно.
Не зависит от архитектуры.

**Важно:** патч применяется к репозиторию ядра `kernel/linux/linux-6.6`.

Применение:
```
cd /mnt/ohos/OpenHarmony7/kernel/linux/linux-6.6
P=/mnt/ohos/OpenHarmony7/oneir_patches/common_modules/0002-proc-include-linux-compat.h-for-sched_qos_ctrl.patch
git apply --check $P
git apply $P
```

Откатить:
```
cd /mnt/ohos/OpenHarmony7/kernel/linux/linux-6.6
P=/mnt/ohos/OpenHarmony7/oneir_patches/common_modules/0002-proc-include-linux-compat.h-for-sched_qos_ctrl.patch
git apply -R --check $P
git apply -R $P
```

После применения или отката нужно пересобрать ядро.
