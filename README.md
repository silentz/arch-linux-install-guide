<img src="./logo.png" />

<div align="center">

[![Author](https://img.shields.io/badge/Author-Maxim_Pershin-ff6f00)](https://github.com/silentz)
[![License](https://img.shields.io/badge/License-Apache--2.0-blue)](./LICENSE.txt)
![Last Updated](https://img.shields.io/badge/Last_Updated-August_2024-02b532)

</div>

<h1 align="center">Arch Linux with Xfce4 and i3 Window Manager Installation Guide</h1>

<div align="center">
    <i>How to install Arch Linux with i3 and not spend ages on debugging</i>
</div>

### Getting Started

Welcome to the Arch Linux with i3 Tiling Window Manager Installation Guide!

This guide provides you with a step-by-step walkthrough of installing
Arch Linux along with the i3 tiling window manager. It has been carefully created
based on my own experience of installation Arch Linux on multiple devices over the years.
This guide aims to make your installation process as smooth as possible.

To begin your Arch Linux installation journey, please follow the step-by-step instructions provided below.

### Support and Feedback

If you have any suggestions, corrections, or encounter any issues while following
the guide, I encourage you to get involved through Github.

**Issues:** If you come across any problems or have specific questions, please open
an issue on the Github repository for this guide. This allows me to track and
address your concerns effectively.

**Pull Requests:** If you have improvements or additions to the guide, feel free to submit
a pull request. Your contributions can help enhance the clarity of the guide for everyone.

<h1 align="center">
    Section 01: Step-by-step guide of installing Arch Linux on your hardware &#128640;
</h1>

### Step 01: Downloading Arch Linux image

1. Go to Arch Linux downloads page https://archlinux.org/download/

2. Find **HTTP Direct Downloads** section and choose any download mirror.
   Select a mirror that is geographically closer to your location.

3. On the mirror page find archive named like `archlinux-YYYY.MM.DD-x86_64.iso` or `archlinux-x86_64.iso`
   or any other file with `.iso` suffix. Other files (like _.txt_, _.tar.gz_ and even _.iso.sig_)
   are not needed for installation process.

### Step 02: Preparing installation medium

1. Insert a USB-stick into your PC with at least 2Gb of space availavle on it.

2. Find corresponding block device for USB-stick in `/dev` folder. Usually it is `/dev/sdb`.

<dl><dd>
<b>IMPORTANT NOTE</b>: you need block device without a number on the end.
If you have for example <i>/dev/sdb</i>, <i>/dev/sdb1</i> and <i>/dev/sdb2</i> you need <i>/dev/sdb</i> !
</dd></dl>

3. Burn previously downloaded Arch Linux ISO-image on a USB-stick (in my case it is `/dev/sdb`):

<dl><dd>
<pre>
$ <b>sudo dd conv=fsync oflag=direct status=progress \
          if=./archlinux-YYYY.MM.DD-x86_64.iso of=/dev/sdb</b>
</pre>
</dd></dl>

### Step 03: Boot into Arch Linux medium

1. Insert the installation medium into the computer on which you are installing Arch Linux.

2. Power on your PC and press _boot menu key_. For _Lenovo ThinkPad X1 Carbon_ series laptop,
   this key is `F12`.

3. Boot from USB-stick and wait until boot process is finished.

<dl><dd>
<b>IMPORTANT NOTE</b>: not every device can run a system from USB-stick out of the box.
Many BIOS'es by default come with activated <i>Secure boot</i> option.You might need to
deactivate it in your BIOS.
</dd></dl>

### Step 04: Syncronize packages

1. [Optional] Connect to WiFi using `iwctl` and check connection is established:

<dl><dd>
<pre>
$ <b>iwctl</b>
[iwd]# <b>station wlan0 get-networks</b>
[iwd]# <b>station wlan0 connect &lt;Name of WiFi access point&gt;</b>
[iwd]# <b>exit</b>
$ <b>ping 1.1.1.1</b>
</pre>
</dd></dl>

2. Syncronize pacman packaes:

<dl><dd>
<pre>
$ <b>pacman -Syy</b>
</pre>
</dd></dl>

### Step 05: Disk partitioning

1. Partition main storage device using `fdisk` utility. You can find storage device name using `lsblk` command.

<dl><dd>
<pre>
$ <b>fdisk /dev/nvme0n1</b>
                <i>[repeat this command until existing partitions are deleted]</i>
Command (m for help): <b>d</b>
Command (m for help): <b>d</b>
Command (m for help): <b>d</b>
<span />
                <i>[create partition 1: efi]</i>
Command (m for help): <b>n</b>
Partition number (1-128, default 1): <b>Enter &crarr;</b>
First sector (..., default 2048): <b>Enter &crarr;</b>
Last sector ...: <b>+256M</b>
<span />
                <i>[create partition 2: main]</i>
Command (m for help): <b>n</b>
Partition number (2-128, default 2): <b>Enter &crarr;</b>
First sector (..., default ...): <b>Enter &crarr;</b>
Last sector ...: <b>-32G</b> <i>// double size of your RAM</i>
<span />
                <i>[create partition 3: swap]</i>
Command (m for help): <b>n</b>
Partition number (3-128, default 3): <b>Enter &crarr;</b>
First sector (..., default ...): <b>Enter &crarr;</b>
Last sector ...: <b>Enter &crarr;</b>
<span />
                <i>[change partition types]</i>
Command (m for help): <b>t</b>
Partition number (1-3, default 1): <b>1</b>
Partion typr or alias (type L to list all): <b>uefi</b>
Command (m for help): <b>t</b>
Partition number (1-3, default 2): <b>2</b>
Partion typr or alias (type L to list all): <b>linux</b>
Command (m for help): <b>t</b>
Partition number (1-3, default 3): <b>3</b>
Partion typr or alias (type L to list all): <b>swap</b>
<span />
                <i>[write partitioning to disk]</i>
Command (m for help): <b>w</b>
</pre>
</dd></dl>

2. Create filesystems on created disk partitions:

<dl><dd>
<pre>
$ <b>mkfs.fat -F 32 /dev/nvme0n1p1</b> <i># on EFI System partition</i>
$ <b>mkfs -t ext4 /dev/nvme0n1p2</b>   <i># on Linux filesystem partition</i>
$ <b>mkswap /dev/nvme0n1p3</b>         <i># on Linux swap partition</i>
</pre>
</dd></dl>

3. Correctly mount all filesystems to the `/mnt`:

<dl><dd>
<pre>
$ <b>mount /dev/nvme0n1p2 /mnt</b>
$ <b>mkdir -p /mnt/boot/efi</b>
$ <b>mount /dev/nvme0n1p1 /mnt/boot/efi</b>
$ <b>swapon /dev/nvme0n1p3</b>
</pre>
</dd></dl>

4. Install essential packages into new filesystem and generate fstab:

<dl><dd>
<pre>
$ <b>pacstrap -i /mnt base linux linux-firmware sudo vim</b>
$ <b>genfstab -U -p /mnt > /mnt/etc/fstab</b>
</pre>
</dd></dl>

### Step 06: Basic configuration of new system

1. Chroot into freshly created filesystem:

<dl><dd>
<pre>
$ <b>arch-chroot /mnt</b>
</pre>
</dd></dl>

2. Setup system locale and timezone, sync hardware clock with system clock:

<dl><dd>
<pre>
$ <b>vim /etc/locale.gen</b>   <i># uncomment your locales, i.e. `en_US.UTF-8` or `en_GB.UTF-8`</i>
$ <b>locale-gen</b>
$ <b>echo "LANG=en_US.UTF-8" > /etc/locale.conf</b>                <i># choose your locale</i>
$ <b>ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime</b>   <i># choose your timezone</i>
$ <b>hwclock --systohc</b>
</pre>
</dd></dl>

3. Setup system hostname:

<dl><dd>
<pre>
$ <b>echo <i>yourhostname</i> > /etc/hostname</b>
$ <b>vim /etc/hosts</b>
    <i>127.0.0.1 localhost</i>
    <i>::1       localhost</i>
    <i>127.0.1.1 yourhostname</i>
</pre>
</dd></dl>

4. Add new users and setup passwords:

<dl><dd>
<pre>
$ <b>useradd -m -G wheel,storage,power,audio,video,docker -s /bin/bash yourusername</i></b>
$ <b>passwd root</b>
$ <b>passwd <i>yourusername</i></b>
</pre>
</dd></dl>

5. Add wheel group to sudoers file to allow users to run sudo:

<dl><dd>
<pre>
$ <b>visudo</b>
    <i>[uncomment following line in file]</i>
    <i>%wheel ALL=(ALL) ALL</i>
</pre>
</dd></dl>

6. Install and configure GRUB:

<dl><dd>
<pre>
$ <b>pacman -S grub efibootmgr</b>
$ <b>grub-install /dev/nvme0n1</b>
$ <b>grub-mkconfig -o /boot/grub/grub.cfg</b>
</pre>
</dd></dl>

7. Setup networking stack:

<dl><dd>
<pre>
$ <b>pacman -S dhcpcd networkmanager resolvconf</b>
$ <b>systemctl enable dhcpcd</b>
$ <b>systemctl enable NetworkManager</b>
$ <b>systemctl enable systemd-resolved</b>
</pre>
</dd></dl>

8. Exit chroot, unmount all disks and reboot:

<dl><dd>
<pre>
$ <b>exit</b>
$ <b>umount /mnt/boot/efi</b>
$ <b>umount /mnt</b>
$ <b>reboot</b>
</pre>
</dd></dl>

<h1 align="center">
    Section 02: Configuring userspace after initial system setup &#127919;
</h1>

1. Activate time syncronization using NTP:

<dl><dd>
<pre>
$ <b>timedatectl set-ntp true</b>
</pre>
</dd></dl>

2. [Optional] Connect to WiFi using `nmcli`:

<dl><dd>
<pre>
$ <b>nmcli device wifi connect &lt;Name of WiFi access point&gt; password &lt;password&gt;</b>
</pre>
</dd></dl>

3. Install X.Org and its utilities:

<dl><dd>
<pre>
$ <b>sudo pacman -S xorg xorg-apps xorg-xinit xdotool xclip xsel</b>
</pre>
</dd></dl>

4. Install a bunch of useful utilities:

<dl><dd>
<pre>
$ <b>sudo pacman -S dbus intel-ucode fuse2 lshw powertop inxi</b>
$ <b>sudo pacman -S base-devel git zip unzip htop tree w3m</b>
$ <b>sudo pacman -S dialog reflector bash-completion arandr</b>
$ <b>sudo pacman -S iw wpa_supplicant</b>
$ <b>sudo pacman -S tcpdump mtr net-tools conntrack-tools ethtool</b>
$ <b>sudo pacman -S wget rsync socat openbsd-netcat axel</b>
$ <b>sudo pacman -S sof-firmware pulseaudio alsa-utils alsa-plugins pavucontrol</b>
</pre>
</dd></dl>

5. Install Xfce4, i3, or both:

<dl><dd>
<pre>
<i># Instructions for installing Xfce4</i>
<div></div>
$ <b>sudo pacman -S xfce4</b>
$ <b>sudo pacman -S xfce4-notifyd xfce4-screensaver xfce4-screenshooter</b>
$ <b>sudo pacman -S thunar-archive-plugin thunar-media-tags-plugin</b>
$ <b>sudo pacman -S network-manager-applet</b>
$ <b>sudo pacman -S xfce4-xkb-plugin         xfce4-battery-plugin \
                 xfce4-datetime-plugin    xfce4-mount-plugin   \
                 xfce4-netload-plugin     xfce4-wavelan-plugin \
                 xfce4-pulseaudio-plugin  xfce4-weather-plugin \
                 xfce4-whiskermenu-plugin</b>
<div></div>
<i># Instructions for installing i3</i>
<div></div>
$ <b>sudo pacman -S i3-wm i3status i3lock pango</b>
$ <b>sudo pacman -S lxappearance</b>
<div></div>
<i># You will most probably need these apps for i3</i>
<div></div>
$ <b>sudo pacman -S polybar</b> \     <i># nice statusbar for i3-based UIs</i>
                 <b>rofi</b> \        <i># like dmenu, but more customizable</i>
                 <b>ranger</b> \      <i># console file manager</i>
                 <b>alacritty</b> \   <i># terminal emulator</i>
                 <b>dunst</b> \       <i># notification manager</i>
                 <b>feh</b> \         <i># fast and light image viewer</i>
                 <b>xss-lock</b> \    <i># screen lock controller</i>
                 <b>light</b> \       <i># utility to control screen brightness</i>
                 <b>flameshot</b> \   <i># screenshot app</i>
                 <b>gsimplecal</b>    <i># small calendar widget</i>
</pre>
</dd></dl>

6. Install login session manager, I prefer `ly` for it's minimalism:

<dl><dd>
<pre>
$ <b>sudo pacman -S ly</b>
$ <b>sudo systemctl enable ly</b>
</pre>
</dd></dl>

7. Install essential system fonts:

<dl><dd>
<pre>
$ <b>sudo pacman -S ttf-dejavu ttf-freefont ttf-liberation ttf-droid terminus-font</b>
$ <b>sudo pacman -S noto-fonts noto-fonts-emoji ttf-ubuntu-font-family ttf-roboto</b>
</pre>
</dd></dl>

8. [Optional] Enable bluetooth support on your PC:

<dl><dd>
<pre>
$ <b>sudo pacman -S bluez bluez-utils blueman</b>
$ <b>sudo systemctl enable bluetooth</b>
</pre>
</dd></dl>

9. [Optional] Enable printing support on your PC:

<dl><dd>
<pre>
$ <b>sudo pacman -S cups cups-filters cups-pdf system-config-printer</b>
$ <b>sudo systemctl enable cups.service</b>
</pre>
</dd></dl>

<dl><dd>
<b>IMPORTANT NOTE</b>: if there is no option for system-config-printer in xfce4-settings-manager,
go to <code>/usr/share/applications/system-config-printer.desktop</code> and set
<code>Categories=System;Settings;X-XFCE-SettingsDialog;X-XFCE-HardwareSettings;</code>
</dd></dl>

10. [Optional] Improve battary usage with TLP - utility that basically does kernel settings
    tweaking that improve power consumption. More information about TLP
    [can be found here](https://linrunner.de/tlp/). More information about TLP-RDW (radio device wizard)
    [can be found here](https://linrunner.de/tlp/settings/rdw.html).

<dl><dd>
<pre>
$ <b>sudo pacman -S tlp tlp-rdw acpi acpi_call</b>
$ <b>sudo systemctl enable tlp</b>
<div></div>
<i># execute following commands only if using TLP-RDW:</i>
<div></div>
$ <b>systemctl enable NetworkManager-dispatcher.service</b>
$ <b>sudo systemctl mask systemd-rfkill.service</b>
$ <b>sudo systemctl mask systemd-rfkill.socket</b>
</pre>
</dd></dl>

11. [Optional] Run service that will discard unused blocks on mounted filesystems. This is useful for
    solid-state drives (SSDs) and thinly-provisioned storage. More information on fstrim
    [can be found here](https://man7.org/linux/man-pages/man8/fstrim.8.html).

<dl><dd>
<pre>
$ <b>sudo systemctl enable fstrim.timer</b>
</pre>
</dd></dl>

12. [Optional] Install GTK themes and icons:

<dl><dd>
<pre>
$ <b>sudo pacman -S arc-gtk-theme adapta-gtk-theme materia-gtk-theme</b>
$ <b>sudo pacman -S papirus-icon-theme</b>
</pre>
</dd></dl>

13. [Optional] Choose fastest pacman mirrors (use your own country list):

<dl><dd>
<pre>
$ <b>sudo reflector --country Germany,Austria,Switzerland \
                 --fastest 10 \
                 --threads $(nproc) \
                 --save /etc/pacman.d/mirrorlist</b>
</pre>
</dd></dl>

14. [Optional] Install NetworkManager addons:

<dl><dd>
<pre>
$ <b>sudo pacman -S nm-connection-editor networkmanager-openvpn</b>
</pre>
</dd></dl>

15. [Optional] Install vulkan drivers:

<dl><dd>
<pre>
$ <b>pacman -S vulkan-intel</b>   <i># only for systems with Intel graphics</i>
$ <b>pacman -S nvidia-utils</b>   <i># only for systems with Nvidia graphics</i>
$ <b>pacman -S amdvlk</b>         <i># only for systems with AMD graphics</i>
</pre>
</dd></dl>

16. Reboot to finalize installation:

<dl><dd>
<pre>
$ <b>reboot</b>
</pre>
</dd></dl>

## Hibernation support

1. Open you `/etc/fstab` and find UUID for your swap partition.

2. Open grub configuration file:

```

sudo vim /etc/default/grub

```

3. Find option `GRUB_CMDLINE_LINUX_DEFAULT="..."`

4. Insert `resume=UUID=<uuid of swap partition from /etc/fstab>`
   (example: `GRUB_CMDLINE_LINUX_DEFAULT='... resume=UUID=17f82588-0b79-419a-954c-4a9a1ee90b70'`)

5. Regenerate grub config:

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

<h1 align="center">
    Section 03: Installing third-party apps and <br>
    setting up dev environment &#129489;&#8205;&#128187;
</h1>
<div align="center">
    <i>These is my personal list of apps and utilities which I use on regular basis, <br>
    so feel free to fork this repo and add something for yourself</i>
</div>

### General

```

sudo pacman -S chromium telegram-desktop discord libreoffice fontforge gparted obs-studio \
 tilix vlc remmina wireshark-qt neofetch evince gimp spotify-launcher \
 shotwell file-roller shotcut inkscape evolution mousepad redshift klavaro

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

sudo pacman -S neovim tree-sitter tree-sitter-cli stow sqlite3 tldr \
 jq tmux openvpn wireguard-tools zip unzip virtualbox \
 nmap masscan pgcli redis ripgrep gitui lazygit \
 gpick apache rclone websocat ansible sshpass meld

sudo setcap 'cap_net_raw+epi' /usr/bin/masscan

# Devops

sudo pacman -S docker docker-compose kubectl helm aws-cli-v2 terraform etcdctl
sudo systemctl enable docker
sudo usermod -a -G docker max
newgrp docker

# Python

sudo pacman -S python-pip python-poetry

# C, C++ and Low Level Tools

sudo pacman -S gcc gdb cmake ninja clang cuda
sudo pacman -S nasm cdrtools qemu-full

# Lua

sudo pacman -S lua

# Golang

sudo pacman -S go
go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest

# Javascript

sudo pacman -S nodejs npm yarn

# Java

sudo pacman -S jdk20-openjdk

# Rust

sudo pacman -S rust

# Virtualbox

sudo pacman -S linux-headers
sudo pacman -S virtualbox-host-dkms
sudo pacman -S virtualbox

# Architecture

sudo pacman -S plantuml
yay -S gaphor

# Network emulation

yay -S gns3-server gns3-gui

# Hugo

sudo pacman -S hugo dart-sass

```

### Install AUR package manager

```

git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si

```

### Install AUR packages

```

# Etcd

yay -S etcd

# Slack

yay -S slack-desktop

# Legacy python versions

yay -S python36 python37 python38 python39 python310

# PCB design

yay -S logisim-evolution qucs-s

```

### Requires manual download

1. Dart / Flutter (https://docs.flutter.dev/get-started/install/linux)
2. Joplin (https://joplinapp.org/help/#installation)

### Install texlive (LaTeX)

1. Donwload texlive installer

```

wget http://mirrors.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz

```

2. Upack archive

```

mkdir ./texlive
tar -xvf install-tl-unx.tar.gz -C texlive --strip-components=1

```

3. Enter `texlive` dir

```

cd ./texlive

```

4. Run install (and select nearest CTAN mirror):

```

sudo ./install-tl -select-repository

```

## Setup Android DevTools

-   Download zip-archive from here: https://developer.android.com/studio
    (_Command line tools only_ section).
-   Run `unzip commandlinetools-linux-..._latest.zip`
-   Run `mkdir -p ~/Android/cmdline-tools/latest`
-   Run `mv ./cmdline-tools/* ~/Android/cmdline-tools/latest/`
-   Run `rmdir cmdline-tools` (downloaded one, not in `~/Android/...`).
-   Set `ANDROID_HOME` environment variable to `$HOME/Andoird`.
-   Run `sdkmanager "platform-tools" "platforms;android-29"`
-   Run `sdkmanager "build-tools;29.0.3"`
-   Run `sdkmanager --licenses`
-   Run `sdkmanager --update`

## Tools for reverse engineering CTF's

Binaries:
`gdb`, `strace`, `ltrace`, `ldd`, `objdump` `radare2`, `frida`,
`Ghidra`, `IDA Pro`, `cutter` + `rz-ghidra` + `cutterref`, `angr-management`
`API Monitor`, `PEiD`, `UpxUnpacker`

Python: `pycdc`

Java: `jd-gui`, `jadx`

C#: `Avalonia ILSpy`

<h1 align="center">
    Section 04: F.A.Q.s and bug fixes for known
    Arch Linux issues &#129714;
</h1>

### Fix broken hibernation

In some Linux kernels there are some broken USB 3.0 device drivers, that _sometimes_ wake up
the system right after you launch hibernation process. If you see errors like this in your
`dmesg` command output after an unsuccessful hibernation:

```

[80136.941050] xhci_hcd 0000:00:14.0: PM: pci_pm_freeze(): hcd_pci_suspend+0x0/0x20 returns -16
[80136.941062] xhci_hcd 0000:00:14.0: PM: dpm_run_callback(): pci_pm_freeze+0x0/0xc0 returns -16
[80136.941073] xhci_hcd 0000:00:14.0: PM: failed to freeze async: error -16

```

Put it in `/usr/lib/systemd/system-sleep/xhci` (file must be executable):

```

#!/bin/sh

run_pre_hook() {
echo "Disable broken xhci module before suspending at $(date)..." >> /tmp/systemd_suspend_log
grep XHC.\*enable /proc/acpi/wakeup && echo XHC > /proc/acpi/wakeup
}

run_post_hook() {
echo "Enable broken xhci module at wakeup from $(date)" >> /tmp/systemd_suspend_log
grep XHC.\*disable /proc/acpi/wakeup && echo XHC > /proc/acpi/wakeup
}

case $1 in
pre) run_pre_hook ;;
post) run_post_hook ;;
esac

```

Original solution: https://gist.github.com/ioggstream/8f380d398aef989ac455b93b92d42048

### Grub resolution fix

_This can help if you have very tiny grub font on your 4k monitor_

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

### Lightdm resolution fix

_This can help if you have very tiny lightdm font on your 4k monitor_

1. Open `/etc/lightdm/lightdm.conf` with text editor.

2. Add following lines to the file:

```

display-setup-script=xrandr --output eDP-1 --mode 1920x1080

```

**Important: place this line in [Seat:\*] section of lightdm.conf file!!!**

P.S. _your screen output name, like eDP-1 in my case, can be found in `xrandr -q`_

### Other small fixes:

-   Activate file-roller dark mode: `gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'`
-   Slack remove annoying menu bar: `Window -> Always show menu bar -> disable`
-   If system goes to sleep after 3-5 minutes, this might be screensaver. To stop this, disable option `Settings -> Screensaver -> Activate Screensaver when computer is idle`
-   Sometimes there is Wireguard VPN perimiter, but no DNS server in your company. If so, use `resolvectl revert wg0` (change `wg0` to your wireguard interface name)
    to prevent sending DNS requests through interface. This command will disable "DefaultRoute" feature of `wg0` interface in systemd-resolved.
-   If you face video freezes (or hangs) while not touching keyboard or mouse for some time (usually 1-10 minutes), this might be an issue with picom. Try disabling it to
    see if this helps. If so, try to change rendering backend of picom from `xrender` to `glx` and check if it helps (worked for me).
