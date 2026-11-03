**0001-ueventd-load-kernel-modules-by-MODALIAS.patch**

В OpenHarmony нет механизма загрузки модулей ядра под найденное железо: `ueventd` не читает поле `MODALIAS` из событий устройств, а init умеет только загружать модуль по явному пути (`insmod` в конфиге платы). Для плат с известным железом этого достаточно, но универсальный образ (например, для ПК с произвольным Wi-Fi-адаптером) так не сделать.

Патч добавляет в `ueventd` загрузку модулей по `MODALIAS`: при событии подключения устройства `ueventd` находит подходящие модули в `/lib/modules/<версия>/modules.alias` и загружает их вместе с зависимостями из `modules.dep` системным вызовом `finit_module`. Поскольку `ueventd` при старте повторяет события для уже найденных устройств, это работает и для устройств, подключённых до загрузки, и для подключаемых позже. Загрузка выполняется в отдельном процессе, чтобы медленные модули (например, ждущие прошивку) не задерживали обработку событий. `ueventd` получает привилегию `CAP_SYS_MODULE`. Загрузчик включается только в стандартную сборку. Если файла `modules.alias` нет, автозагрузка отключается с предупреждением в журнале.

Требует модулей ядра в образе (`/lib/modules/<версия>/` с `modules.dep` и `modules.alias`), см. патч `kernel_build/0006`.

**Важно:** патч применяется к репозиторию `base/startup/init`.

Применение:
```
cd /mnt/ohos/OpenHarmony7/base/startup/init
P=/mnt/ohos/OpenHarmony7/oneir_patches/init/0001-ueventd-load-kernel-modules-by-MODALIAS.patch
git apply --check $P
git apply $P
```

Откатить:
```
cd /mnt/ohos/OpenHarmony7/base/startup/init
P=/mnt/ohos/OpenHarmony7/oneir_patches/init/0001-ueventd-load-kernel-modules-by-MODALIAS.patch
git apply -R --check $P
git apply -R $P
```

Проверка после загрузки системы:
```
dmesg | grep -i "Module autoload"
lsmod
```
