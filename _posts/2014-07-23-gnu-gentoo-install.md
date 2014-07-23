---
layout: post
title: "GNU Gentoo Linux 安装"
description: "GNU Gentoo Linux 安装, kernel, xorg, emacs, fcitx, awesome, 五笔输入法。"
keywords: "gnu, linux, gentoo, emerge, fcitx, emacs, 五笔"
category: Gentoo
tags: [gentoo,install]
---
{% include JB/setup %}

### 资源下载 ###

{% highlight bash  %}
http://www.gentoo.org/doc/en/handbook/handbook-x86.xml?full=1
http://www.pendrivelinux.com/tag/bootable-usb/
wget http://mirrors.163.com/gentoo/releases/x86/current-iso/install-x86-minimal-20140708.iso
wget http://mirrors.xmu.edu.cn/gentoo/releases/amd64/current-iso/install-amd64-minimal-20140717.iso
{% endhighlight %}

制作USB启动盘，windows 系统可用

<!-- more -->

### 网络ssh ###
{% highlight bash  %}
ifconfig wlp3s0 10.10.10.20 netmask 255.255.255.0 up
iwconfig wlp3s0 key 1234554321
iwconfig wlp3s0 essid coolyu_com
ping -c 3 8.8.8.8
links www.gentoo.org
{% endhighlight %}

{% highlight bash  %}
passwd root
/etc/init.d/sshd start
{% endhighlight %}

### 硬盘分区 ###

{% highlight bash  %}
mkdir install && cd install
parted /dev/sda "mktable gpt yes"
parted /dev/sda < ibm-t60-parted.txt
{% endhighlight %}

file:./ibm-t60-parted.txt

{% highlight bash  %}
mktable gpt yes
mkpart primary 1 2
name 1 grub
set 1 bios_grub on
mkpart primary ext2 2 100
name 2 boot
set 2 boot on
mkpart primary ext4 100 2.1G
name 3 root
mkpart primary ext4 2.1G 32.1G
name 4 usr
mkpart primary ext4 32.1G 52.1G
name 5 var
mkpart primary ext4 52.1G 56G
name 6 tmp
mkpart primary ext4 56G 60G
name 7 swap
mkpart primary ext4 60G -1
name 8 home
align-check opt 1
align-check min 1
align-check opt 2
align-check min 2
align-check opt 3
align-check min 3
align-check opt 4
align-check min 4
align-check opt 5
align-check min 5
align-check opt 6
align-check min 6
align-check opt 7
align-check min 7
align-check opt 8
align-check min 8
p
u s p
q
{% endhighlight %}

{% highlight bash  %}
echo "---parted---" > new-parted-table.txt
parted /dev/sda p >> new-parted-table.txt
{% endhighlight %}

### 文件系统 ###

    mkfs.ext2 -O extent -L "boot" /dev/sda2
    mkfs.ext4 -j -O extent -L "root" /dev/sda3
    mkfs.ext4 -j -O extent -L "usr" /dev/sda4
    mkfs.ext4 -j -O extent -L "var" /dev/sda5
    mkfs.ext4 -j -O extent -L "tmp" /dev/sda6
    mkswap /dev/sda7 && swapon /dev/sda7
    mkfs.ext4 -j -O extent -L "home" /dev/sda8
    echo "---mkfs---" >> new-parted-table.txt
    parted /dev/sda p >> new-parted-table.txt

### 基本系统 ###

    mount /dev/sda3 /mnt/gentoo
    mkdir -pv /mnt/gentoo/{boot,usr,var,tmp,home}
    mount /dev/sda2 /mnt/gentoo/boot
    mount /dev/sda4 /mnt/gentoo/usr
    mount /dev/sda5 /mnt/gentoo/var
    mount /dev/sda6 /mnt/gentoo/tmp
    mount /dev/sda8 /mnt/gentoo/home
    df -h --total > df-output.txt

    cd /mnt/gentoo && pwd && ls -l ./
    date MMDDhhmmYYYY # use UTC
    uname -m # i686
    wget http://mirrors.163.com/gentoo/releases/x86/current-iso/stage3-i686-20140708.tar.bz2
    tar xvjpf stage3-*.tar.bz2
    pwd && ls -l ./

### make.conf chroot ###

    nano -w /mnt/gentoo/etc/portage/make.conf

