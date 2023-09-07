# Arch Linux with Xfce4/i3 Installation Guide

<sub>*Updated: August 2023*</sub>
<br>
<sub>*Author: <a href="https://github.com/silentz">Maxim Pershin</a>*</sub>

<img src="./logo.png" />

### Getting Started

Welcome to the Arch Linux Installation Guide with Xfce4 Desktop Environment!

This guide provides you with a step-by-step walkthrough of installing
Arch Linux along with the Xfce4 desktop environment. It has been carefully created
based on my own experience of installation Arch Linux on multiple devices over the years.
This guide aims to make your installation process as smooth as possible.

To begin your Arch Linux installation, please follow the step-by-step instructions provided below.

### Support and Feedback

If you have any suggestions, corrections, or encounter any issues while following
the guide, I encourage you to get involved through Github.

**Issues:** If you come across any problems or have specific questions, please open
an issue on the Github repository for this guide. This allows me to track and
address your concerns effectively.

**Pull Requests:** If you have improvements or additions to the guide, feel free to submit
a pull request. Your contributions can help enhance the clarity of the guide for everyone.

## Downloading Arch Linux image

1. Go to Arch Linux downloads page https://archlinux.org/download/

2. Find **HTTP Direct Downloads** section and choose any download mirror.
Select a mirror that is geographically closer to your location.

3. On the mirror page find `archlinux-2023.06.01-x86_64.iso` or `archlinux-x86_64.iso`
or any other file with `.iso` suffix. Other files (like *.txt*, *.tar.gz* and even *.iso.sig*)
are not needed for installation process.

## Preparing installation medium

1. Insert a USB-stick into your PC with at least 2Gb of space availavle on it.

2. Find corresponding block device for USB-stick in `/dev` folder. Usually it is `/dev/sdb`.

**IMPORTANT NOTE**: you need block device without a number on the end.
If you have for example */dev/sdb*, */dev/sdb1* and */dev/sdb2* you need */dev/sdb* !

3. Burn previously downloaded Arch Linux ISO-image on a USB-stick (in my case it is `/dev/sdb`):
```
sudo dd if=./archlinux-2023.06.01-x86_64.iso of=/dev/sdb conv=fsync oflag=direct status=progress
```

## Installation

1. Insert the installation medium into the computer on which you are installing Arch Linux.

2. Power on your PC and press *boot menu key*. For *Lenovo ThinkPad X1 Carbon* series laptop,
this key is `F12`.

3. Boot from USB-stick and wait until boot process is finished.

**IMPORTANT NOTE**: not every device can run a system from USB-stick out of the box.
Many BIOS'es by default come with activated *Secure boot* option.You might need to
deactivate it in your BIOS.

4. *(Step is optional if you are using wired connection)*
Connect to WiFi using `iwctl` and check connection is established:
```
iwctl
[iwd]# station wlan0 get-networks
[iwd]# station wlan0 connect <Name of WiFi access point>
[iwd]# exit
ping 1.1.1.1
```

5. Syncronize pacman packaes:
```
pacman -Syy
```

6. Partition main device using `fdisk` utility (you can find the name using `lsblk` command).
```
fdisk /dev/nvme0n1

# repeat until all existing partitions are deleted

Command (m for help): d

# create partition 1: efi

Command (m for help): n
Partition number (1-128, default 1): \Enter
First sector (..., default 2048): \Enter
Last sector ...: +256M

# create partition 2: main (use -<double size of your RAM on last step>G)

Command (m for help): n
Partition number (2-128, default 2): \Enter
First sector (..., default ...): \Enter
Last sector ...: -32G

# create partition 3: swap

Command (m for help): n
Partition number (3-128, default 3): \Enter
First sector (..., default ...): \Enter
Last sector ...: \Enter

# change partition types

Command (m for help): t
Partition number (1-3, default 1): 1
Partion typr or alias (type L to list all): uefi

Command (m for help): t
Partition number (1-3, default 2): 2
Partion typr or alias (type L to list all): linux

Command (m for help): t
Partition number (1-3, default 3): 3
Partion typr or alias (type L to list all): swap

# write partitioning to disk

Command (m for help): w
```

