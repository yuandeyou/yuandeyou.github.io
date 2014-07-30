---
layout: post
title: "GNU Gentoo Linux 安装"
description: "yuandeyou,袁德优，GNU Gentoo Linux 安装, kernel, xorg, emacs, fcitx, awesome, 五笔输入法。通过U盘安装 GNU Gentoo Linux, 配置Xorg、Alsa声卡、Awesome窗口管理器、Fcitx五笔输入法、Urxvt、GNU Emacs以及最新版Firefox浏览器。"
keywords: "yuandeyou,袁德优，gnu, linux, gentoo, emerge, fcitx, emacs, 五笔"
category: Gentoo
tags: [gentoo,install]
datee: "2014-07-25 15:42:13 +0800"
---
{% include JB/setup %}

通过U盘安装 GNU Gentoo Linux, 配置Xorg、Alsa声卡、Awesome窗口管理器、Fcitx五笔输入法、Urxvt、GNU Emacs以及最新版Firefox浏览器。

### 资源下载 ###

GNU Gentoo Linux 安装官方文档：[x86][] [amd64][]。制作 GNU Gentoo Linux USB 启动盘，Linux 系统用 `dd if`， Windows 系统可用 [pendrivelinux][]。

[下载][] `install*.iso` 制作 GNU Gentoo Linux [USB 启动盘][]：
{% highlight bash  %}
wget http://mirrors.163.com/gentoo/releases/x86/current-iso/install-x86-minimal-20140708.iso # x86
wget http://mirrors.xmu.edu.cn/gentoo/releases/amd64/current-iso/install-amd64-minimal-20140717.iso # amd64

dd if=~/install*.iso of=/dev/sdb # sdb 为 U 盘
{% endhighlight %}

<!-- more -->

[x86]: http://www.gentoo.org/doc/en/handbook/handbook-x86.xml?full=1

[amd64]: http://www.gentoo.org/doc/en/handbook/handbook-amd64.xml?full=1

[pendrivelinux]: http://www.pendrivelinux.com/tag/bootable-usb/

[下载]: http://mirrors.163.com/gentoo/releases/ "iso"

[USB 启动盘]: https://wiki.gentoo.org/wiki/LiveUSB/HOWTO

### 网络ssh ###

使用制作好的 GNU Gentoo Linux USB 启动盘启动计算机A，进入系统后，配置网络连接至互联网，开启 ssh 服务供root远程ssh登录安装，方便复制粘贴命令：

{% highlight bash  %}
# 查看网卡接口名称，有线enp2s0，无线wlp3s0
# 无显示，说明需要网卡驱动
ifconfig
# WIFI上网，wep热点，记下此计算机A的ip地址
ifconfig wlp3s0 10.10.10.20 netmask 255.255.255.0 up
iwconfig wlp3s0 key 1234554321
iwconfig wlp3s0 essid GNU
route add default gw 10.10.10.1 # 可选
ping -c 3 8.8.8.8 # 测试互联网连接
links www.gentoo.org

passwd root
/etc/init.d/sshd start
{% endhighlight %}

### 硬盘分区 ###

通过局域网内其他计算机B远程ssh登录要安装 GNU Gentoo Linux 的计算机A：

{% highlight bash  %}
ssh root@10.10.10.20
{% endhighlight %}

如果计算机B是Windows可下载[putty][]以使用ssh，远程登录计算机A后，

{% highlight bash  %}
mkdir install && cd install
{% endhighlight %}


创建 `ibm-t60-parted.txt` 文件，此文件为GPT分区表，不支持非UEFI模式的Windows系统，**如果考虑安装双系统，请改为MBR分区表**，内容如下：

{% highlight bash  %}
mktable gpt yes # 兼容Windows，gpt 改为 msdos
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

进行硬盘GPT分区。

{% highlight bash  %}
parted /dev/sda "mktable gpt yes"
parted /dev/sda < ibm-t60-parted.txt
{% endhighlight %}

保存结果：

{% highlight bash  %}
echo "---parted---" > new-parted-table.txt
parted /dev/sda p >> new-parted-table.txt
{% endhighlight %}

[putty]: http://www.putty.org/

