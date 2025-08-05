<!DOCTYPE html>
<html>
<body>

<h1>Установка Arch Linux в Dual Boot</h1>

<div class="step">
    <h2>1. Начальная настройка</h2>
    <p>Увеличиваем шрифт для удобства:</p>
    <pre><code>setfont /usr/share/kbd/consolefonts/ter-124n.psf.gz</code></pre>
    <p>Проверяем поддержку EFI:</p>
    <pre><code>ls /sys/firmware/efi/efivars</code></pre>
    <p>Проверяем интернет соединение:</p>
    <pre><code>ping 8.8.8.8</code></pre>
</div>

<div class="step">
    <h2>2. Настройка сети</h2>
    <p>Если интернет не работает:</p>
    <pre><code>ip link                  <span class="comment"># Смотрим название интерфейса</span>
rfkill unblock wifi     <span class="comment"># Разблокируем Wi-Fi если выключен</span>
ip link set wlan0 up    <span class="comment"># Включаем беспроводной интерфейс</span></code></pre>
    <p>Подключаемся через iwctl:</p>
    <pre><code>iwctl
[iwd]# device list
[iwd]# station wlan0 scan
[iwd]# station wlan0 get-networks
[iwd]# station wlan0 connect ИмяСети</code></pre>
    <p>Или через nmcli:</p>
    <pre><code>nmcli device wifi list
nmcli device wifi connect ИмяСети password ваш_пароль</code></pre>
</div>

<div class="step">
    <h2>3. Настройка времени</h2>
    <pre><code>timedatectl set-timezone Europe/Moscow
timedatectl status</code></pre>
</div>

<div class="step">
    <h2>4. Разметка диска</h2>
    <pre><code>lsblk                     <span class="comment"># Просмотр дисков</span>
cfdisk /dev/Мой-диск      <span class="comment"># Разметка диска</span></code></pre> 
    <div class="note">
        <p>Создаем разделы:</p>
        <ul>
            <li>Корневой раздел (linux filesystem)</li>
            <li>Раздел подкачки (linux swap) - рекомендуется половина от объема ОЗУ, минимум 4 ГБ</li>
        </ul>
        <p>Не забудьте нажать <code>Write</code> и <code>Quit</code> после завершения.</p>
    </div>
  <p> Форматируем о монтирум разделы</p>
    <pre><code>mkfs.ext4 /dev/sda5       <span class="comment"># Форматируем корневой раздел</span>
mkswap /dev/sda6         <span class="comment"># Форматируем swap</span>
swapon /dev/sda6         <span class="comment"># Активируем swap</span>
mount /dev/sda5 /mnt     <span class="comment"># Монтируем корневой раздел</span>
mkdir /mnt/efi           <span class="comment"># Создаем директорию для загрузчика</span>
mount /dev/sda1 /mnt/efi <span class="comment"># Монтируем EFI раздел</span></code></pre>
</div>

<div class="step">
    <h2>5. Установка системы</h2>
    <pre><code>pacman -S base base-devel linux linux-headers linux-firmware nano networkmanager networkmanager-applet wireless-tools bluez bluez-utils git</code></pre>
    <p>Или минимальный вариант:</p>
    <pre><code>pacstrap /mnt base linux linux-firmware</code></pre>
    <p>Генерируем fstab:</p>
    <pre><code>genfstab /mnt >> /mnt/etc/fstab</code></pre>
</div>

<div class="step">
    <h2>6. Основная настройка</h2>
    <pre><code>arch-chroot /mnt</code></pre>
    <p>Настройка времени:</p>
    <pre><code>ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc</code></pre>
    <p>Настройка локали:</p>
    <pre><code>nano /etc/locale.gen   <span class="comment"># Раскомментируем нужный язык</span>
locale-gen</code></pre>
    <p>Настройка сети:</p>
    <pre><code>echo linuxtechi > /etc/hostname
echo "127.0.1.1 linuxtechi" >> /etc/hosts
pacman -Sy netctl dhcpcd ifplugd</code></pre>
    <p>Создание пользователя:</p>
    <pre><code>useradd -G wheel -m linuxtechi          <span class="comment"># Пользователь linuxtechi в группу wheel</span>
passwd linuxtechi                       <span class="comment"># Пароль пользователя</span>
passwd                                  <span class="comment"># Пароль рута</span></code></pre>
    <div class="warning">
        <p>Не забудьте добавить строку в <code>/etc/sudoers</code>:</p>
        <pre><code>wheel ALL=(ALL) ALL</code></pre>
    </div>
</div>

<div class="step">
    <h2>7. Установка загрузчика</h2>
    <pre><code>pacman -S grub efibootmgr os-prober
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg</code></pre>
    <p>Включаем необходимые сервисы:</p>
    <pre><code>systemctl enable networkmanager bluetooth</code></pre>
    <p>Выходим и перезагружаемся:</p>
    <pre><code>exit
reboot</code></pre>
</div>

<div class="step">
    <h2>8. Исправление загрузки Windows (если не отображается)</h2>
  <p>Мне вот этот кусок с монтированием был не нужен</p>
    <pre><code>lsblk                              <span class="comment"># Определяем раздел Windows</span>
mkdir /mnt/windows                <span class="comment"># Создаем точку монтирования</span>
mount /dev/&lt;имя_раздела_windows&gt; /mnt/windows</code></pre>
    <p>Редактируем конфиг GRUB:</p>
    <pre><code>nano /etc/default/grub</code></pre>
    <p>Раскомментируем строку:</p>
    <pre><code>GRUB_DISABLE_OS_PROBER=false</code></pre>
    <p>Перегенерируем конфиг GRUB:</p>
    <pre><code>grub-mkconfig -o /boot/grub/grub.cfg</code></pre>
    <p>Перезагружаем систему:</p>
    <pre><code>reboot</code></pre>
</div>

</body>
</html>
