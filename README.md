# Установка арча

Настройка времени
```
timedatectl set-ntp true
```

Разметить в GPT диск следующим образом:
```
cfdisk --zero /dev/sda
```
  Partition | Type | Size | Mount
  --- | --- | --- | ---
  /dev/sda1 | EFI System | At least 300 MiB | /boot/efi
  /dev/sda2 | Linux swap | More than 512 MiB | [SWAP]
  /dev/sda3 | Linux filesystem | Remainder of the device | /

Форматирование созданных разделов
```
mkfs.fat -F 32 /dev/sda1
mkswap /dev/sda2
mkfs.btrfs -f /dev/sda3
```

Создание сабволумов в бтрфс и их монтирование
```
mount /dev/sda3 /mnt
cd /mnt
btrfs subvolume create @
btrfs subvolume create @home
cd /
umount /mnt
mount -o noatime,compress=zstd,subvol=@ /dev/sda3 /mnt
mkdir -p /mnt/{home,boot/efi}
mount -o noatime,compress=zstd,subvol=@home /dev/sda3 /mnt/home
```

Монтируем остальные разделы
```
mount /dev/sda1 /mnt/boot/efi
swapon /dev/sda2
```

Установка базы
```
pacstrap /mnt base linux linux-firmware amd-ucode intel-ucode
```
_(для бтрфс) Не забыть убрать fsck из HOOKS в **/etc/mkinitcpio.conf** и выполнить команду `mkinitcpio -P`_

Генерация фстаба
```
genfstab -U /mnt >> /mnt/etc/fstab
```
_(для бтрфс) Из **/etc/fstab** убрать subvolid у монтированных сабволумов_

Таймзона
```
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
```

Установить хардварные часы в UTC
```
hwclock --systohc
```

Локализация
Предварительно поправить файл **/etc/locale.gen** и раскомментировать `en_US.UTF-8 UTF-8` и другие нужные языки
```
locale-gen
```