### 文件系统 ###

建立文件系统，除 `/boot` 使用ext2, 其他分区都使用ext4文件系统。

    mkfs.ext2 -O extent -L "boot" /dev/sda2
    mkfs.ext4 -j -O extent -L "root" /dev/sda3
    mkfs.ext4 -j -O extent -L "usr" /dev/sda4
    mkfs.ext4 -j -O extent -L "var" /dev/sda5
    mkfs.ext4 -j -O extent -L "tmp" /dev/sda6
    mkswap /dev/sda7 && swapon /dev/sda7
    mkfs.ext4 -j -O extent -L "home" /dev/sda8
    echo "---mkfs---" >> new-parted-table.txt
    parted /dev/sda p >> new-parted-table.txt

系统分区结果：

{% highlight bash  %}
Model: ATA WDC WD10JPVT-00A (scsi)
Disk /dev/sda: 1000GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system     Name  Flags
 1      1049kB  2097kB  1049kB                  grub  bios_grub
 2      2097kB  99.6MB  97.5MB  ext2            boot  boot
 3      99.6MB  2100MB  2001MB  ext4            root
 4      2100MB  32.1GB  30.0GB  ext4            usr
 5      32.1GB  52.1GB  20.0GB  ext4            var
 6      52.1GB  56.0GB  3901MB  ext4            tmp
 7      56.0GB  60.0GB  3999MB  linux-swap(v1)  swap
 8      60.0GB  1000GB  940GB   ext4            home
{% endhighlight %}

### 基本系统 ###

挂载分区：

    mount /dev/sda3 /mnt/gentoo
    mkdir -pv /mnt/gentoo/{boot,usr,var,tmp,home}
    mount /dev/sda2 /mnt/gentoo/boot
    mount /dev/sda4 /mnt/gentoo/usr
    mount /dev/sda5 /mnt/gentoo/var
    mount /dev/sda6 /mnt/gentoo/tmp
    mount /dev/sda8 /mnt/gentoo/home
    df -h --total > df-output.txt

确认时间以UTC显示正确，不正确需更改：

    date MMDDhhmmYYYY # use UTC，月日时分年
    uname -m # i686

下载 `stage3*` 文件并解压安装至 `/mnt/gentoo` :

    cd /mnt/gentoo && pwd && ls -l ./
	wget http://mirrors.163.com/gentoo/releases/x86/current-iso/stage3-i686-20140708.tar.bz2
    tar xvjpf stage3-*.tar.bz2 # p 保留文件权限
    pwd && ls -l ./

### make.conf ###

根据硬件及需要设置 `make.conf` 文件：

    emacs /mnt/gentoo/etc/portage/make.conf

我的 `/mnt/gentoo/etc/portage/make.conf` 文件内容：

    # /mnt/gentoo/usr/share/portage/config/make.conf.example
    # /usr/portage/profiles/use.desc
    CHOST="i686-pc-linux-gnu"
    CFLAGS="-O2 -march=i686 -pipe"
    CXXFLAGS="${CFLAGS}"
    MAKEOPTS="-j3" # cpu核心数加1
    USE="-gtk -gtk3 -gnome unicode qt4 kde dvd alsa cdr mmx sse sse2 X bindist bluetooth cjk"
	# USE="dbus consolekit" # default for desktop profiles
    PORTAGE_TMPDIR="/var/tmp"
    PORTDIR="/usr/portage"
    DISTDIR="${PORTDIR}/distfiles" # 下载的软件压缩包存放位置
    PKGDIR="${PORTDIR}/packages" # 二进制软件包位置
    PORTDIR_OVERLAY="/usr/local/portage" # 个人或其他人编写的本地软件包位置
    INPUT_DEVICES="synaptics evdev"
    VIDEO_CARDS="intel i915" # Intel Corporation Mobile 945GM/GMS
    ACCEPT_KEYWORDS="x86"
    ACCEPT_LICENSE="* -@EULA"
	# mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
    # mirrorselect -i -r -o >> /mnt/gentoo/etc/portage/make.conf
    GENTOO_MIRRORS="http://mirrors.xmu.edu.cn/gentoo" # 设置同步
    SYNC="rsync://rsync2.cn.gentoo.org/gentoo-portage" # 设置同步

