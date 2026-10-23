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
