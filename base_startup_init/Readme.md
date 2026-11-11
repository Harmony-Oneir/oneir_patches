**0001-ueventd-by-name-links-without-boot-device.patch**:
ueventd создаёт короткие ссылки /dev/block/by-name/<GPT-метка> только для диска, указанного в cmdline параметром default_boot_device=<sysfs-путь>. Путь зависит от железа (PCI-слот NVMe), поэтому на ПК/ноутбуке он неудобен. Патч: если `default_boot_device` НЕ задан — ссылки создаются для любого диска.
Если задан — поведение оригинальное.
Нужно для fstab вида:
  /dev/block/by-name/userdata  /data  ext4  ...
  
Репозиторий: base/startup/init

Файл:        ueventd/ueventd_device_handler.c, функция BootDeviceIsMatched()

Применить:
```bash
cd /mnt/ohos/OpenHarmony7
P=$PWD/oneir/oneir_patches/base_startup_init/0001-ueventd-by-name-links-without-boot-device.patch
git -C base/startup/init apply --check -v $P && git -C base/startup/init apply $P
```

Проверить, применён ли
```bash
git -C base/startup/init apply --reverse --check $P && echo "применён"
```
Откатить
```bash
git -C base/startup/init apply --reverse $P
```
Меняется только бинарник ueventd (/system/bin/ueventd), ядро не трогается. Необходимо пересобрать цель rootfs_init.

---

**0002-ueventd-by-name-links-for-pci-block-devices.patch**:
ueventd создаёт ссылки by-name только для дисков на platform-шине (ARM-платы).
На x86 NVMe/SATA/USB-диски находятся на PCI — ссылок не было вообще.

Патч: при обходе sysfs вверх от раздела принимается и /sys/bus/pci.
Поиск останавливается на ближайшем PCI-устройстве (контроллер диска),
чтобы не плодить дубли за PCI-мостами. Для platform-устройств поведение
не изменилось.

Результат для NVMe:
  /dev/block/platform/pci0000:00/0000:00:03.0/by-name/userdata
  /dev/block/by-name/userdata        (вместе с патчем 0001)

Репозиторий: base/startup/init
Файл:        ueventd/ueventd_device_handler.c, функция GetBlockDeviceSymbolLinks()
Зависит от:  применять ПОСЛЕ 0001

Применить:
```bash
cd /mnt/ohos/OpenHarmont7
D=$PWD/oneir/oneir_patches/base_startup_init
git -C base/startup/init apply --check -v $D/0002-*.patch && git -C base/startup/init apply $D/0002-*.patch
```
Проверить, применён ли:
```
git -C base/startup/init apply --reverse --check $D/0002-*.patch && echo "применён"
```
Откатить (только 0002):
```bash
git -C base/startup/init apply --reverse $D/0002-*.patch
```

