# 🐧 Arch Linux Installation Guide (RU)

> Ручная установка Arch Linux с нуля.
> Русская версия.

## Перед началом

* Все данные на выбранном диске будут удалены.
* Проверь, что у тебя есть резервная копия важных файлов.
* Для простого сценария лучше отключить Secure Boot.
* Гайд рассчитан на установку в UEFI, но внизу есть и вариант для BIOS/Legacy.

## Содержание

* [Подготовка](#подготовка)
* [Интернет](#интернет)
* [Подготовка окружения](#подготовка-окружения)
* [Разметка диска](#разметка-диска)
* [Форматирование](#форматирование)
* [Монтирование](#монтирование)
* [Установка системы](#установка-системы)
* [Настройка системы](#настройка-системы)
* [Загрузчик](#загрузчик)
* [Рабочий стол](#рабочий-стол)
* [Пользователь](#пользователь)
* [Первый запуск](#первый-запуск)
* [После установки](#после-установки)
* [Полезные ссылки](#полезные-ссылки)

---

# Подготовка

## Скачать ISO

Скачай официальный образ Arch Linux:

```text
https://archlinux.org/download/
```

Проверить контрольную сумму ISO тоже хорошая идея, если нужна максимальная надёжность.

## Запись на флешку

Флешка нужна минимум на 4 ГБ.

### Windows

Можно использовать:

* Rufus
* Ventoy

### Linux

Можно использовать:

* Ventoy
* balenaEtcher
* dd

Ventoy удобен тем, что ISO-файлы можно просто копировать на флешку.

## Загрузка

Перезагрузи компьютер и открой BIOS/UEFI:

* Del
* F2
* F11
* F12
* Esc

Убедись, что загрузка идёт в нужном режиме:

* UEFI — современный вариант
* BIOS/Legacy — старый режим

---

# Интернет

Проверь соединение:

```bash
ping -c 3 archlinux.org
```

Если работает, идём дальше.

## Wi‑Fi через iwctl

Запусти:

```bash
iwctl
```

Посмотри устройства:

```bash
device list
```

Пример интерфейса:

```text
wlan0
```

Сканирование:

```bash
station wlan0 scan
station wlan0 get-networks
```

Подключение:

```bash
station wlan0 connect ИМЯ_СЕТИ
```

После выхода:

```bash
exit
```

Проверка:

```bash
ping -c 3 archlinux.org
```

---

# Подготовка окружения

## Русская раскладка

```bash
loadkeys ru
setfont cyr-sun16
```

## Проверка UEFI

```bash
cat /sys/firmware/efi/fw_platform_size
```

Если вывод:

```text
64
```

значит система загружена в UEFI.

## Время

```bash
timedatectl set-ntp true
```

---

# Разметка диска

Посмотреть диски:

```bash
lsblk -f
fdisk -l
```

Примеры устройств:

```text
/dev/sda      SATA диск
/dev/nvme0n1  NVMe SSD
/dev/vda      виртуальный диск
```

Открываем разметку:

```bash
cfdisk /dev/ДИСК
```

## UEFI-схема

Создай:

| Раздел | Размер        | Назначение |
| ------ | ------------- | ---------- |
| EFI    | 512 МБ — 1 ГБ | загрузчик  |
| Swap   | по желанию    | подкачка   |
| Root   | остальное     | система    |

## BIOS-схема

Создай:

| Раздел | Назначение |
| ------ | ---------- |
| Swap   | подкачка   |
| Root   | система    |

После изменений выбери:

```text
Write
Quit
```

---

# Форматирование

Ниже пример с такой разметкой:

* EFI — `/dev/sdX1`
* Swap — `/dev/sdX2`
* Root — `/dev/sdX3`

Если у тебя другие номера разделов, замени их под свою схему.

## UEFI

EFI-раздел:

```bash
mkfs.fat -F32 /dev/sdX1
```

Root:

```bash
mkfs.ext4 /dev/sdX3
```

Swap:

```bash
mkswap /dev/sdX2
swapon /dev/sdX2
```

## BIOS (Legacy)

Root:

```bash
mkfs.ext4 /dev/sdX2
```

Swap:

```bash
mkswap /dev/sdX1
swapon /dev/sdX1
```

---

# Монтирование

## UEFI

В этом варианте EFI-раздел монтируется в `/boot`.

```bash
mount /dev/sdX3 /mnt
mkdir -p /mnt/boot
mount /dev/sdX1 /mnt/boot
```

## BIOS

```bash
mount /dev/sdX2 /mnt
```

---

# Установка системы

Ставим базовую систему:

```bash
pacstrap -K /mnt base linux linux-firmware nano sudo networkmanager
```

Если нужен заголовочный пакет для DKMS или сборки модулей:

```bash
pacstrap -K /mnt linux-headers
```

## Микрокод CPU

Intel:

```bash
pacstrap -K /mnt intel-ucode
```

AMD:

```bash
pacstrap -K /mnt amd-ucode
```

---

# Настройка системы

Создай `fstab`:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

Перейди в установленную систему:

```bash
arch-chroot /mnt
```

## Часовой пояс

Пример для Москвы:

```bash
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc
```

Если нужен другой город, замени путь на свой.

## Локализация

Открой файл:

```bash
nano /etc/locale.gen
```

Раскомментируй строки:

```text
ru_RU.UTF-8 UTF-8
en_US.UTF-8 UTF-8
```

Создай локали:

```bash
locale-gen
```

Настрой язык системы:

```bash
echo "LANG=ru_RU.UTF-8" > /etc/locale.conf
```

## Имя компьютера

```bash
echo "arch-pc" > /etc/hostname
```

## Файл hosts

Открой:

```bash
nano /etc/hosts
```

Добавь:

```text
127.0.0.1 localhost
::1 localhost
127.0.1.1 arch-pc.localdomain arch-pc
```

## Раскладка консоли

```bash
echo "KEYMAP=ru" > /etc/vconsole.conf
echo "FONT=cyr-sun16" >> /etc/vconsole.conf
```

## Пароль root

```bash
passwd
```

## Сеть

Включи NetworkManager:

```bash
systemctl enable NetworkManager.service
```

---

# Загрузчик

Выбери один вариант: **systemd-boot** или **GRUB**.

## UEFI + systemd-boot

Установка:

```bash
bootctl --path=/boot install
```

Создай конфиг:

```bash
nano /boot/loader/loader.conf
```

Пример:

```text
default arch
timeout 5
editor no
```

Создай запись загрузки:

```bash
nano /boot/loader/entries/arch.conf
```

Пример для Intel:

```text
title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options root=UUID=ТВОЙ_UUID rw
```

Для AMD замени:

```text
intel-ucode.img
```

на:

```text
amd-ucode.img
```

UUID можно посмотреть так:

```bash
blkid
```

## UEFI + GRUB

Установи пакеты:

```bash
pacman -S grub efibootmgr
```

Установка GRUB:

```bash
grub-install \
  --target=x86_64-efi \
  --efi-directory=/boot \
  --bootloader-id=Arch
```

Создай конфиг:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

## BIOS + GRUB

Установи пакет:

```bash
pacman -S grub
```

Установка GRUB:

```bash
grub-install --target=i386-pc /dev/sdX
```

Важно: здесь указывается диск целиком, а не раздел.

Создай конфиг:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

---

# Рабочий стол

Выбирай **только одну** среду: GNOME или KDE Plasma.

## Вариант A: GNOME

Установка:

```bash
pacman -S gnome gdm gnome-tweaks xdg-desktop-portal-gnome
systemctl enable gdm.service
```

Если нужен набор дополнительных приложений GNOME, можно поставить:

```bash
pacman -S gnome-extra
```

## Вариант B: KDE Plasma

Установка:

```bash
pacman -S plasma-meta sddm xdg-desktop-portal-kde plasma-nm
systemctl enable sddm.service
```

Если нужен более полный набор приложений KDE, можно поставить:

```bash
pacman -S kde-applications
```

---

# Пользователь

Создай обычного пользователя:

```bash
useradd -m -G wheel username
passwd username
```

Разреши `sudo` для группы `wheel`:

```bash
EDITOR=nano visudo
```

Найди строку:

```text
%wheel ALL=(ALL:ALL) ALL
```

Убери `#` в начале строки.

---

# Первый запуск

Выйди из chroot:

```bash
exit
```

Размонтируй разделы:

```bash
umount -R /mnt
```

Перезагрузи систему:

```bash
reboot
```

После перезагрузки вынь флешку.

---

# После установки

Сразу после первого входа обычно делают обновление:

```bash
sudo pacman -Syu
```

Полезные базовые пакеты:

```bash
sudo pacman -S man-db man-pages git bash-completion
```

Если нужен Bluetooth:

```bash
sudo pacman -S bluez bluez-utils
sudo systemctl enable bluetooth.service
```

Если нужен NetworkManager в графической среде, просто убедись, что сервис включён:

```bash
systemctl status NetworkManager
```

Проверить загрузку можно так:

```bash
systemd-analyze
```

---

# Полезные ссылки

* [ArchWiki](https://wiki.archlinux.org/)
* [Arch Installation Guide](https://wiki.archlinux.org/title/Installation_guide)
* [Arch Linux Downloads](https://archlinux.org/download/)
* [systemd-boot](https://wiki.archlinux.org/title/Systemd-boot)
* [GRUB](https://wiki.archlinux.org/title/GRUB)
* [NetworkManager](https://wiki.archlinux.org/title/NetworkManager)
* [GNOME](https://wiki.archlinux.org/title/GNOME)
* [KDE Plasma](https://wiki.archlinux.org/title/KDE)
