**compile_standard_whitelist_rs_samples.patch**:
вносит в систему сборки OpenHarmony (build/) три группы изменений, нужных для сборки Oneir:
- `compile_standard_whitelist.json`: в список `deps_added_external_part_module` добавлены пять примеров Render Service: hello_rosen, drawing_sample_rs, hello_vsync, hello_native_window, hello_composer. Эти примеры подключают библиотеки других компонентов (например, ipc_core) напрямую через deps, а не через external_deps, и без белого списка проверка зависимостей при сборке их отклоняет. Кроме того, в строке с dslm_sdk_small изменились только пробелы: это побочный эффект редактирования, на работу он не влияет;
- `subsystem_components_whitelist.json`: добавлен продукт product_oneir;
- `subsystem_config.json`: зарегистрирована подсистема oneir с путём oneir. Без этого компоненты oneir_display, oneir_display_composer, oneir_hdf_config и другие не распознаются системой сборки.

Применить (из корня дерева OpenHarmony):

* `P=/mnt/ohos/OpenHarmony7/oneir_patches/build_patches/compile_standard_whitelist_rs_samples.patch`
* `git -C build apply --check "$P"`
* `git -C build apply "$P"`

Откатить:

* `git -C build restore -- compile_standard_whitelist.json subsystem_components_whitelist.json subsystem_config.json`

---

**graphic_2d_native_vsync_stdint.patch**:
исправляет заголовок NDK `interfaces/inner_api/composer/native_vsync.h` в `graphic_2d`. Заголовок использует типы `int32_t` и `uint64_t`, но сам не подключает `<stdint.h>`, поэтому собирается только тогда, когда этот заголовок уже подключён раньше другим файлом. Если `native_vsync.h` включается первым (как в примере hello_vsync), компиляция падает с ошибкой unknown type name 'int32_t'. Патч добавляет `#include <stdint.h>` после защиты от повторного включения. На остальные компоненты это не влияет.

Применить (из корня дерева OpenHarmony):

* `P=/mnt/ohos/OpenHarmony7/oneir_patches/graphics/graphic_2d_native_vsync_stdint.patch`
* `git -C foundation/graphic/graphic_2d apply --check "$P"`
* `git -C foundation/graphic/graphic_2d apply "$P"`

Откатить:

* `git -C foundation/graphic/graphic_2d restore -- interfaces/inner_api/composer/native_vsync.h`

---

**build-define-HDC_NO_ACCOUNT-consistently.patch**:
В `developtools/hdc/BUILD.gn` зависимость `hdcd` от компонента учётных записей (`os_account`) подключается при условии `build_selinux && image_name == "system" && product_name != "ohos-sdk" && support_jsapi`, а макрос `HDC_NO_ACCOUNT`, отключающий использование `os_account_manager.h`, задаётся только при `!support_jsapi`. У продуктов с `build_selinux = false` и `support_jsapi = true` не выполняется ни то, ни другое, и сборка падает с ошибкой `'os_account_manager.h' file not found`. Патч вычисляет условие один раз (`hdc_with_account`) и использует его и для зависимости, и для макроса. Без `os_account` `hdcd` использует пользователя по умолчанию. Не зависит от архитектуры.

**Важно:** патч применяется к репозиторию `developtools/hdc`.

Применение:
```
cd /mnt/ohos/OpenHarmony7/developtools/hdc
P=/mnt/ohos/OpenHarmony7/oneir_patches/hdc/build-define-HDC_NO_ACCOUNT-consistently.patch
git apply --check $P
git apply $P
```

Откатить:
```
cd /mnt/ohos/OpenHarmony7/developtools/hdc
P=/mnt/ohos/OpenHarmony7/oneir_patches/hdc/build-define-HDC_NO_ACCOUNT-consistently.patch
git apply -R --check $P
git apply -R $P
```
---
**hostapd-guard-CheckApBand-with-the-same-condition-as-its-caller.patch**:
функция `CheckApBand()` в `hostapd/ap_config_file.c` определена без условий, а её единственный вызов компилируется только при `CONFIG_OPEN_HARMONY_PATCH` и `CONFIG_IEEE80211AC` или `CONFIG_IEEE80211AX`. В конфигурациях без поддержки 802.11ac/ax функция не используется, и сборка падает с `-Werror,-Wunused-function`. Патч ставит определение функции под то же условие, что и вызов. Не зависит от архитектуры.

**Важно:** патч применяется к репозиторию `third_party/wpa_supplicant`.

Применение: `cd /mnt/ohos/OpenHarmont7/third_party/wpa_supplicant && git apply $P`
Откат: `cd /mnt/ohos/OpenHarmont7/third_party/wpa_supplicant && git apply -R $P`
