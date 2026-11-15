## 0001-dhcp-server-reply-length-and-end-option.patch

### Что исправляет

Ответы DHCP-сервера (`OFFER`, `ACK`) уходили обрезанными и без завершающей опции:

1. `DHCP_MSG_HEADER_SIZE` был равен 234, а заголовок BOOTP/DHCP — 236 байт (`op`, `htype`, `hlen`, `hops`, `xid`, `secs`, `flags`, 4 адреса, `chaddr`, `sname`, `file`). Длина ответа считалась как `234 + длина опций`, и `sendto` отправлял на 2 байта меньше записанного: у последней опции отрезались данные.
2. Опция `END` (255) добавлялась в список с длиной 0, а `CreateOptionNode` такие узлы отвергает (в журнале — `failed to create option node`), поэтому маркер конца в пакет не попадал.

Клиент DHCP OpenHarmony останавливается на конце пакета и ошибку не замечает. Android читает длину последней опции, выходит за границу буфера и отбрасывает весь ответ (`DhcpClient: Can't parse packet: BufferUnderflowException`), поэтому телефоны на Android не получали адрес при раздаче Wi-Fi.

### Изменённые файлы

| Файл | Изменение |
|---|---|
| `services/dhcp_server/include/dhcp_message.h` | `DHCP_MSG_HEADER_SIZE` = 236 |
| `services/dhcp_server/src/dhcp_s_server.cpp` | в `ParseReplyOptions` опция `END` записывается явно, если её нет в списке |

### Применить

```bash
git -C foundation/communication/dhcp apply $(pwd)/oneir/oneir_patches/foundation_communication_dhcp/0001-dhcp-server-reply-length-and-end-option.patch
```

### Откатить

```bash
git -C foundation/communication/dhcp apply --reverse $(pwd)/oneir/oneir_patches/foundation_communication_dhcp/0001-dhcp-server-reply-length-and-end-option.patch
```

### Пересобрать

```bash
./make.sh hotspot
```

Пересобирается `libdhcp_server.z.so` (SA 1127, процесс `wifi_manager_service`). На стенде: остановить `wifi_manager_service`, залить через `push_module_info.sh` (`.../dhcp/services/dhcp_server/dhcp_server_module_info.json`), перезагрузить.

### Проверка

Подключить Android к раздаче: в журнале сервера появляются `DHCPREQUEST` и `ACK`, у телефона адрес `192.168.43.x`, в `adb logcat` нет `BufferUnderflowException`.
