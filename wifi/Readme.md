**wifi-register-supplicant-event-callback-for-HDI-wpa.patch**:
В `wifi_supplicant_hal_interface.cpp` функции `RegisterSupplicantEventCallback()` и `UnRegisterSupplicantEventCallback()` сохраняют обработчик событий `wpa_supplicant` только при `HDI_CHIP_INTERFACE_SUPPORT` (включён HDI-сервис чипа), а в остальных случаях возвращают ошибку. Однако сохранённый обработчик используется и при работе через HDI `wpa` (`wifi_hdi_wpa_callback.cpp`): через него сканирование получает от `wpa_supplicant` событие о завершении. В конфигурациях с HDI `wpa`, но без HDI чипа (`wifi_feature_with_hdi_chip_supported = false`: стандартные драйверы Linux через `nl80211`, как в конфигурации x86-эмулятора) сервис сканирования не инициализируется:

```
ScanMonitor: RegisterSupplicantEventCallback failed.
ScanService: InitScanMonitor failed.
```

Патч сохраняет обработчик, если включён любой из двух путей (`HDI_CHIP_INTERFACE_SUPPORT` или `HDI_WPA_INTERFACE_SUPPORT`), что согласуется с остальными функциями файла. При включённом сервисе чипа поведение не меняется. Не зависит от архитектуры.

**Важно:** патч применяется к репозиторию `foundation/communication/wifi`.

Применение:
```
cd /mnt/ohos/OpenHarmony7/foundation/communication/wifi
P=/mnt/ohos/OpenHarmony7/oneir_patches/wifi/wifi-register-supplicant-event-callback-for-HDI-wpa.patch
git apply --check $P
git apply $P
```

Откатить:
```
cd /mnt/ohos/OpenHarmony7/foundation/communication/wifi
P=/mnt/ohos/OpenHarmony7/oneir_patches/wifi/wifi-register-supplicant-event-callback-for-HDI-wpa.patch
git apply -R --check $P
git apply -R $P
```

После применения или отката пересобрать сетевой стек и образ:
```
cd /mnt/ohos/OpenHarmony7
./make.sh netstack
```