设置同步，如果在 `make.conf` 手动填了 gentoo_mirrors、sync，就可跳过:

    mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
    mirrorselect -i -r -o >> /mnt/gentoo/etc/portage/make.conf

设置系统DNS服务器：

    nano -w /mnt/gentoo/etc/resolv.conf

`/mnt/gentoo/etc/resolv.conf` 内容：

    nameserver 8.8.8.8
    nameserver 208.67.222.222

### chroot ###

    mount -t proc proc /mnt/gentoo/proc
    mount --rbind /sys /mnt/gentoo/sys
    mount --rbind /dev /mnt/gentoo/dev
    chroot /mnt/gentoo /bin/bash
    source /etc/profile
    export PS1="(chroot) $PS1"
    pwd && ls -l

创建并更新 `/usr/portage` 目录：

    emerge-webrsync # create /usr/portage
    emerge --sync --quiet # update portage tree

### profile ###

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
	  Available profile symlink targets:
	  [1]   default/linux/x86/13.0
	  [2]   default/linux/x86/13.0/selinux
	  [3]   default/linux/x86/13.0/desktop *
	  [4]   default/linux/x86/13.0/desktop/gnome
	  [5]   default/linux/x86/13.0/desktop/gnome/systemd
	  [6]   default/linux/x86/13.0/desktop/kde
	  [7]   default/linux/x86/13.0/desktop/kde/systemd
	  [8]   default/linux/x86/13.0/developer
	  [9]   hardened/linux/x86
	  [10]  hardened/linux/x86/selinux
	  [11]  hardened/linux/uclibc/x86
	  [12]  hardened/linux/musl/x86
    eselect news read/list/purge # 阅读、列出、删除 News

### 本地化 ###

产生 `/etc/timezone` 、`/etc/localtime` 、`/etc/locale.gen` 及 `/etc/env.d/02locale` 文件。

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
    eselect locale set 3 # 或直接编辑文件 /etc/env.d/02locale 为：
	  LANG="en_US.utf8" # 系统语言英文，utf8编码
	  LC_CTYPE="zh_CN.utf8" # fcitx 五笔中文输入法的需要，局部设置中文
    env-update && source /etc/profile

### 编译内核 ###

    emerge gentoo-sources && ls -l /usr/src/linux
    cd /usr/src/linux
    make menuconfig # 通过菜单界面调整修改linux内核选项，配置保存为.config

查看硬件信息：

    lspci
    cat /proc/cpuinfo

配置linux内核 `.config`主要考虑的选项：

    1. cpu
    processor family
	SMP support
      vendor_id       : GenuineIntel
      cpu family      : 6
      model           : 14
      model name      : Genuine Intel(R) CPU           T2400  @ 1.83GHz
    2. file systems
	   ext2 ext4
	   devtmpfs support
    3. PPPoE necessary drivers
    4. Bluetooth subsystem support
    5. USB Support:
	USB controller: Intel Corporation NM10/ICH7 Family USB UHCI Controller #1 (rev 02)
    6. PCMCIA card
    7. 显卡 VGA compatible controller:
	Intel Corporation Mobile 945GM/GMS, 943/940GML Express Integrated Graphics Controller (rev 03)
    8. 声卡 Audio device:
	Intel Corporation NM10/ICH7 Family High Definition Audio Controller (rev 02)
    9. 硬盘SATA controller: Intel Corporation 82801GBM/GHM (ICH7-M Family) SATA Controller [AHCI mode] (rev 02)
    10. 有线网卡 Ethernet controller: Intel Corporation 82573L Gigabit Ethernet Controller
    11. 无线网卡 Ethernet controller: Atheros Communications Inc. AR5212 802.11abg NIC (rev 01)
    12. 网卡 Open VPN: Universal TUN/TAP device driver support Open VPN

硬件驱动既可以编译进内核，也可以编译成模块（根据实际需要动态加载 modprobe module）。高度定制的话，可全部编译进内核。编译内核并安装内核模块至 `/lib/modules/内核版本/` ：

    make && make modules_install
    make install
    ls /boot

