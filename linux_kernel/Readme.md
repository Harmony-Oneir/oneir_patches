**0001-x86-mm-pat-fix-track_pfn_copy-after-CVE-backport.patch**:
В ветку OpenHarmony-7.0-Release ядра linux-6.6 портировали исправление уязвимости CVE-2025-22090. Она касается учёта типа кэширования (PAT) для памяти устройств при неудачном fork(). Общий код ядра перевели на новую сигнатуру track_pfn_copy(dst, src, &pfn) и добавили вызов untrack_pfn_copy(), а x86-реализацию в memtype.c обновить забыли. Из-за этого сборка падает с ошибкой conflicting types for 'track_pfn_copy'. Патч приводит x86-часть к виду из upstream linux-6.6.y.

Применение (Из корня дерева ядра):
```
cd /mnt/ohos/OpenHarmont7/kernel/linux/linux-6.6
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0001-x86-mm-pat-fix-track_pfn_copy-after-CVE-backport.patch
git apply --check $P
git apply $P
```

Откатить:

```
P=/mnt/ohos/OpenHarmony7/oneir_patches/linux_kernel/0001-x86-mm-pat-fix-track_pfn_copy-after-CVE-backport.patch
git apply -R $P
```
