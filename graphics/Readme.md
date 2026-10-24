**mesa_x86_64_no_neon.patch**:
исправляет сборку Mesa из дерева OpenHarmony под x86_64. В `src/util/u_mm.c` функция `memcpy_neon()` и подключение заголовка `arm_neon.h` включаются по условию `DETECT_OS_OHOS` без проверки архитектуры, поэтому под x86_64 компилятор получает ARM NEON-интринсики и сборка падает. Патч добавляет к условию проверку `defined(__aarch64__) || defined(__ARM_NEON)` в двух местах. На ARM поведение не меняется, а на x86_64 используется ветка `#else` с обычным `memcpy`.
Применить (из корня дерева OpenHarmony):
* `P=/mnt/ohos/OpenHarmony7/oneir_patches/mesa_x86_64_no_neon.patch`
* `git -C third_party/mesa3d apply --check "$P"`
* `git -C third_party/mesa3d apply "$P"`

Откатить:
* `git -C third_party/mesa3d restore -- src/util/u_mm.c`


---

**mesa_libcxx_algorithm.patch**:
исправляет сборку компилятора шейдеров Intel из Mesa с библиотекой libc++ из OpenHarmony clang. В `src/intel/compiler/brw_compile_fs.cpp` используется `std::min`, но заголовок `<algorithm>`, в котором он объявлен, явно не подключается. С `libstdc++` он приходит транзитивно через другие заголовки, а `libc++` так не делает, поэтому сборка падает с ошибкой `no member named 'min' in namespace 'std'`. Патч добавляет в начало файла явный `#include <algorithm>`. На поведение кода это не влияет и сборку с другими библиотеками C++ не ломает.
Применить (из корня дерева OpenHarmony):
* `P=/mnt/ohos/OpenHarmony7/oneir_patches/mesa_libcxx_algorithm.patch`
* `git -C third_party/mesa3d apply --check "$P"`
* `git -C third_party/mesa3d apply "$P"`

Откатить:
* `git -C third_party/mesa3d restore -- src/intel/compiler/brw_compile_fs.cpp`

---

**mesa_ohos_no_wsi_display.patch**:
исключает из сборки Mesa под OpenHarmony поддержку `VK_KHR_display`, то есть вывода изображения из Vulkan напрямую через KMS, минуя оконную систему. Реализация в `src/vulkan/wsi/wsi_common_display.c` использует `pthread_cancel` и `pthread_setcanceltype`, которых нет в musl OpenHarmony, поэтому линковка `libvulkan_intel.so` падает с неопределёнными символами. В OpenHarmony это расширение не нужно: изображение выводит композитор, а поверхности Vulkan создаются через `VK_OHOS_*`. Патч дополняет условие `not with_platform_android` проверкой `not with_platform_ohos` в двух местах: в `src/vulkan/wsi/meson.build` (сборка файла) и в `src/vulkan/meson.build` (`define VK_USE_PLATFORM_DISPLAY_KHR`). Так OpenHarmony обрабатывается так же, как Android, а сборки под другие платформы не меняются.
Применить (из корня дерева OpenHarmony):
* `P=/mnt/ohos/OpenHarmony7/oneir_patches/mesa_ohos_no_wsi_display.patch`
* `git -C third_party/mesa3d apply --check "$P"`
* `git -C third_party/mesa3d apply "$P"`

Откатить:
* `git -C third_party/mesa3d restore -- src/vulkan/wsi/meson.build src/vulkan/meson.build`

---

**mesa_ohos_vulkan_kms.patch**:
возвращает поддержку DRM/KMS для сборок Mesa под OpenHarmony, в которых собираются Vulkan-драйверы. В верхнем `meson.build` при включённой платформе OHOS (-`Dplatforms=ohos`) Mesa безусловно выставляет `system_has_kms_drm = false`: это рассчитано на штатную EGL-сборку OpenHarmony поверх Zink, где прямая работа с DRM не нужна. Из-за этого anv теряет `libdrm` и не собирается (`xf86drm.h` не найден). Патч оставляет отключение DRM только для сборок без Vulkan-драйверов (`_vulkan_drivers` пуст), так что штатная EGL-сборка работает как раньше, а anv получает `libdrm`. Define `VK_USE_PLATFORM_OHOS=1` сохраняется в обоих случаях. Патч применяется вместе с `mesa_ohos_no_wsi_display.patch`: при возвращённом DRM тот исключает из сборки `wsi_common_display.c`, несовместимый с musl OpenHarmony.

Применить (из корня дерева OpenHarmony):
* `P=/mnt/ohos/OpenHarmony7/oneir_patches/graphics/mesa_ohos_vulkan_kms.patch`
* `git -C third_party/mesa3d apply --check "$P"`
* `git -C third_party/mesa3d apply "$P"`
Откатить:
* `git -C third_party/mesa3d restore -- meson.build`

---

**mesa_ohos_vk_registry.patch**:
добавляет в Mesa описание расширений VK_OHOS_native_buffer и VK_OHOS_external_memory, необходимых Render Service в Vulkan-режиме. В реестре Mesa (src/vulkan/registry/vk.xml) из OpenHarmony было только VK_OHOS_surface, поэтому генераторы Mesa не знали этих расширений, и драйвер не мог объявить их поддержку. Патч переносит из third_party/vulkan-headers описания структур, команд и обоих расширений. Значения VkStructureType (1000453001…, 1000452000…) и биты типа внешней памяти (0x2000, 0x8000) приведены в соответствие с вручную заданными значениями в vulkan_core.h, с которым собраны RS и слой swapchain. Структура VkPhysicalDevicePresentationPropertiesOHOS отвязана от VkPhysicalDeviceProperties2, чтобы не конфликтовать с Android-аналогом в генераторе свойств. Также обновлены заголовки include/vulkan/vulkan_core.h и vulkan_ohos.h, и добавлен vk_ohos_native_buffer.h. Сама реализация расширений в драйвере в этот патч не входит.

Применить (из корня дерева OpenHarmony):
* `P=/mnt/ohos/OpenHarmony7/oneir_patches/graphics/mesa_ohos_vk_registry.patch`
* `git -C third_party/mesa3d apply --check "$P"`
* `git -C third_party/mesa3d apply "$P"`

Откатить:
* `git -C third_party/mesa3d restore -- src/vulkan/registry/vk.xml include/vulkan/vulkan_core.h include/vulkan/vulkan_ohos.h`
* `rm -f third_party/mesa3d/include/vulkan/vk_ohos_native_buffer.h`
