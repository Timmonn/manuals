# Подготовка диска

## Разметка диска 

`fdisk -l` *смотрим какие диски и разделы есть*

`fdisk /dev/sdx` подключаемся к диску и далее работаем в утилите fdisk 

​    Если диск размечен и есть разделы:

​    `d` *-удаляем ненужные разделы по очереди (если разделов нет то пропускаем этот шаг)*

Далее там же в fdisk:
`g` *-таблицу разделов размечаем как GPT*
`w` *-записываем и выходим*

## Создание разделов

`cfdisk /dev/sdx` 

`/dev/sdx1` размером 31 Мб типа Bios boot  
`/dev/sdx2` размером 300-500 Мб типа Efi system  
`/dev/sdx3` размером 512 Мб типа swap   (swap создаем только если меньше 2 gb оперативной памяти)  
`/dev/sdx4` остальной размер типа root или Linux file system  

## Форматирование диска

`mkfs.vfat /dev/sdx2`  (раздел Efi system)   
`mkfs.btrfs /dev/sdx4` (раздел root)  
`mkswap  /dev/sdx3` (раздел swap )  
`swapon /dev/sdx3`  (включить swap)  

Раздел который 31 Мб типа Bios boot не трогаем  

## Монтирование разделов 

`mount /dev/sdx4 /mnt`  монтируем корневой или root  
`mkdir /mnt/boot` создаем каталог для загрузчика - если обычный BIOS  

​    если UEFI вводим еще одну команду:  

​    `mkdir /mnt/boot/EFI`  

`mount /dev/sdx2 /mnt/boot` монтируем Efi system бут раздел для обычного Bios  

​    `mount /dev/sdx2 /mnt/boot/EFi` для UEFI биоса  



# Установка базовой системы 

`pacstrap -i /mnt base base-devel linux-zen linux-zen-headers linux-firmware dosfstools btrfs-progs intel-ucode iucode-tool nano`  (`amd-ucode` если amd)

## Генерация конфига разделов 

`genfstab -U /mnt >> /mnt/etc/fstab`

## Переход в chroot

`arch-chroot /mnt`

## Часовой пояс

`ln -sf /usr/share/zoneinfo/Регион(Europe)/Город(Ekaterinburg) /etc/localtime`  

`hwclock --systohc`

## Локализация

`nano /etc/locale.gen`   убрать решетки с нужных локалей  (английский обязательно !)  

*например оставляем это* 

`en_US.UTF-8 UTF-8`  
`ru_RU.UTF-8 UTF-8`  

`locale-gen` потом генерируем локали  
`nano /etc/locale.conf` -далее редактируем конф пишем там:  

`LANG=ru_RU.UTF-8`  

`nano /etc/vconsole.conf` -потом точно также редактируем vconsole пишем там:  

`KEYMAP=ru`  

``FONT=cyr-sun16`  

## Настройка сети

`nano /etc/hostname` имя компьютера  

`nano /etc/hosts` далее файл доменных имен  

127.0.0.1	localhost  
::1		localhost  
127.0.1.1	моёимякомпьютера.localdomain    моёимякомпьютера  

## Initramfs

`mkinitcpio -p linux-zen` если ядер несколько  

`mkinitcpio -P linux-zen`  если ядро одно  


## Пароль суперпользователя

`passwd`

# Устанавливаем загрузчик и сетевые утилиты 

`pacman -S grub efibootmgr dhcpcd dhclient networkamanager` 

устанавливаем загрузчик 

`grub-install /dev/sdx` загрузчик ставим НЕ на раздел , а на диск на котором ставим систему

`grub-mkconfig -o /boot/grub/grub.cfg` конфигурируем загрузчик 


потом выходим из chroot

`exit` 


`umount -R /mnt` 


`useradd -m -G wheel -s /bin/bash юзернейм` логинимся в систему создаем учетную запись юзера 

`passwd юзернейм` 

`nano /etc/sudoers`  раскоменчиваем строку разрешающую группе или юзеру wheel запускать любые команды

# Устанавливаем пакеты графики и оболочку

Наборы пакетов видео ускорения вводим после sudo pacman -Syu

## Intel

`lib32-mesa vulkan-intel lib32-vulkan-intel vulkan-icd-loader lib32-vulkan-icd-loader libva-intel-driver xf86-video-intel`    

## Nvidia

`nvidia-dkms nvidia-utils lib32-nvidia-utils nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader lib32-opencl-nvidia opencl-nvidia libxnvctrl` 

Сюда же оптимизированные DKMS модули проприетарного драйвера NVIDIA

https://aur.archlinux.org/packages/nvidia-dkms-performance/

устанавливаем:

1) git clone [https://aur.archlinux.org/nvidia-dkms...](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbS0xNzE5Qk1lTVJCTVV0S1ZYNzdNVk82ZDExUXxBQ3Jtc0trX1M4c3dxVjNObEd0V1VCMlo0dnJVS21jQ1dTQ2pDLWd4NnJxUmNGczYwWUVJZU1EaXN1Rm5GOVdQSTNrcG52TFBHZ1BHek1JZG5SSktpaXl6eEtmVWcwZXNRVzY0eXp3WDVoZlljSjFQa3lZVWhuUQ&q=https%3A%2F%2Faur.archlinux.org%2Fnvidia-dkms-performance.git&v=aMnaM7llZhM) 

2) cd nvidia-dkms-performance
3) makepkg -sric  ( там соглашаемся с заменой пакета )
4) sudo mkinitcpio -p наименование вашего ядра
5) reboot 

## AMD

`lib32-mesa vulkan-radeon lib32-vulkan-radeon vulkan-icd-loader lib32-vulkan-icd-loader`



## Графические оболочки



### Gnome

` pacman -S  xorg xorg-server gnome gnome-extra gdm`

 Включаем дисплей менеджер

`systemctl enable gdm`

### XFCE

`pacman -S xorg xorg-server xfce4 xfce4-goodies lightdm lightdm-gtk-greeter`

Включает дисплей менеджер

`systemctl enable lightdm`

### KDE plasma

`pacman -S xorg xorg-server plasma  plasma-wayland-session  egl-wayland sddm sddm-kcm packagekit-qt5 kde-applications`

Включаем дисплей менеджер

`systemctl enable sddm`

### Сinnamon

`pacman -S xorg xorg-server cinnamon`

Включаем дисплей менеджер

`systemctl enable gdm`

### Deepin

`pacman -S xorg xorg-server deepin deepin-extra lightdm lightdm-deepin-greeter`

Включаем дисплей менеджер

`systemctl enable lightdm`

### Enlightenment

`pacman -S xorg xorg-server enlightenment lightdm lightdm-gtk-greeter`

Включаем дисплей менеджер

`systemctl enable lightdm`

### Mate

`pacman -S xorg xorg-server  mate   mate-extra  mate-panel   mate-session-manager`

Включаем дисплей менеджер

`systemctl enable mdm`

### LXDE

`pacman -S xorg xorg-server lxde-common  lxsession openbox lxde lxdm`

Включаем дисплей менеджер

`systemctl enable lxdm`

### Прочие графические оболочки

https://wiki.archlinux.org/title/desktop_environment