file:/mnt/gentoo/etc/portage/make.conf

    # /mnt/gentoo/usr/share/portage/config/make.conf.example
    # /usr/portage/profiles/use.desc
    CHOST="i686-pc-linux-gnu"
    CFLAGS="-O2 -march=i686 -pipe"
    CXXFLAGS="${CFLAGS}"
    MAKEOPTS="-j3"
    USE="-gtk -gtk3 -gnome unicode qt4 kde dvd alsa cdr mmx sse sse2 X bindist bluetooth cjk"
    PORTAGE_TMPDIR="/var/tmp"
    PORTDIR="/usr/portage"
    DISTDIR="${PORTDIR}/distfiles"
    PKGDIR="${PORTDIR}/packages"
    PORTDIR_OVERLAY="/usr/local/portage"
    INPUT_DEVICES="synaptics evdev"
    VIDEO_CARDS="intel i915"
    ACCEPT_KEYWORDS="x86"
    ACCEPT_LICENSE="* -@EULA"
    GENTOO_MIRRORS="http://mirrors.xmu.edu.cn/gentoo"
    SYNC="rsync://rsync2.cn.gentoo.org/gentoo-portage"


    mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
    mirrorselect -i -r -o >> /mnt/gentoo/etc/portage/make.conf
    nano -w /mnt/gentoo/etc/resolv.conf

{ file:/mnt/gentoo/etc/resolv.conf

    nameserver 211.141.90.68
    nameserver 208.67.222.222
}

    mount -t proc proc /mnt/gentoo/proc
    mount --rbind /sys /mnt/gentoo/sys
    mount --rbind /dev /mnt/gentoo/dev
    chroot /mnt/gentoo /bin/bash
    source /etc/profile
    export PS1="(chroot) $PS1"
    pwd && ls -l .

    emerge-webrsync # create /usr/portage
    emerge --sync --quiet # update portage tree

### profile list ###

    eselect profile list
      [1]   default/linux/x86/13.0 *
      [2]   default/linux/x86/13.0/selinux
      [3]   default/linux/x86/13.0/desktop
      [4]   default/linux/x86/13.0/desktop/gnome
      [5]   default/linux/x86/13.0/desktop/gnome/systemd
      [6]   default/linux/x86/13.0/desktop/kde
      [7]   default/linux/x86/13.0/desktop/kde/systemd
      [8]   default/linux/x86/13.0/developer
      [9]   hardened/linux/x86
      [10]  hardened/linux/x86/selinux
      [11]  hardened/linux/uclibc/x86
      [12]  hardened/linux/musl/x86
    eselect profile set 3
    eselect news read/list/purge

### 本地化 ###

    ls /usr/share/zoneinfo
    echo "Asia/Shanghai" > /etc/timezone
    emerge --config sys-libs/timezone-data # /etc/localtime
    nano -w /etc/locale.gen
    { file:/etc/locale.gen
    en_US.UTF-8 UTF-8
    zh_CN.UTF-8 UTF-8
    }
    locale-gen && locale -a
    eselect locale list
    Available targets for the LANG variable:
      [1]   C
      [2]   POSIX
      [3]   en_US.utf8
      [4]   zh_CN.utf8
      [ ]   (free form)
    eselect locale set 3 # /etc/env.d/02locale
    env-update && source /etc/profile

### 编译内核 ###

    emerge gentoo-sources && ls -l /usr/src/linux
    cd /usr/src/linux
    make menuconfig

{ file:.config

    processor family
      vendor_id       : GenuineIntel
      cpu family      : 6
      model           : 14
      model name      : Genuine Intel(R) CPU           T2400  @ 1.83GHz
      devtmpfs support
    file systems
    PPPoE necessary drivers
    Bluetooth subsystem support
    SMP support
    USB Support: USB controller: Intel Corporation NM10/ICH7 Family USB UHCI Controller #1 (rev 02)
    PCMCIA card
    VGA compatible controller: Intel Corporation Mobile 945GM/GMS, 943/940GML Express Integrated Graphics Controller (rev 03)
    Audio device: Intel Corporation NM10/ICH7 Family High Definition Audio Controller (rev 02)
    SATA controller: Intel Corporation 82801GBM/GHM (ICH7-M Family) SATA Controller [AHCI mode] (rev 02)
    Ethernet controller: Intel Corporation 82573L Gigabit Ethernet Controller
    Ethernet controller: Atheros Communications Inc. AR5212 802.11abg NIC (rev 01)
    Universal TUN/TAP device driver support Open VPN

}

    make && make modules_install
    make install
    ls /boot
    emerge genkernel && genkernel --install initramfs # /usr or /var are on separate partitions
    ls /boot

    find /lib/modules/3.12.21-gentoo-r1/ -type f -iname '*.o' -or -iname '*.ko' | less
    nano -w /etc/conf.d/modules