7. Create filesystems on created disk partitions:
```
mkfs.fat -F 32 /dev/nvme0n1p1 # on EFI System
mkfs -t ext4 /dev/nvme0n1p2   # on Linux filesystem partition
mkswap /dev/nvme0n1p3         # on Linux swap
```

8. Correctly mount all filesystems to the `/mnt`:
```
mount /dev/nvme0n1p2 /mnt
mkdir -p /mnt/boot/efi
mount /dev/nvme0n1p1 /mnt/boot/efi
swapon /dev/nvme0n1p3
```

9. Install essential packages into new filesystem and generate fstab:
```
pacstrap -i /mnt base linux linux-firmware sudo vim
genfstab -U -p /mnt > /mnt/etc/fstab
```

10. Chroot into filesystem:
```
arch-chroot /mnt
```

11. Setup system locale:
```
vim /etc/locale.gen
# uncomment `en_US.UTF-8` and `en_GB.UTF-8` inside /etc/locale.gen
locale-gen
```

12. Configure timezone, set your own:
```
echo "LANG=en_US.UTF-8" > /etc/locale.conf
ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime
```

13. Setting up hardware clock:
```
hwclock --systohc
```

14. Setup hostname and `/etc/hosts`, use your own hostname:
```
echo carbon > /etc/hostname
vim /etc/hosts

# 127.0.0.1    localhost
# ::1          localhost
# 127.0.1.1    carbon
```

15. Add new user:
```
useradd -m -G wheel,storage,power,audio,video,docker -s /bin/bash max
```

16. Setup root and user passwords:
```
passwd
passwd max
```

17. Add wheel group to sudoers to allow sudo from user:
```
visudo

# uncomment this line in file:
# %wheel ALL=(ALL) ALL
```

18. Install and configure grub:
```
pacman -S grub efibootmgr
grub-install /dev/nvme0n1
grub-mkconfig -o /boot/grub/grub.cfg
```

19. Install NetworkManager:
```
pacman -S dhcpcd networkmanager
systemctl enable sshd
systemctl enable dhcpcd
systemctl enable NetworkManager
```

20. Exit chroot, unmount all disks and reboot:
```
exit
umount /mnt/boot/efi
umount /mnt
reboot
```

## Configuring installed Arch Linux

1. Activate time syncronization using NTP:
```
timedatectl set-ntp true
```

2. Connect to WiFi using `nmcli`:
```
nmcli device wifi connect <SSID> password <password>
```

3. Install Xorg:
```
sudo pacman -S xorg xorg-apps xorg-xinit xdotool
```

4. Install useful packages:
```
sudo pacman -S iw wpa_supplicant dialog intel-ucode git reflector rsync bash-completion w3m
sudo pacman -S base-devel lshw zip unzip htop wget xsel tree fuse2 keychain axel
sudo pacman -S sof-firmware pulseaudio alsa-utils alsa-plugins pavucontrol
```

5. Install SSD TRIM:
```
sudo systemctl enable fstrim.timer
```

6. Install Xfce
```
sudo pacman -S dbus xfce4 xfce4-screenshooter \
  thunar-archive-plugin thunar-media-tags-plugin \
  xfce4-xkb-plugin xfce4-battery-plugin xfce4-datetime-plugin xfce4-mount-plugin \
  xfce4-netload-plugin xfce4-notifyd xfce4-pulseaudio-plugin xfce4-screensaver \
  xfce4-wavelan-plugin xfce4-weather-plugin xfce4-whiskermenu-plugin network-manager-applet
```
or install i3
```
sudo pacman -S i3-wm i3status i3blocks i3lock lxappearance
sudo pacman -S polybar rofi ranger thunar alacritty dunst feh xss-lock \
               system-config-printer light pango flameshot gsimplecal
```

7. Install desktop manager:
```
sudo pacman -S ly
sudo systemctl enable ly
```

