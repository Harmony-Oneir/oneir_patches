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
