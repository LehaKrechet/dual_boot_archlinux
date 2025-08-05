# dual_boot_archlinux

setfont /usr/share/kbd/consolefonts/ter-124n.psf.gz --ставим шрифт побольше

1.Чтобы подтвердить наличие поддержки EFI, выполните команду:

ls /sys/firmware/efi/efivars

2. Для проверки наличия интернета

ping 8.8.8.8 

3. Если интернета нет

ip link --смотрим название интерфейса
rfkill unblock wifi --если интерфейс есть но выключен
ip link set wlan0 up  --Влючам беспроводной интерфейс

iwctl --утилита для подключения

Далее команды внутри утилки

device list --смотрим сетевую карту

station wlan0 scan

station wlan0 get-networks --смотрим сети

station wlan0 connect ИмяСети --подключаемся к сети

или если iwctl не работает

nmcli device wifi list --смотрим список сетей

nmcli device wifi connect access_point_name password your_password --подключаемся к сети

4. Обновляем время и дату

timedatectl set-ntp true

timedatectl status

5. Создание и форматирование раздела

lsblk --смотрим диски

cfdisk /dev/Мой-диск

создаем два раздела корневой(linux filesystem) и подкачки(linux swap) - 
как доп оперативная память чем меньше на компе тем больше swap желательно половину от оперативы минимум 4 гб

в конце write quit

вместо sda6 свой диск

mkfs.ext4 /dev/sda5 --форматируем корневой раздела

mkswap /dev/sda6 --форматируем раздел подкачки

swapon /dev/sda6 --включаем раздел swap

mount /dev/sda5 /mnt  --монтируем раздел

mkdir /mnt/efi --директоия для загрузчика

mount /dev/sda1 /mnt/efi -- монтирум загрузчик

6. Установка системы

pacman -S base base-devel linux linux-headers linux-firmware nano networkmanager networkmanager-applet wireless-tools bluez bluez-utils git

или

pacstrap /mnt base linux linux-firmware


genfstab /mnt >> /mnt/etc/fstab --создаем fstab


7. Основная настройка

arch-chroot /mnt --запускаемя в arch

ln -sf /usr/share/zoneinfo/US/Pacific /etc/localtime --устанавливаем временную зону

hwclock --systohc --синхронизируем часы

nano /etc/locale.gen --раскоментировать нужный язык

locale-gen --генерирум локал файл

echo linuxtechi > /etc/hostname --устанавливаем имя хоста

echo "127.0.1.1 linuxtechi" >> /etc/hosts


pacman -Sy netctl

pacman -Sy dhcpcd ifplugd  --ставим сетевой месенджер


useradd -G wheel -m linuxtechi --добавляем пользователя в группу wheel

passwd linuxtechi --устанавливаем пароль пользователя

passwd --ставим пароль рута

/etc/sudoers --тама wheel ALL=(ALL) ALL что-бы дать рут группе wheel


8. Ставим загрузчик

pacman -S grub efibootmgr

pacman -S os-prober

grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB

grub-mkconfig -o /boot/grub/grub.cfg


systemctl enable networkmanager bluetooth --включаем интернет и блютуз

exit
reboot


Исправление записи в меню загрузки Windows (если она не отображается):

I. Загрузите Arch Linux.

II. Определите раздел Windows: lsblk

III. Монтирование раздела Windows:

монтировать /dev/ < имя_раздела_windows > /mnt/windows

IV. Редактировать /etc/default/grub:
Отредактируйте файл с помощью nano.

Раскомментируйте строку:GRUB_DISABLE_OS_PROBER

V. Регенерация конфигурации Grub:

grub-mkconfig -o /boot/grub/grub.cfg

6. Перезагрузите систему.