8. Setup bluetooth:
```
sudo pacman -S bluez bluez-utils blueman
sudo systemctl enable bluetooth
```

9. Impreove battary usage:
```
sudo pacman -S tlp tlp-rdw acpi acpi_call
sudo systemctl enable tlp
sudo systemctl mask systemd-rfkill.service
sudo systemctl mask systemd-rfkill.socket
```

10. Install essential fonts:
```
sudo pacman -S noto-fonts ttf-ubuntu-font-family ttf-dejavu ttf-freefont
sudo pacman -S ttf-liberation ttf-droid ttf-roboto terminus-font
```

11. Install themes and icons:
```
sudo pacman -S arc-gtk-theme
sudo pacman -S papirus-icon-theme
```

12. Setup the fastest pacman mirror, choose nearest countries:
```
sudo reflector --country Germany,Austria,Switzerland --fastest 10 --save /etc/pacman.d/mirrorlist
```

13. Intall printing settings:
```
sudo pacman -S cups cups-filters cups-pdf system-config-printer --needed
sudo systemctl enable cups.service
```

**Important** if there is no option for system-config-printer in xfce4-settings-manager,
go to `/usr/share/applications/system-config-printer.desktop` and set
`Categories=System;Settings;X-XFCE-SettingsDialog;X-XFCE-HardwareSettings;`

14. Install NetworkManager additionals:
```
sudo pacman -S nm-connection-editor networkmanager-openvpn
```

**Important** if there is no option for nm-connection-editor in xfce4-settings-manager,
go to `/usr/share/applications/nm-connection-editor.desktop` and set
`Categories=System;Settings;X-XFCE-SettingsDialog;X-XFCE-HardwareSettings;`

15. Reboot again:
```
reboot
```

## Hibernation support

1. Open you `/etc/fstab` and find UUID for your swap partition.

2. Open grub configuration file:
```
sudo vim /etc/default/grub
```

3. Find option `GRUB_CMDLINE_LINUX_DEFAULT="..."`

4. Insert `resume=UUID=<uuid of swap partition from /etc/fstab>`
(example: `GRUB_CMDLINE_LINUX_DEFAULT='... resume=UUID=17f82588-0b79-419a-954c-4a9a1ee90b70'`)

4. Regenerate grub config:
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

5. Open mkinitcpio configuration file:
```
sudo vim /etc/mkinitcpio.conf
```

6. Find option `HOOKS="base udev autodetect modconf block filesystems keyboard fsck"`

7. After `udev` insert hook `resume` (like this: `... base udev resume ...`)

8. Regenerate initramfs:
```
sudo mkinitcpio -p linux
```

9. You can now use:
```
sudo systemctl hibernate
```

## Additional: installing client apps

*These all are my personal apps, so you can use your own*

### General
```
sudo pacman -S chromium telegram-desktop fontforge gparted obs-studio \
               tilix vlc remmina wireshark-qt neofetch evince gimp spotify-launcher \
               shotwell file-roller shotcut inkscape evolution mousepad redshift klawaro
```

### Yubikey
```
sudo pacman -S yubikey-personalization-gui yubikey-manager
```

### Wine

1. Go to `/etc/pacman.conf` and uncomment (or add) following lines:
```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

2. Update pacman package databases:
```
sudo pacman -Syu
```

3. Install wine using pacman:
```
sudo pacman -S wine wine-mono wine-gecko winetricks zenity
```

4. Configure smooth font:
```
winetricks settings fontsmooth=rgb
```

**Important**: if you are stuck with error
`wine: Read access denied for device L"\\??\\Z:\\", FS volume label and serial are not available.`,
go to `~/.wine/dosdevices`, remove `z:` symbolic link and make it point to your `$HOME`.

### DevTools
```
# General
sudo pacman -S neovim stow sqlite3 tldr jq tmux openvpn \
               wireguard-tools zip unzip virtualbox nmap masscan \
               pgcli redis plantuml ripgrep gitui gpick

