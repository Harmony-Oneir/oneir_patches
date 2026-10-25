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