{ file:/etc/conf.d/modules

    modules_2_6="3c59x"

}

### fstab ###

    nano -w /etc/fstab

{ file:/etc/fstab

    /dev/sda2   /boot        ext4    defaults,noatime     0 2
    /dev/sda7   none         swap    sw                   0 0
    /dev/sda3   /            ext4    noatime              0 1
    /dev/sda4   /usr         ext4    defaults,noatime     0 2
    /dev/sda5   /var         ext4    defaults,noatime     0 2
    /dev/sda6   /tmp         ext4    defaults,noatime     0 2
    /dev/sda8   /home        ext4    defaults,noatime     0 2
    /dev/cdrom  /mnt/cdrom   auto    noauto,user          0 0

}

    nano -w /etc/conf.d/hostname

{

    hostname="ydyby"

}

    # /usr/share/doc/netifrc-*/net.example.bz2
    emerge --noreplace netifrc
    ifconfig

---

    enp2s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.1.102  netmask 255.255.255.0  broadcast 255.255.255.255
            inet6 fe80::216:41ff:fee1:619c  prefixlen 64  scopeid 0x20<link>
            ether 00:16:41:e1:61:9c  txqueuelen 1000  (Ethernet)
            RX packets 805710  bytes 910909466 (868.7 MiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 543150  bytes 221318876 (211.0 MiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
            device interrupt 16  memory 0xee000000-ee020000
    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            inet6 ::1  prefixlen 128  scopeid 0x10<host>
            loop  txqueuelen 0  (Local Loopback)
            RX packets 295  bytes 5414 (5.2 KiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 295  bytes 5414 (5.2 KiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    wlp3s0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
            ether 00:19:7d:cc:fa:30  txqueuelen 1000  (Ethernet)
            RX packets 0  bytes 0 (0.0 B)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 0  bytes 0 (0.0 B)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

---

    nano -w /etc/conf.d/net

{

    config_eth0="192.168.0.2 netmask 255.255.255.0 brd 192.168.0.255"
    routes_eth0="default via 192.168.0.1"
    #
    config_enp2s0="dhcp"
    config_eth0="dhcp"

}

    cd /etc/init.d
    ln -s net.lo net.eth0
    rc-update add net.eth0 default

    nano -w /etc/hosts

{

    127.0.0.1	ydyby
    127.0.0.1	localhost

}

    nano -w /etc/rc.conf
    nano -w /etc/conf.d/hwclock
    nano -w /etc/conf.d/keymaps

{

    # http://www.emacswiki.org/emacs/MovingTheCtrlKey#toc12
    keymap="emacs"
    # /etc/init.d/keymaps restart # /usr/share/keymaps/i386/qwerty/emacs.map.gz

}

    On Unix-like systems, I have a ~/.xmodmap file:

    !
    ! Swap Caps_Lock and Control_L
    !
    remove Lock = Caps_Lock
    remove Control = Control_L
    keysym Control_L = Caps_Lock
    keysym Caps_Lock = Control_L
    add Lock = Caps_Lock
    add Control = Control_L

    emerge xmodmap
    which is sourced from my ~/.xsession with the line:
    xmodmap ~/.xmodmap

# /etc/X11/xorg.conf

    Section "InputClass"
        Identifier            "Keyboard Setting"
        Driver 		  "evdev"
        MatchIsKeyboard       "yes"
        Option                "XkbOptions" "ctrl:swapcaps"
    EndSection

{% highlight bash  %}
echo "app-i18n/fcitx autostart pango table" >> /etc/portage/package.use
emerge app-i18n/fcitx
If you use GDM, KDM or LightDM to login, add them to ~/.xprofile. If you use startx, SLiM, etc. to login, add them to ~/.xinitrc.
 [Collapse] 
File~/.xprofile or ~/.xinitrc
eval "$(dbus-launch --sh-syntax --exit-with-session)"
export XMODIFIERS="@im=fcitx"
export QT_IM_MODULE=fcitx
export GTK_IM_MODULE=xim
{% endhighlight %}

    passwd root
    useradd -m -G users,wheel,audio,video -s /bin/bash ydy
    passwd ydy
{% highlight bash  %}

emerge pciutils pcmciautils logrotate syslog-ng mlocate
rc-update add syslog-ng default
rc-update add sshd default

emerge dhcpcd
emerge ppp
emerge cronie
rc-update add cronie default

emerge sys-boot/grub
grub2-install /dev/sda
grub2-mkconfig -o /boot/grub/grub.cfg

cd / && rm /stage3-*.tar.bz2*

exit
cd
umount -l /mnt/gentoo/dev{/shm,/pts,}
umount -l /mnt/gentoo{/boot,/proc,/boot,/usr,/var,/tmp,/home}
reboot

echo "x11-base/xorg-server udev" >> /etc/portage/package.use
emerge --ask xorg-server xorg-drivers
env-update
source /etc/profile
startx
{
https://wiki.gentoo.org/wiki/Xorg/Configuration
.xinitrc
or echo XSESSION="Xfce4" > /etc/env.d/90xsession
}

{% endhighlight %}

# Start X on login
{% highlight bash  %}
{ file:~/.bash_profile or ~/.bashrc.
[[ $(tty) = "/dev/tty1" ]] && exec startx
}
https://wiki.gentoo.org/wiki/Xorg/Hardware_3D_acceleration_guide

https://wiki.gentoo.org/wiki/Awesome
emerge --ask awesome 
emerge --ask --changed-use --deep @world
rc-update add dbus default
rc-update add consolekit default
{ file:~/.xinitrc
exec ck-launch-session dbus-launch --sh-syntax --exit-with-session awesome
}
mkdir -p ~/.config/awesome/
cp /etc/xdg/awesome/rc.lua ~/.config/awesome/rc.lua
{ file:~/.config/awesome/rc.lua
terminal = "urxvt"
}
awesome -k
echo 'terminal = "urxvt"' >> ~/.config/awesome/rc.lua
echo "x11-terms/rxvt-unicode 256-color alt-font-width fading-colors focused-urgency font-styles mousewheel perl pixbuf wcwidth xft -vanilla" >> /etc/portage/package.use
emerge --ask rxvt-unicode

echo "app-editors/emacs gif jpeg png svg xpm tiff alsa dbus gpm inotify sound xft source athena Xaw3d -gtk -motif" >> /etc/portage/package.use
emerge --ask app-editors/emacs
Warning: Cannot convert string "-*-courier-medium-r-*-*-*-120-*-*-*-*-iso8859-*" to
emerge media-fonts/font-adobe-75dpi x11-apps/bdftopcf media-fonts/font-alias media-fonts/font-util


echo "net-print/cups -usb" >> /etc/portage/package.use
 * Your usb printers will be managed via libusb. In this case,
 * cups-1.7.1 requires the USB_PRINTER support disabled.
 * Please disable it:
 *     CONFIG_USB_PRINTER=n
 * in /usr/src/linux/.config or
 *     Device Drivers --->
 *         USB support  --->
 *             [ ] USB Printer support
 * Alternatively, just disable the usb useflag for cups (your printer will still         work).

echo www-client/firefox >>/etc/portage/package.accept_keywords
echo "dev-lang/python sqlite" >> /etc/portage/package.use
emerge firefox

emacs: Terminal type "dumb" is not powerful enough to run Emacs.
It lacks the ability to position the cursor.
If that is not the actual type of terminal you have,
use the Bourne shell command `TERM=... export TERM' (C-shell:
`setenv TERM ...') to specify the correct type.  It may be necessary
to do `unset TERMINFO' (C-shell: `unsetenv TERMINFO') as well.

emerge xmodmap
emerge  media-fonts/wqy-bitmapfont

cat /etc/X11/xinit/xinitrc.d/100-xinputrc
# !/bin/bash
# This script set the "XIM" and some other environment variable,
# then starts fcitx automatically when loading X

XIM="fcitx"
XIM_PROGRAM="fcitx"
XIM_ARGS="-d"
XMODIFIERS="@im=fcitx"
GTK_IM_MODULE="fcitx"

export XIM XIM_PROGRAM XMODIFIERS GTK_IM_MODULE

# start xim server
$XIM_PROGRAM $XIM_ARGS &
emerge  gentoolkit

mkfontscale
mkfontdir
fc-cache -fv


emerge xfontsel

{% endhighlight %}
