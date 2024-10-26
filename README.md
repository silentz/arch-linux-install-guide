<img src="./logo.png" />

<div align="center">

[![Author](https://img.shields.io/badge/Author-Maxim_Pershin-ff6f00)](https://github.com/silentz)
[![License](https://img.shields.io/badge/License-Apache--2.0-blue)](./LICENSE.txt)
![Last Updated](https://img.shields.io/badge/Last_Updated-September_2024-02b532)

</div>

<h1 align="center">Arch Linux with Xfce4 and i3 Window Manager Installation Guide</h1>

<div align="center">
    <i>How to install Arch Linux with Xfce/i3 and not spend ages on debugging</i>
</div>

### Getting Started

Welcome to the Arch Linux with Xfce4 and i3 Window Manager Installation Guide!

This guide provides you with a step-by-step walkthrough of installing
Arch Linux along with the Xfce4 and i3 window manager. It has been carefully created
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
$ <b>useradd -m -G wheel,storage,power,audio,video -s /bin/bash yourusername</i></b>
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

### Step 01: Basic configuration of userspace

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
$ <b>sudo pacman -S dbus</b>              <i># Message bus used by many applications</i>
$ <b>sudo pacman -S intel-ucode</b>       <i># Microcode update files for Intel CPUs</i>
$ <b>sudo pacman -S fuse2</b>             <i># Interface for programs to export a filesystem to the Linux kernel</i>
$ <b>sudo pacman -S lshw</b>              <i># Provides detailed information on the hardware of the machine</i>
$ <b>sudo pacman -S powertop</b>          <i># A tool to diagnose issues with power consumption and power management</i>
$ <b>sudo pacman -S inxi</b>              <i># Full featured CLI system information tool</i>
$ <b>sudo pacman -S acpi</b>              <i># Client for battery, power, and thermal readings</i>
<div><div/>
$ <b>sudo pacman -S base-devel</b>        <i># Basic tools to build Arch Linux packages</i>
$ <b>sudo pacman -S git</b>               <i># Distributed version control system</i>
$ <b>sudo pacman -S zip</b>               <i># Compressor/archiver for creating and modifying zipfiles</i>
$ <b>sudo pacman -S unzip</b>             <i># For extracting and viewing files in .zip archives</i>
$ <b>sudo pacman -S htop</b>              <i># Interactive CLI process viewer</i>
$ <b>sudo pacman -S tree</b>              <i># A directory listing program</i>
<div><div/>
$ <b>sudo pacman -S dialog</b>            <i># A tool to display dialog boxes from shell scripts</i>
$ <b>sudo pacman -S reflector</b>         <i># Script to retrieve and filter the latest Pacman mirror list</i>
$ <b>sudo pacman -S bash-completion</b>   <i># Programmable completion for the bash shell</i>
<div><div/>
$ <b>sudo pacman -S iw</b>                <i># CLI configuration utility for wireless devices</i>
$ <b>sudo pacman -S wpa_supplicant</b>    <i># A utility providing key negotiation for WPA wireless networks</i>
$ <b>sudo pacman -S tcpdump</b>           <i># Powerful command-line packet analyzer</i>
$ <b>sudo pacman -S mtr</b>               <i># Combines the functionality of traceroute and ping into one tool</i>
$ <b>sudo pacman -S net-tools</b>         <i># Configuration tools for Linux networking</i>
$ <b>sudo pacman -S conntrack-tools</b>   <i># Userspace tools to interact with the Netfilter tracking system</i>
$ <b>sudo pacman -S ethtool</b>           <i># Utility for controlling network drivers and hardware</i>
$ <b>sudo pacman -S wget</b>              <i># Network utility to retrieve files from the Web</i>
$ <b>sudo pacman -S rsync</b>             <i># File copying tool for remote and local files</i>
$ <b>sudo pacman -S socat</b>             <i># Multipurpose socket relay</i>
$ <b>sudo pacman -S openbsd-netcat</b>    <i># Netcat program. OpenBSD variant.</i>
$ <b>sudo pacman -S axel</b>              <i># Light command line download accelerator</i>
$ <b>sudo pacman -S bind</b>              <i># I use dig utility for DNS resolution from this package</i>
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
$ <b>sudo pacman -S polybar</b>      <i># nice statusbar for i3-based UIs</i>
$ <b>sudo pacman -S rofi</b>         <i># like dmenu, but more customizable</i>
$ <b>sudo pacman -S ranger</b>       <i># console file manager</i>
$ <b>sudo pacman -S alacritty</b>    <i># terminal emulator</i>
$ <b>sudo pacman -S dunst</b>        <i># notification manager</i>
$ <b>sudo pacman -S feh</b>          <i># fast and light image viewer</i>
$ <b>sudo pacman -S xss-lock</b>     <i># screen lock controller</i>
$ <b>sudo pacman -S flameshot</b>    <i># screenshot app</i>
$ <b>sudo pacman -S gsimplecal</b>   <i># small calendar widget</i>
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
$ <b>sudo pacman -S noto-fonts noto-fonts-emoji ttf-ubuntu-font-family ttf-roboto ttf-roboto-mono</b>
</pre>
</dd></dl>

8. [Optional] Enable sound support on your PC:

<dl><dd>
<pre>
$ <b>sudo pacman -S sof-firmware</b>    # Sound Open Firmware
$ <b>sudo pacman -S pulseaudio</b>      # A featureful, general-purpose sound server
$ <b>sudo pacman -S pavucontrol</b>     # PulseAudio Volume Control
$ <b>sudo pacman -S alsa-utils</b>      # Advanced Linux Sound Architecture - Utilities
$ <b>sudo pacman -S alsa-plugins</b>    # Additional ALSA plugins
</pre>
</dd></dl>

9. [Optional] Enable bluetooth support on your PC:

<dl><dd>
<pre>
$ <b>sudo pacman -S bluez bluez-utils blueman</b>
$ <b>sudo systemctl enable bluetooth</b>
</pre>
</dd></dl>

10. [Optional] Enable printing support on your PC:

<dl><dd>
<pre>
$ <b>sudo pacman -S cups cups-filters cups-pdf system-config-printer hplip</b>
$ <b>sudo systemctl enable cups.service</b>
</pre>
</dd></dl>

<dl><dd>
<b>IMPORTANT NOTE</b>: if there is no option for system-config-printer in xfce4-settings-manager,
go to <code>/usr/share/applications/system-config-printer.desktop</code> and set
<code>Categories=System;Settings;X-XFCE-SettingsDialog;X-XFCE-HardwareSettings;</code>
</dd></dl>

11. [Optional] Improve battary usage with TLP - utility that basically does kernel settings
    tweaking that improve power consumption. More information about TLP
    [can be found here](https://linrunner.de/tlp/). More information about TLP-RDW (radio device wizard)
    [can be found here](https://linrunner.de/tlp/settings/rdw.html).

<dl><dd>
<pre>
$ <b>sudo pacman -S tlp tlp-rdw</b>
$ <b>sudo systemctl enable tlp</b>
<div></div>
<i># execute following commands only if using TLP-RDW:</i>
<div></div>
$ <b>sudo systemctl enable NetworkManager-dispatcher.service</b>
$ <b>sudo systemctl mask systemd-rfkill.service</b>
$ <b>sudo systemctl mask systemd-rfkill.socket</b>
</pre>
</dd></dl>

12. [Optional] Run service that will discard unused blocks on mounted filesystems. This is useful for
    solid-state drives (SSDs) and thinly-provisioned storage. More information on fstrim
    [can be found here](https://man7.org/linux/man-pages/man8/fstrim.8.html).

<dl><dd>
<pre>
$ <b>sudo systemctl enable fstrim.timer</b>
</pre>
</dd></dl>

13. [Optional] Install GTK themes and icons:

<dl><dd>
<pre>
$ <b>sudo pacman -S arc-gtk-theme adapta-gtk-theme materia-gtk-theme</b>
$ <b>sudo pacman -S papirus-icon-theme</b>
</pre>
</dd></dl>

14. [Optional] Choose fastest pacman mirrors (use your own country list):

<dl><dd>
<pre>
$ <b>sudo reflector --country Germany,Austria,Switzerland \
                 --fastest 10 \
                 --threads $(nproc) \
                 --save /etc/pacman.d/mirrorlist</b>
</pre>
</dd></dl>

15. [Optional] Install NetworkManager addons:

<dl><dd>
<pre>
$ <b>sudo pacman -S nm-connection-editor networkmanager-openvpn</b>
</pre>
</dd></dl>

16. [Optional] Install vulkan drivers:

<dl><dd>
<pre>
$ <b>pacman -S vulkan-intel</b>   <i># only for systems with Intel graphics</i>
$ <b>pacman -S nvidia-utils</b>   <i># only for systems with Nvidia graphics</i>
$ <b>pacman -S amdvlk</b>         <i># only for systems with AMD graphics</i>
</pre>
</dd></dl>

17. Reboot to finalize installation:

<dl><dd>
<pre>
$ <b>reboot</b>
</pre>
</dd></dl>

### Step 02: Enable hibernation support

1. Open your `/etc/fstab` and find UUID for your swap partition

2. Open GRUB configuration file and add resume UUID to `GRUB_CMDLINE_LINUX_DEFAULT`:

<dl><dd>
<pre>
$ <b>sudo vim /etc/default/grub</b>
  <i>Example: </i>
  <i>...</i>
  <i>GRUB_CMDLINE_LINUX_DEFAULT="quiet splash <b>resume=UUID=&lt;UUID of your swap partition&gt;</b>"</i>
  <i>GRUB_CMDLINE_LINUX_DEFAULT="quiet splash <b>resume=UUID=97d9e9f5-899f-4e9e-910e-623a5f665271</b>"</i>
  <i>...</i>
</pre>
</dd></dl>

3. Generate GRUB config:

<dl><dd>
<pre>
$ <b>sudo grub-mkconfig -o /boot/grub/grub.cfg</b>
</pre>
</dd></dl>

4. Open mkinitcpio configuration file and add `resume` hook:

<dl><dd>
<pre>
$ <b>sudo vim /etc/mkinitcpio.conf</b>
  <i>Example: </i>
  <i>...</i>
  <i>HOOKS="base udev <b>resume</b> autodetect modconf block filesystems keyboard fsck"</i>
  <i>...</i>
</pre>
</dd></dl>

5. Generate initramfs:

<dl><dd>
<pre>
$ <b>sudo mkinitcpio -p linux</b>
</pre>
</dd></dl>

6. From now onwards, you can hibernate your system using:

<dl><dd>
<pre>
$ <b>sudo systemctl hibernate</b>
</pre>
</dd></dl>

<h1 align="center">
    Section 03: Installing third-party apps and <br>
    setting up dev environment &#129489;&#8205;&#128187;
</h1>
<div align="center">
    <i>These is my personal list of apps and utilities which I use on regular basis,
    so feel free to fork this repo and add something yours</i>
</div>

### Step 01: General-purpose apps

<dl><dd>
<pre>
$ <b>sudo pacman -S chromium</b>          <i># web-browser</i>
$ <b>sudo pacman -S obsidian</b>          <i># note-taking app</i>
$ <b>sudo pacman -S mousepad</b>          <i># simple graphical text editor</i>
$ <b>sudo pacman -S file-roller</b>       <i># archive manager</i>
$ <b>sudo pacman -S evince</b>            <i># pdf viewer</i>
$ <b>sudo pacman -S libreoffice</b>       <i># office packages</i>
$ <b>sudo pacman -S gimp</b>              <i># image editor</i>
$ <b>sudo pacman -S gpick</b>             <i># color picker</i>
$ <b>sudo pacman -S inkscape</b>          <i># vector graphics editor</i>
$ <b>sudo pacman -S fontforge</b>         <i># fonts editor</i>
$ <b>sudo pacman -S gparted</b>           <i># grphical disk management tool</i>
$ <b>sudo pacman -S vlc</b>               <i># video player</i>
$ <b>sudo pacman -S remmina</b>           <i># remote desktop client</i>
$ <b>sudo pacman -S shotcut</b>           <i># video editing tool</i>
$ <b>sudo pacman -S evolution</b>         <i># email client</i>
$ <b>sudo pacman -S redshift</b>          <i># adjusts the color temperature of your screen</i>
$ <b>sudo pacman -S neofetch</b>          <i># command-line system information</i>
$ <b>sudo pacman -S obs-studio</b>        <i># screencasting and streaming app</i>
$ <b>sudo pacman -S wireshark-qt</b>      <i># network protocol analyzer</i>
$ <b>sudo pacman -S spotify-launcher</b>  <i># spotify client</i>
$ <b>sudo pacman -S telegram-desktop</b>  <i># my preffered messenger</i>
$ <b>sudo pacman -S rclone</b>            <i># manage or migrate files on cloud storage</i>
$ <b>sudo pacman -S openvpn</b>           <i># openvpn client</i>
$ <b>sudo pacman -S wireguard-tools</b>   <i># wireguard client</i>
$ <b>sudo pacman -S arandr</b>            <i># gui for xrandr</i>
</pre>
</dd></dl>

### Step 02: Install package manager for AUR (Arch User Repository)

<dl><dd>
<pre>
$ <b>git clone https://aur.archlinux.org/yay.git</b>
$ <b>cd yay</b>
$ <b>makepkg -si</b>
</pre>
</dd></dl>

### Step 03: Software development tools

1. General purpose development tools:

<dl><dd>
<pre>
$ <b>sudo pacman -S neovim</b>          <i># powerful console editor</i>
$ <b>sudo pacman -S zed</b>             <i># ultimate graphical editor</i>
$ <b>sudo pacman -S tree-sitter</b>     <i># parsing system for programming tools</i>
$ <b>sudo pacman -S tree-sitter-cli</b> <i># cli tool tree-sitter parsers</i>
$ <b>sudo pacman -S stow</b>            <i># configuration manager</i>
$ <b>sudo pacman -S sqlite3</b>         <i># console sqlite client</i>
$ <b>sudo pacman -S tldr</b>            <i># collection of simplified man pages</i>
$ <b>sudo pacman -S jq</b>              <i># cli json processor</i>
$ <b>sudo pacman -S tmux</b>            <i># terminal session multiplexer</i>
$ <b>sudo pacman -S nmap</b>            <i># network scanner with advanced features</i>
$ <b>sudo pacman -S masscan</b>         <i># high performance network scanner</i>
$ <b>sudo pacman -S pgcli</b>           <i># console client for PostgreSQL</i>
$ <b>sudo pacman -S redis</b>           <i># console client for Redis</i>
$ <b>sudo pacman -S apache</b>          <i># http server + some useful utilities (htpasswd)</i>
$ <b>sudo pacman -S meld</b>            <i># git visual diff and merge tool</i>
$ <b>sudo pacman -S websocat</b>        <i># command line client for websockets</i>
$ <b>sudo pacman -S sshpass</b>         <i># noninteractive ssh password provider</i>
</pre>
</dd></dl>

<dl><dd>
<b>IMPORTANT NOTE</b>: execute <code>sudo setcap 'cap_net_raw+epi' /usr/bin/masscan</code> to enable
the ability to run <code>masscan</code> as non-root user.
</dd></dl>

2. Infrastructure as a Code and DevOps tools:

<dl><dd>
<pre>
$ <b>sudo pacman -S ansible</b>          <i># infrastructure as a code tool (bare metal)</i>
$ <b>sudo pacman -S podman</b>           <i># cli tool for container management</i>
$ <b>sudo pacman -S docker</b>           <i># cli tool for container management</i>
$ <b>sudo pacman -S docker-compose</b>   <i># run multi-container applications with docker</i>
$ <b>sudo pacman -S kubectl</b>          <i># cli tool for managing kubernetes clusters</i>
$ <b>sudo pacman -S helm</b>             <i># package manager for kubernetes</i>
$ <b>sudo pacman -S terraform</b>        <i># infrastructure as a code tool (clouds)</i>
<div></div>
<i># configure docker</i>
<div></div>
$ <b>sudo systemctl enable docker</b>            <i># enable docker daemon on system start</i>
# <b>sudo usermod -a -G docker yourusername</b>  <i># to be able to run docker as non-root</i>
$ <b>newgrp docker</b>                           <i># login to docker group without restart</i>
</pre>
</dd></dl>

3. Install Golang and its tools:

<dl><dd>
<pre>
$ <b>sudo pacman -S go</b>
$ <b>go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest</b>
$ <b>go install github.com/hairyhenderson/gomplate/v4/cmd/gomplate@latest</b>
</pre>
</dd></dl>

4. Install Java and its tools:

<dl><dd>
<pre>
$ <b>sudo pacman -S jdk8-openjdk</b>    <i># OpenJDK Java  8 development kit</i>
$ <b>sudo pacman -S jdk11-openjdk</b>   <i># OpenJDK Java 11 development kit</i>
$ <b>sudo pacman -S jdk17-openjdk</b>   <i># OpenJDK Java 17 development kit</i>
$ <b>sudo pacman -S jdk21-openjdk</b>   <i># OpenJDK Java 21 development kit</i>
$ <b>sudo pacman -S jdk-openjdk</b>     <i># OpenJDK Java 22 development kit</i>
$ <b>sudo pacman -S maven</b>           <i># Java project management tool</i>
$ <b>sudo pacman -S gradle</b>          <i># Java project management tool</i>
</pre>
</dd></de>

<dl><dd>
<b>IMPORTANT NOTE</b>: JVM version can be switched using <code>archlinux-java</code>. List all available
JVM versions using <code>archlinux-java status</code> and set one using <code>archlinux-java set VERSION</code>.
</dd></dl>

5. Install Dart and Flutter following instructions from https://docs.flutter.dev/get-started/install/linux

6. Install C, C++ and tools for low-level development:

<dl><dd>
<pre>
$ <b>sudo pacman -S gcc</b>         <i># GNU Compiler Collection, C and C++ frontends</i>
$ <b>sudo pacman -S gdb</b>         <i># GNU Debugger</i>
$ <b>sudo pacman -S clang</b>       <i># C/C++ frontend compiler for LLVM</i>
$ <b>sudo pacman -S cmake</b>       <i># C/C++ project management tool</i>
$ <b>sudo pacman -S ninja</b>       <i># Build system with a focus on speed</i>
$ <b>sudo pacman -S cuda</b>        <i># NVIDIA GPU programming toolkit</i>
$ <b>sudo pacman -S nasm</b>        <i># Asssembler for the x86 CPU architecture</i>
$ <b>sudo pacman -S boost</b>       <i># C++ library with general purpose utils and data structures</i>
$ <b>sudo pacman -S cdrtools</b>    <i># CD/DVD/BluRay command line recording software</i>
$ <b>sudo pacman -S qemu-full</b>   <i># Open source machine emulator and virtualizer</i>
</pre>
</dd></dl>

7. Install Python and its tools:

<dl><dd>
<pre>
$ <b>sudo pacman -S python</b>          <i># python itself</i>
$ <b>sudo pacman -S python-pip</b>      <i># python package manager</i>
$ <b>sudo pacman -S python-poetry</b>   <i># python package manager (better one)</i>
</pre>
</dd></dl>

8. Install Lua:

<dl><dd>
<pre>
$ <b>sudo pacman -S lua</b>       <i># Collection of Lua tools</i>
</pre>
</dd></dl>

9. Install JavaScript and its tools:

<dl><dd>
<pre>
$ <b>sudo pacman -S nodejs</b>    <i># JavaScript runtime</i>
$ <b>sudo pacman -S npm</b>       <i># JavaScript package manager</i>
$ <b>sudo pacman -S yarn</b>      <i># JavaScript package manager</i>
</pre>
</dd></dl>

10. Install Rust and its tools:

<dl><dd>
<pre>
$ <b>sudo pacman -S rust</b>     <i># Rust compiler and tools for project management</i>
</pre>
</dd></dl>

11. Install Virtualbox:

<dl><dd>
<pre>
$ <b>sudo pacman -S linux-headers</b>          <i># Headers for building Linux kernel modules</i>
$ <b>sudo pacman -S virtualbox-host-dkms</b>   <i># VirtualBox Host kernel modules sources</i>
$ <b>sudo pacman -S virtualbox</b>             <i># Hypervisor for x86 virtualization</i>
</pre>
</dd></dl>

12. Architecture diagraming tools:

<dl><dd>
<pre>
$ <b>sudo pacman -S plantuml</b>    <i># Tool for creating UML diagrams</i>
</pre>
</dd></dl>

13. Install hugo (static website generator):

<dl><dd>
<pre>
$ <b>sudo pacman -S hugo</b>        <i># fast and flexible static site generator in go</i>
$ <b>sudo pacman -S dart-sass</b>   <i># implementation of sass (required for hugo)</i>
</pre>
</dd></dl>

14. Accounting software:

<dl><dd>
<pre>
$ <b>sudo pacman -S gnucash</b>   <i># Personal and small-business financial-accounting application</i>
</pre>
</dd></dl>

15. 3D-Printing software:

<dl><dd>
<pre>
$ <b>sudo pacman -S freecad</b>       <i># Feature based parametric 3D CAD modeler</i>
$ <b>sudo pacman -S prusa-slicer</b>  <i># G-code generator for 3D printers</i>
</pre>
</dd></dl>

### Step 04: Install Wine (Windows application runner)

1. Go to `/etc/pacman.conf` and uncomment (or add) following lines:

<dl><dd>
<pre>
<i>[multilib]</i>
<i>Include = /etc/pacman.d/mirrorlist</i>
</pre>
</dd></dl>

2. Update package database:

<dl><dd>
<pre>
$ <b>sudo pacman -Syu</b>
</pre>
</dd></dl>

3. Install Wine and its utilities:

<dl><dd>
<pre>
$ <b>sudo pacman -S wine</b>         <i># Compatibility layer for running Windows programs</i>
$ <b>sudo pacman -S wine-mono</b>    <i># Wine's replacement for Microsoft's .NET Framework</i>
$ <b>sudo pacman -S wine-gecko</b>   <i># Wine's replacement for Microsoft's Internet Explorer</i>
$ <b>sudo pacman -S winetricks</b>   <i># Installer for various runtime libraries in Wine</i>
$ <b>sudo pacman -S zenity</b>       <i># Display dialog boxes from shell scripts (wine dependency)</i>
</pre>
</dd></dl>

4. Configure smooth font in Wine applications:

<dl><dd>
<pre>
$ <b>winetricks settings fontsmooth=rgb</b>
</pre>
</dd></dl>

<dl><dd>
<b>IMPORTANT NOTE</b>: if you are facing error
<code>wine: Read access denied for device L"\\??\\Z:\\", FS volume label and serial are not available</code>,
go to <code>~/.wine/dosdevices</code>, remove <code>z:</code> symbolic link and make it point to your <code>$HOME</code>
</dd></dl>

### Step 05: Install texlive (LaTeX distribution)

1. Donwload texlive installer:

<dl><dd>
<pre>
$ <b>wget http://mirrors.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz</b>
</pre>
</dd></dl>

2. Upack texlive installer archive:

<dl><dd>
<pre>
$ <b>mkdir ./texlive</b>
$ <b>tar -xvf install-tl-unx.tar.gz -C texlive --strip-components=1</b>
</pre>
</dd></dl>

3. Run texlive install and select nearest CTAN mirror:

<dl><dd>
<pre>
$ <b>cd ./texlive</b>
$ <b>sudo ./install-tl -select-repository</b>
</pre>
</dd></dl>

### Step 06: Setup Android development tools

1. Download zip-archive from here: https://developer.android.com/studio from _Command line tools only_ section.

2. Unpack archive and copy cmdline-tools to `$ANDROID_HOME` (in my case `~/Android`):

<dl><dd>
<pre>
$ <b>unzip commandlinetools-linux-..._latest.zip</b>    <i># archive you got from website</i>
$ <b>mkdir -p ~/Android/cmdline-tools/latest</b>
$ <b>mv ./cmdline-tools/* ~/Android/cmdline-tools/latest/</b>
</pre>
</dd></dl>

3. Set `ANDROID_HOME` environment variable to `~/Andoird` in `.bashrc`

4. Install platform tools, build tools and accept all licenses:

<dl><dd>
<pre>
$ <b>sdkmanager "platform-tools" "platforms;android-29"</b>
$ <b>sdkmanager "build-tools;29.0.3"</b>
$ <b>sdkmanager --licenses</b>
$ <b>sdkmanager --update</b>
</pre>
</dd></dl>

### Step 07: Install Yubikey tools

<dl><dd>
<pre>
$ <b>sudo pacman -S yubikey-manager</b>
$ <b>sudo pacman -S yubikey-personalization-gui</b>
</pre>
</dd></dl>

### Bonus: My list of reverse engineering tools

1. Binary reverse engineering:
   `gdb`, `strace`, `ltrace`, `ldd`, `objdump` `radare2`, `frida`,
   `Ghidra`, `IDA Pro`, `cutter` + `rz-ghidra` + `cutterref`, `angr-management`
   `API Monitor`, `PEiD`, `UpxUnpacker`

2. Python: `pycdc`

3. Java: `jd-gui`, `jadx`

4. C#: `Avalonia ILSpy`

<h1 align="center">
    Section 04: F.A.Q.s, bug fixes and other useful tips and playbooks
    for Arch Linux &#129714;
</h1>

### Playbook 01: Fix XHCI hibernation error

In some Linux kernels there are some broken USB 3.0 device drivers, that _sometimes_ wake up
the system right after you launch hibernation process. If you see errors like this in your
`dmesg` command output after an unsuccessful hibernation:

<dl><dd>
<pre>
xhci_hcd 0000:00:14.0: PM: pci_pm_freeze(): hcd_pci_suspend+0x0/0x20 returns -16
xhci_hcd 0000:00:14.0: PM: dpm_run_callback(): pci_pm_freeze+0x0/0xc0 returns -16
xhci_hcd 0000:00:14.0: PM: failed to freeze async: error -16
</pre>
</dd></dl>

To fix the issue put following lines in `/usr/lib/systemd/system-sleep/xhci` and make this file executable:

<dl><dd>
<pre>
#!/bin/sh
<div></div>
run_pre_hook() {
    echo "Disable xhci module before suspend at $(date)..." >> /tmp/systemd_suspend_log
    grep XHC.\*enable /proc/acpi/wakeup && echo XHC > /proc/acpi/wakeup
}
<div></div>
run_post_hook() {
    echo "Enable xhci module after wakeup from $(date)" >> /tmp/systemd_suspend_log
    grep XHC.\*disable /proc/acpi/wakeup && echo XHC > /proc/acpi/wakeup
}
<div></div>
case $1 in
    pre) run_pre_hook ;;
    post) run_post_hook ;;
esac
</pre>
</dd></dl>

Original solution: https://gist.github.com/ioggstream/8f380d398aef989ac455b93b92d42048

### Playbook 02: Fix GRUB screen resolution

_This can help if you have very tiny grub font on your 4k monitor_

1. Open `/etc/default/grub` with text editor and add following lines:

<dl><dd>
<pre>
<i>GRUB_TERMINAL_OUTPUT="gfxterm"</i>
<i>GRUB_GFXPAYLOAD_LINUX=keep</i>
<i>GRUB_GFXMODE=1920x1080x32,1024x768x32,auto</i>
</pre>
</dd></dl>

2. Generate `grub.cfg`:

<dl><dd>
<pre>
$ <b>sudo grub-mkconfig -o /boot/grub/grub.cfg</b>
</pre>
</dd></dl>

### Playbook 03: Fix Lightdm screen resolution

_This can help if you use lightdm and have very tiny font on your 4k monitor_

<dl><dd>
<p>Open <code>/etc/lightdm/lightdm.conf</code> file and add following line under <code>[Seat:\*]</code> section:</p>
<pre>
<i>display-setup-script=xrandr --output eDP-1 --mode 1920x1080</i>
</pre>
P.S. <i>your screen output name, like eDP-1 in my case, can be found in <code>xrandr -q</code></i>
</dd></dl>

### Playbook 04: Activate dark mode in GTK apps

<dl><dd>
<pre>
$ <b>gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'</b>
</pre>
</dd></dl>

### Playbook 05: System goes to sleep too fast with Xfce

-   If system goes to sleep after 3-5 minutes, this might be screensaver.
    To stop this, disable option `Settings -> Screensaver -> Activate Screensaver when computer is idle`.

### Playbook 06: All requests, expept those to internal addresses, fail after launching Wireguard VPN

-   This happens when your Wireguard server can only handle requests only to configured IP addresses and DNS names.
    Use `resolvectl revert wg0` (change `wg0` to your wireguard interface name).
    This will prevent system from using Wireguard interface for all routes.

### Playbook 07: Screen freezed (or hangs) after 2-10 minutes of inactivity when using Picom

-   If you screen freezes (or hangs) while not touching keyboard or mouse for some time (usually 2-10 minutes),
    this might be an issue with picom. Try first stopping picom at all to see if this helps.
    If yes, try to change rendering backend of picom from `xrender` to `glx` and check if it helps.
    Worked for me.

### Playbook 08: Remove annoying menubar from Slack

-   `Window -> Always show menu bar -> disable`

### Playbook 09: Encrypt external disk

1. [Only once] Select disk partition to be encrypted (in this example `/dev/sdb1`) and initialize LUKS:

<dl><dd>
<pre>
$ <b>sudo cryptsetup luksFormat /dev/sdb1</b>
</pre>
</dd></dl>

2. Open and decrypt LUKS partition, this will create decrypted device at `/dev/mapper/cryptdev`:

<dl><dd>
<pre>
$ <b>sudo cryptsetup open /dev/sdb1 <i>cryptdev</i></b>
</pre>
</dd></dl>

3. [Only once] Initialize filesystem on decrypted partition, in this example `ext4`:

<dl><dd>
<pre>
$ <b>sudo mkfs.ext4 /dev/mapper/cryptdev</b>
</pre>
</dd></dl>

4. Mount created filesystem, to `/mnt` folder in this example, and use it as you want:

<dl><dd>
<pre>
$ <b>sudo mount /dev/mapper/<i>cryptdev</i> /mnt</b>
</pre>
</dd></dl>

5. Unmount filesystem and close LUKS device after using it:

<dl><dd>
<pre>
$ <b>sudo umount /mnt</b>
$ <b>sudo cryptsetup close <i>cryptdev</i></b>
</pre>
</dd></dl>