如果 `/boot /usr /var` 等使用了独立的分区，那么需要创建 initramfs 文件：

    emerge genkernel && genkernel --install initramfs # /usr or /var are on separate partitions
	ls -l /boot/initramfs* 
	-rw-r--r-- 1 root root 1290044 Jul 19 14:05 /boot/initramfs-genkernel-x86-3.12.21-gentoo-r1

选择要开机加载的内核模块：

    find /lib/modules/3.12.21-gentoo-r1/ -type f -iname '*.o' -or -iname '*.ko' | less
    nano -w /etc/conf.d/modules
      # file:/etc/conf.d/modules
      # modules_2_6="3c59x"
	  # modules="3c59x"

### fstab ###

    nano -w /etc/fstab

**注意 `/boot` 虽然是ext2，但挂载选项仍使用ext4，不然启动时会出现错误。**file:`/etc/fstab`:

    /dev/sda2   /boot        ext4    defaults,noatime     0 2
    /dev/sda7   none         swap    sw                   0 0
    /dev/sda3   /            ext4    noatime              0 1
    /dev/sda4   /usr         ext4    defaults,noatime     0 2
    /dev/sda5   /var         ext4    defaults,noatime     0 2
    /dev/sda6   /tmp         ext4    defaults,noatime     0 2
    /dev/sda8   /home        ext4    defaults,noatime     0 2
    /dev/cdrom  /mnt/cdrom   auto    noauto,user          0 0

编辑 `/etc/conf.d/hostname`:

    nano -w /etc/conf.d/hostname
      hostname="ydyby"

配置网络连接，

    # /usr/share/doc/netifrc-*/net.example.bz2
    emerge --noreplace netifrc
    ifconfig
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

编辑 `/etc/conf.d/net`：

    nano -w /etc/conf.d/net
	{
    config_eth0="192.168.0.2 netmask 255.255.255.0 brd 192.168.0.255"
    routes_eth0="default via 192.168.0.1"
    #
    config_enp2s0="dhcp"
    config_eth0="dhcp"
	}

开机自动连接：

    cd /etc/init.d
    ln -s net.lo net.enp2s0
    rc-update add net.enp2s0 default

    nano -w /etc/hosts
	{
    127.0.0.1	ydyby
    127.0.0.1	localhost
	}

查看开机启动的服务：

    nano -w /etc/rc.conf

设置系统时针：

    nano -w /etc/conf.d/hwclock # UTC
	{
    # Set CLOCK to "UTC" if your Hardware Clock is set to UTC (also known as
    # Greenwich Mean Time).  If that clock is set to the local time, then
    # set CLOCK to "local".  Note that if you dual boot with Windows, then
    # you should set it to "local".
    clock="UTC"
	}


### keymaps 交换CAPS与CTRL_L ###

终端控制台 `Ctrl+Alt+Fn` 交换：

    nano -w /etc/conf.d/keymaps
	{
    # http://www.emacswiki.org/emacs/MovingTheCtrlKey#toc12
    keymap="emacs"
    # /etc/init.d/keymaps restart # /usr/share/keymaps/i386/qwerty/emacs.map.gz
	}

X Window System, Xorg 交换：

第一种方式，xkb （推荐），`/etc/X11/xorg.conf` ：

    emacs /etc/X11/xorg.conf
	{
    Section "InputClass"
        Identifier            "Keyboard Setting"
        Driver 		  "evdev"
        MatchIsKeyboard       "yes"
        Option                "XkbOptions" "ctrl:swapcaps"
    EndSection
	}

第二种方式，xmodmap：

    # On Unix-like systems, X11
	nano -w ~/.xmodmap
	{
    !
    ! Swap Caps_Lock and Control_L
    !
    remove Lock = Caps_Lock
    remove Control = Control_L
    keysym Control_L = Caps_Lock
    keysym Caps_Lock = Control_L
    add Lock = Caps_Lock
    add Control = Control_L
	}
	
    emerge xmodmap

	nano -w ~/.xsession
	{
    xmodmap ~/.xmodmap
	}

### fcitx 五笔输入法 ###