sudo setcap 'cap_net_raw+epi' /usr/bin/masscan

# Devops
sudo pacman -S docker docker-compose kubectl helm aws-cli-v2 terraform
sudo systemctl enable docker

# C++
sudo pacman -S gcc gdb cmake ninja clang

# Lua
sudo pacman -S lua

# Golang
sudo pacman -S golang
go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest

# Javascript
sudo pacman -S nodejs npm yarn

# Java
sudo pacman -S jdk20-openjdk

# Solidity
sudo pacman -S solidity

# Virtualbox
sudo pacman -S virtualbox-host-dkms
sudo pacman -S virtualbox
```

### Install AUR package manager

```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### Install AUR packages

```
yay -S python36 python37 python38 python39 python310
```

### Install with manual download

1. Slack desktop client
2. Flutter (+dart inside flutter distribution)
3. Python pip

### Install texlive (LaTeX)


1. Install `texlive-core`
```
sudo pacman -S texlive-core
```

2. Go to `/usr/share/texmf-dist/scripts/texlive/tlmgr.pl` and replace:
`$Master = "$Master/../..";` with `$Master = "${Master}/../../..";`

3. Create an alias in your `.bashrc`:
```
alias tlmgr='/usr/share/texmf-dist/scripts/texlive/tlmgr.pl --usermode'
```

4. initialize tlmgr in user mode:
```
tlmgr init-usertree
```

5. Set your preferred mirror. For example:
```
tlmgr option repository http://mirrors.rit.edu/CTAN/systems/texlive/tlnet
```

6. Now you can install CTAN packages. Install scheme-full (all packages):
```
tlmgr install scheme-full
```

## Setup Android DevTools

* Download zip-archive from here: https://developer.android.com/studio
(*Command line tools only* section).
* Run `unzip commandlinetools-linux-..._latest.zip`
* Run `mkdir -p ~/Android/cmdline-tools/latest`
* Run `mv ./cmdline-tools/* ~/Android/cmdline-tools/latest/`
* Run `rmdir cmdline-tools` (downloaded one, not in `~/Android/...`).
* Set `ANDROID_HOME` environment variable to `$HOME/Andoird`.
* Run `sdkmanager "platform-tools" "platforms;android-29"`
* Run `sdkmanager "build-tools" "build-tools;29.0.3"`
* Run `sdkmanager --licenses`
* Run `sdkmanager --update`

## Tools for reverse engineering CTF's

Binaries:
`gdb`, `strace`, `ltrace`, `lld`, `objdump` `radare2`, `frida`,
`Ghidra`, `IDA Pro`, `cutter` + `rz-ghidra` + `cutterref`, `angr-management`
`API Monitor`, `PEiD`, `UpxUnpacker`

Python: `pycdc`

Java: `jd-gui`, `jadx`

C#: `Avalonia ILSpy`

## Grub resolution fix

*This can help if you have very tiny grub font on your 4k monitor*

1. Open `/etc/default/grub` with text editor.

2. Add these lines (setting multiple resolutions as fallback):
```
GRUB_TERMINAL_OUTPUT="gfxterm"
GRUB_GFXPAYLOAD_LINUX=keep
GRUB_GFXMODE=1920x1080x32,1024x768x32,auto
```

3 Regenerate `grub.cfg`:
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

## Lightdm resolution fix

*This can help if you have very tiny lightdm font on your 4k monitor*

1. Open `/etc/lightdm/lightdm.conf` with text editor.

2. Add following lines to the file:
```
display-setup-script=xrandr --output eDP-1 --mode 1920x1080
```
**Important: place this line in [Seat:\*] section of lightdm.conf file!!!**

P.S. *your screen output name, like eDP-1 in my case, can be found in `xrandr -q`*

## Other small fixes:

* Slack remove annoying menu bar: `Window -> Always show menu bar -> disable`
* If system goes to sleep after 3-5 minutes, this might be screensaver. To stop this, disable option `Settings -> Screensaver -> Activate Screensaver when computer is idle`
