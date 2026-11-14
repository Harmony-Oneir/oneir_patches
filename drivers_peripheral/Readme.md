
**0001-wlan-chip-country-code-mode.patch**
Добавляет флаг продукта `drivers_peripheral_wlan_feature_country_code_mode`, который задаёт, как служба чипа Wi-Fi (`chip_interface_service`, библиотека `libwifi_hal_default.z.so`, процесс `wifi_host`) устанавливает код страны.
| Значение | Поведение `WifiSetCountryCode` |
|---|---|
| `"vendor"` (по умолчанию) | только вендорская команда `NL80211_CMD_VENDOR` (`HAL_OUI`, `WIFI_SUBCMD_SET_COUNTRY_CODE`) — поведение OpenHarmony без изменений |
| `"nl80211"` | только стандартная `NL80211_CMD_REQ_SET_REG` с `NL80211_ATTR_REG_ALPHA2` (то же, что `iw reg set`) |
| `"auto"` | сначала вендорская команда, при ошибке — стандартная |

Для oneir используется `"auto"`.

### Зачем

Перед запуском точки доступа `wifi_manager` задаёт код страны (`ApStartedState::SetCountry()` → `WifiApHalInterface::SetWifiCountryCode` → `HalDeviceManager` → служба чипа). Если установка не удалась, запуск прерывается:

```
E chip_iface_proxy: SetCountryCode_ failed, error code is -1
E WifiApStartedState: set countryCode=CN failed
I WifiApService: Ap disabled, set softap toggled false   → состояние 4
```

Штатный `wifi_hal_default` задаёт код страны только вендорской командой, которую понимают драйверы конкретного производителя. Стандартные драйверы Linux (ath10k и другие драйверы cfg80211) её отклоняют, поэтому раздача Wi-Fi не запускается. Обойти это настройками нельзя: `SetWifiCountryCode` существует только в ветке `HDI_CHIP_INTERFACE_SUPPORT`, а без неё всегда возвращает ошибку. Пустой или неверный код страны `SetCountry()` тоже считает ошибкой.

### Изменённые файлы

| Файл | Изменение |
|---|---|
| `wlan/wlan.gni` | объявление `drivers_peripheral_wlan_feature_country_code_mode = "vendor"` в `declare_args()` |
| `wlan/bundle.json` | флаг добавлен в `features`, чтобы его можно было задавать из описания продукта |
| `wlan/chip/hdi_service/BUILD.gn` | для цели `wifi_hal_default` значение флага превращается в макрос `WIFI_HAL_COUNTRY_CODE_MODE` (0 — vendor, 1 — nl80211, 2 — auto) |
| `wlan/chip/wifi_hal/wifi_hal.cpp` | класс `SetRegDomainCommand` (стандартная команда) и выбор способа в `WifiSetCountryCode` |

### Применить

```bash
git -C drivers/peripheral apply $(pwd)/oneir/oneir_patches/drivers_peripheral/0001-wlan-chip-country-code-mode.patch
```

Проверить без применения: добавить `--check`.

### Откатить

```bash
git -C drivers/peripheral apply --reverse $(pwd)/oneir/oneir_patches/drivers_peripheral/0001-wlan-chip-country-code-mode.patch
```

### Включить в продукте

Патч только добавляет возможность выбора, режим задаётся в описании продукта `productdefine/common/products/oneir.json`, в разделе компонента `drivers_peripheral_wlan`:

```json
"drivers_peripheral_wlan_feature_country_code_mode = auto"
```

Без этой строки действует `"vendor"`, то есть поведение OpenHarmony.

### Пересобрать

После применения патча или изменения флага настройки сборки нужно пересоздать:

```bash
./oneir/make.sh --reload hdf_wlan
```

Проверить, что флаг дошёл до кода:

```bash
grep -n 'country_code_mode' out/oneir/args.gn
grep -o 'WIFI_HAL_COUNTRY_CODE_MODE=[0-9]' out/oneir/obj/drivers/peripheral/wlan/chip/hdi_service/wifi_hal_default.ninja
```

Ожидается `"auto"` в `args.gn` и `WIFI_HAL_COUNTRY_CODE_MODE=2`.

---
Пересобирается одна библиотека — `libwifi_hal_default.z.so`. В образ она попадает через `out/oneir/obj/drivers/peripheral/wlan/chip/hdi_service/wifi_hal_default_module_info.json` (раздел `wifi_manager` манифеста).