{% highlight bash  %}
echo "app-i18n/fcitx autostart pango table" >> /etc/portage/package.use
emerge app-i18n/fcitx
{% endhighlight %}

如果使用 GDM, KDM 或者 LightDM 修改 `~/.xprofile`。如果使用 `startx`, SLiM, 修改 `~/.xinitrc`。内容如下：

{% highlight bash  %}
#File~/.xprofile or ~/.xinitrc
eval "$(dbus-launch --sh-syntax --exit-with-session)"
export XMODIFIERS="@im=fcitx"
export QT_IM_MODULE=fcitx # USE 使用了qt4则=fcitx，否则=xim
export GTK_IM_MODULE=xim # USE 使用了gtk则=fcitx，否则=xim
{% endhighlight %}

xorg 启动时自动启用fcitx:

    echo "fcitx" >> ~/.xinitrc

### 系统常用软件 ###

安装系统必要软件服务： `lspci syslog locate ssh dhcp ppp cronie `

{% highlight bash  %}
emerge sys-apps/pciutils pciutils pcmciautils logrotate syslog-ng mlocate
rc-update add syslog-ng default
rc-update add sshd default
emerge gentoolkit # 提供字体相关软件
 mkfontscale
 mkfontdir
 fc-cache -fv
emerge dhcpcd
emerge ppp
emerge cronie
rc-update add cronie default
{% endhighlight %}

### Xorg ###

#### 安装 ####

[参考][]：

{% highlight bash  %}
echo "x11-base/xorg-server udev" >> /etc/portage/package.use
emerge --ask xorg-server xorg-drivers
env-update && source /etc/profile
{% endhighlight %}

[参考]: https://wiki.gentoo.org/wiki/Xorg/Configuration

#### start X on login ####

{% highlight bash  %}
emacs ~/.bashrc
{
# file:~/.bash_profile or ~/.bashrc. add:
[[ $(tty) = "/dev/tty1" ]] && exec startx
}
{% endhighlight %}

#### awesome 窗口管理器 ####

* dbus consolekit

`eselect profile list` 为 desktop, 或者 `make.conf` 文件手动添加：

    # USE="dbus consolekit" # default for desktop profiles

如果修改了 `make.conf` 中的USE设置，需更新系统：

    emerge --ask --changed-use --deep @world

开机启动dbus consolekit:

    rc-update add dbus default
    rc-update add consolekit default
    emerge --ask awesome

* 开机启动awesome窗口管理器

`file:~/.xinitrc` 文件：

    exec ck-launch-session dbus-launch --sh-syntax --exit-with-session awesome

* [配置][]awesome，使用默认配置文件

        mkdir -p ~/.config/awesome/
        cp /etc/xdg/awesome/rc.lua ~/.config/awesome/rc.lua
        awesome -k # 测试配置文件是否正确

默认快捷键：

    mod4+mouse1 = move client with mouse
    mod4+mouse2 = resize client with mouse

    mod4+enter = open terminal
    mod4+r = run command
    mod4+shift+c = kill
    mod4+m = maximize
    mod4+n = minimize
    mod4+ctrl+n = restore minimized clients
    mod4+f = fullscreen
    mod4+tab = switch to previous client
    mod4+ctrl+space = float

    mod4+j = hilight left client
    mod4+k = hilight right client
    mod4+shift+j = move client right
    mod4+shift+k = move client left

    mod4+l = resize tiled client
    mod4+h = resize tiled client

    mod4+left / right = change tag
    mod4+1-9 = change tag
    mod4+shift+1-9 = send client to tag


[配置]: https://wiki.gentoo.org/wiki/Awesome "awesome 配置参考"


#### Urxvt ####

{% highlight bash  %}
echo "x11-terms/rxvt-unicode 256-color alt-font-width fading-colors focused-urgency font-styles mousewheel perl pixbuf wcwidth xft -vanilla" >> /etc/portage/package.use
emerge --ask rxvt-unicode
{% endhighlight %}

在awesome中使用Urxvt，`~/.config/awesome/rc.lua `：

    terminal = "urxvt"

配置Urxvt，`~/.xinitrc`：

    [[ -f ~/.Xresources ]] && xrdb -merge ~/.Xresources

