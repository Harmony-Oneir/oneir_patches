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