文件 `~/.Xresources`：

    !Set background color and transparent property
    URxvt.background: rgba:0010/0010/0010/cccc
    URxvt.foreground: #CBCB7A
    
    !Black
    URxvt.color0: #000000
    URxvt.color8: #555753
    !Red
    URxvt.color1: #CC0000
    URxvt.color9: #EF2929
    !Green
    URxvt.color2: #4E9A06
    URxvt.color10: #8AE234
    !Yellow
    URxvt.color3: #C4A000
    URxvt.color11: #FCE94F
    !Blue
    URxvt.color4: #3465A4
    URxvt.color12: #729FCF
    !Magenta
    URxvt.color5: #75507B
    URxvt.color13: #AD7FA8
    !Cyan
    URxvt.color6: #06989A
    URxvt.color14: #34E2E2
    !White
    URxvt.color7: #D3D7CF
    URxvt.color15: #EEEEEC
    
    !Replace these font settings by your favorate
    URxvt*font:     xft:monaco:size=10,xft:Microsoft Yahei:pixelsize=16:antialias=true
    
    URxvt.secondaryScroll: true
    URxvt.scrollbar: False
    
    URxvt.depth:32

### GNU Emacs ###

{% highlight bash  %}
echo "app-editors/emacs gif jpeg png svg xpm tiff alsa dbus gpm inotify sound xft source athena Xaw3d -gtk -motif" >> /etc/portage/package.use
emerge --ask app-editors/emacs
{% endhighlight %}

#### Gentoo GNU Emacs 不能调出 fcitx 输入法的原因 ####

一、缺少75dpi字体，emacs启动时出现:

    > Warning: Cannot convert string "-*-courier-medium-r-*-*-*-120-*-*-*-*-iso8859-*" to

安装字体，重启系统后，即可在emacs中使用fcitx:

{% highlight bash  %}
emerge media-fonts/font-adobe-75dpi media-fonts/wqy-bitmapfont x11-apps/bdftopcf media-fonts/font-alias media-fonts/font-util
{% endhighlight %}

二、`/etc/env.d/02locale` 未能正确设置：

{% highlight bash  %}
LANG="en_US.utf8" # 系统语言英文，utf8编码
LC_CTYPE="zh_CN.utf8" # fcitx 五笔中文输入法的需要，局部设置中文
{% endhighlight %}

### GRUB2 启动引导 ###

    emerge sys-boot/grub
    grub2-install /dev/sda
    grub2-mkconfig -o /boot/grub/grub.cfg

### Firefox 最新版本 ###
文件 `/etc/portage/package.accept_keywords` ：

    www-client/firefox

编译安装firefox，GNU Gentoo Linux firefox 被称为 Aurora ：

    emerge www-client/firefox

安装 adobe flash player，`/etc/portage/package.license`：

    emacs /etc/portage/package.license
    {
    # required by www-plugins/adobe-flash
    >=www-plugins/adobe-flash-11.2.202.394 AdobeFlash-11.x
    }
    emerge www-plugins/adobe-flash

### Alsa 声音 ###

[参考][alsa]：

    emerge media-libs/alsa-lib media-sound/alsa-utils
	/etc/init.d/alsasound start
     * Caching service dependencies ...                                                                                                                                   [ ok ]
     * Restoring Mixer Levels ...
     * No mixer config in /var/lib/alsa/asound.state, you have to unmute your card!                                                                                       [ ok ]
    rc-update add alsasound boot
	 * service alsasound added to runlevel boot
	# 以普通用户身份，unmute 声卡
	$ alsamixer # 按 m 取消禁音
	
[alsa]: http://wiki.gentoo.org/wiki/ALSA "WiKi"

### 重启系统 ###

新建用户：

    passwd root
    useradd -m -G users,wheel,audio,video -s /bin/bash ydy
    passwd ydy

清理文件：

    cd / && rm /stage3-*.tar.bz2*

重启系统：

    exit # 退出 chroot
    cd
    umount -l /mnt/gentoo/dev{/shm,/pts,}
    umount -l /mnt/gentoo{/boot,/proc,/boot,/usr,/var,/tmp,/home}
    reboot

