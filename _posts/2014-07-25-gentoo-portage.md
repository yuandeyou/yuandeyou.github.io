---
layout: post
title: "GNU Gentoo Linux portage 软件包管理"
description: "yuandeyou,袁德优，gentoo portage 的使用"
keywords: "yuandeyou,袁德优，GNU Gentoo Linux, portage"
category: gentoo
tags: [gentoo,portage]
date: 2014-07-25 18:25:39 +0800
---
{% include JB/setup %}

### 介绍 ###

GNU Gentoo Linux 采用 portage 作为软件管理工具，portage 完全由python及bash写成，我们主要通过emerge命令来使用portage。详细信息，查看man手册：

{% highlight bash  %}
man emerge
{% endhighlight %}

> emerge  is  the  definitive command-line interface to the Portage system.  It is primarily used for installing packages, and emerge can automatically handle any
>        dependencies that the desired package has.  emerge can also update the portage tree, making new and  updated  packages  available.   emerge  gracefully  handles
>        updating  installed  packages to newer releases as well.  It handles both source and binary packages, and it can be used to create binary packages for distribu‐
>        tion.

<!-- more -->

### 使用 ###

#### portage 树 ####

所谓 `portage树` 是指位于 `/usr/portage` 目录下的一系列 **ebuilds** 文件的集合。
{% highlight bash  %}
ls -l /usr/portage | head -5
---
total 996
drwxr-xr-x   48 root root     4096 Jul 19 11:33 app-accessibility
drwxr-xr-x  236 root root    12288 Jul 19 11:33 app-admin
drwxr-xr-x    4 root root     4096 Jul 19 11:34 app-antivirus
drwxr-xr-x  104 root root     4096 Jul 19 11:34 app-arch
{% endhighlight %}
**ebuilds** 文件包含了维护软件（install, query, search）所需的全部信息。

##### 更新 portage 树 #####

{% highlight bash  %}
emerge --sync
{% endhighlight %}
如果存在防火墙等致使上面更新失败，可换用以下方式，通过 `portage tree snapshots` 快照的方式更新：
{% highlight bash  %}
emerge-webrsync
{% endhighlight %}
这种方式还有个好处，可以确保系统安全，确保更新的文件都是可信的，都通过了官方验证。为达到这个效果，需进行如下操作：

1. 保存官方 GPG 密钥

        # mkdir -p /etc/portage/gpg
        # chmod 0700 /etc/portage/gpg
        (... Substitute the keys with those mentioned on the release engineering site ...)
        # gpg --homedir /etc/portage/gpg --keyserver subkeys.pgp.net --recv-keys 0xDB6B8C1F96D8BF6D
        # gpg --homedir /etc/portage/gpg --edit-key 0xDB6B8C1F96D8BF6D trust

2. 修改`make.conf`文件：

        FEATURES="webrsync-gpg"
        PORTAGE_GPG_DIR="/etc/portage/gpg"

3. 禁用 `emerge --sync`，只通过 `emerge-webrsync` 更新：

        $ locate repos.conf
        /usr/share/portage/config/repos.conf
        # emacs /usr/share/portage/config/repos.conf
          # Make sure sync-type and sync-uri are commented out
          # sync-type = rsync
          # sync-uri = ...

##### 维护软件 #####

一、查找软件

* 通过`软件名字`查找：

        emerge --search pdf # emerge -s
        ---
        Searching...    
        [ Results for search key : pdf ]
        [ Applications found : 48 ]
        
        *  app-admin/eselect-pdftex
              Latest version available: 0.3
              Latest version installed: [ Not Installed ]
              Size of files: 0 kB
              Homepage:      http://www.gentoo.org/proj/en/eselect/
              Description:   pdftex module for eselect
              License:       GPL-2
        ...

* 通过`关键字`查找，只要软件描述中出现了关键字就会显示，比较耗时：

        emerge --searchdesc pdf # emerge -S
            ---
            ...
            *  x11-apps/whyteboard [ Masked ]
                  Latest version available: 0.41.1
                  Latest version installed: [ Not Installed ]
                  Size of files: 370 kB
                  Homepage:      http://code.google.com/p/whyteboard
                  Description:   A simple image, PDF and postscript file annotator.
                  License:       ISC
            ...

二、安装软件

    emerge packname # 安装packname
    emerge --pretend packname # 查看安装指定packname的同时需要安装的依赖包
    emerge --fetchonly packname # 只下载packname至/usr/portage/distfiles

三、查询软件包安装了哪些文件，包括二进制文件、文档等

* 查看使用了哪些 `USE`，如是否安装"doc"文档，安装的文档通常位于 `/usr/share/doc/`目录下。全局 `/etc/portage/make.conf`，局部 `/etc/portage/package.use` ：

        emerge -pv emacs
        ---
        These are the packages that would be merged, in order:
        
        Calculating dependencies  . .... done!
        [ebuild   R    ] app-editors/emacs-24.3-r6:24  USE="X Xaw3d alsa athena dbus gif gpm jpeg png sound source svg tiff xft xpm (-aqua) -games -gconf -gnutls -gsettings -gtk -gtk3 -gzip-el -hesiod -imagemagick -kerberos -libxml2 -livecd -m17n-lib -motif -pax_kernel (-selinux) -toolkit-scroll-bars -wide-int" 0 kB
        
        Total: 1 package (1 reinstall), Size of downloads: 0 kB
        
         * IMPORTANT: 2 news items need reading for repository 'gentoo'.
         * Use eselect news to read news items.

* 查看软件的全部文件及目录，equery 由 `app-portage/gentoolkit` 提供：

        equery files emacs
        ---
        * Contents of app-editors/emacs-24.3-r6:
        /etc
        /etc/emacs
        /usr
        /usr/bin
        /usr/bin/ctags-emacs-24
        /usr/bin/ebrowse-emacs-24
        /usr/bin/emacs-24
        /usr/bin/emacsclient-emacs-24
        /usr/bin/etags-emacs-24
        /usr/bin/grep-changelog-emacs-24
        /usr/libexec
        /usr/libexec/emacs
        /usr/libexec/emacs/24.3
        /usr/libexec/emacs/24.3/i686-pc-linux-gnu
        /usr/libexec/emacs/24.3/i686-pc-linux-gnu/hexl
        /usr/libexec/emacs/24.3/i686-pc-linux-gnu/movemail
        /usr/libexec/emacs/24.3/i686-pc-linux-gnu/profile
        /usr/libexec/emacs/24.3/i686-pc-linux-gnu/rcs2log
        /usr/libexec/emacs/24.3/i686-pc-linux-gnu/update-game-score
        /usr/share
        /usr/share/doc
        /usr/share/doc/emacs-24.3-r6
        /usr/share/doc/emacs-24.3-r6/BUGS.bz2
        /usr/share/doc/emacs-24.3-r6/README.bz2
        /usr/share/doc/emacs-24.3-r6/README.gentoo.bz2
        /usr/share/emacs
        /usr/share/emacs/24.3
        /usr/share/emacs/24.3/etc
        /usr/share/emacs/24.3/etc/AUTHORS
        ...

四、删除软件

删除软件并不会**警告**该软件是否被其它软件依赖，除非要删除的软件对于系统十分重要，删除后会危害系统正常运行。

    emerge --unmerge packname # 安装软件时因依赖关系而同时装上的依赖包并不会删除，同时软件的配置文件也不会删除，方便重装免配置
    emerge --depclean # 清理依赖包

五、更新系统

* 更新系统 `/var/lib/portage/world` 中的所有软件

        emerge --update --ask @world

* 同时更新依赖包（运行时依赖）

        emerge --update --ask --deep @world

* 更新依赖包，包括编译过程临时要用到的依赖包

        emerge --update --ask --deep --with-bdeps @world

* 如果改动过 `USE`，通过以下命令，安装因新 `USE`导致的新软件包或重新编译受影响的软件包。

        emerge --newuse @world

* 更新整个系统

        emerge --update --ask --deep --with-bdeps --newuse @world

六、元软件包(Metapackages)

像`kde-meta`这样的软件包，用于安装整个kde桌面系统，它只是包含了一些软件清单集合，并不含有软件实际内容。通过 `emerge --unmerge kde-meta` 来删除的话，没有多大效果，所有的依赖包都没有删除。

用删除这样的元软件包，需使用如下命令，以[删除kde][rmkde]为例子：

    emerge --update --deep --newuse @world # 删除前需更新系统，保证正确的依赖关系
    emerge --ask --depclean kde-base/kdelibs $(qlist -IC 'kde-base/*') $(for name in $(qlist -IC | grep -v '^kde-base/') ; do ( qdepends -C $name | grep -q kdelibs ) && echo $name ; done) # 删除kde
    revdep-rebuild # 重建系统的依赖关系，revdep-rebuild is provided by the gentoolkit package

[rmkde]: http://wiki.gentoo.org/wiki/KDE/Removal "彻底删除kde"


#### output ####
       [blocks B ] app-text/dos2unix ("app-text/dos2unix" is blocking app-text/hd2u-0.8.0)
              Dos2unix is Blocking hd2u from being emerged.  Blockers are defined when two packages will clobber each others files, or otherwise  cause  some  form  of
              breakage in your system.  However, blockers usually do not need to be simultaneously emerged because they usually provide the same functionality.

       [ebuild N ] app-games/qstat-25c
              Qstat is New to your system, and will be emerged for the first time.

       [ebuild NS ] dev-libs/glib-2.4.7
              You already have a version of glib installed, but a 'new' version in a different SLOT is available.

       [ebuild R ] sys-apps/sed-4.0.5
              Sed 4.0.5 has already been emerged, but if you run the command, then portage will Re-emerge the specified package (sed in this case).

       [ebuild F ] media-video/realplayer-8-r6
              The  realplayer package requires that you Fetch the sources manually.  When you attempt to emerge the package, if the sources are not found, then portage
              will halt and you will be provided with instructions on how to download the required files.

       [ebuild f ] media-video/realplayer-8-r6
              The realplayer package's files are already downloaded.

       [ebuild U ] net-fs/samba-2.2.8_pre1 [2.2.7a]
              Samba 2.2.7a has already been emerged and can be Updated to version 2.2.8_pre1.

       [ebuild UD] media-libs/libgd-1.8.4 [2.0.11]
              Libgd 2.0.11 is already emerged, but if you run the command, then portage will Downgrade to version 1.8.4 for you.
              This may occur if a newer version of a package has been masked because it is broken or it creates a security risk on your system and a fix has  not  been
              released yet.
              Another  reason  this may occur is if a package you are trying to emerge requires an older version of a package in order to emerge successfully.  In this
              case, libgd 2.x is incompatible with libgd 1.x.  This means that packages that were created with libgd 1.x will not compile with 2.x and  must  downgrade
              libgd first before they can emerge.

       [ebuild U ] sys-devel/distcc-2.16 [2.13-r1] USE="ipv6* -gtk -qt%"
              Here  we  see  that  the make.conf variable USE affects how this package is built.  In this example, ipv6 optional support is enabled and both gtk and qt
              support are disabled.  The asterisk following ipv6 indicates that ipv6 support was disabled the last time this package was installed.  The  percent  sign
              following  qt  indicates  that  the  qt option has been added to the package since it was last installed.  For information about all USE symbols, see the
              --verbose option documentation above.
              *Note: Flags that haven't changed since the last install are only displayed when you use the --pretend and --verbose options.  Using the  --quiet  option
              will prevent all information from being displayed.

       [ebuild r U ] dev-libs/icu-50.1.1:0/50.1.1 [50.1-r2:0/50.1]
              Icu  50.1-r2  has  already  been emerged and can be Updated to version 50.1.1. The r symbol indicates that a sub-slot change (from 50.1 to 50.1.1 in this
              case) will force packages having slot-operator dependencies on it to be rebuilt (as libxml2 will be rebuilt in the next example).

       [ebuild rR ] dev-libs/libxml2-2.9.0-r1:2 USE="icu"
              Libxml2 2.9.0-r1 has already been emerged, but if you run the command, then portage will Re-emerge it in order  to  satisfy  a  slot-operator  dependency
              which forces it to be rebuilt when the icu sub-slot changes (as it changed in the previous example).

       [ebuild U *] sys-apps/portage-2.2.0_alpha6 [2.1.9.25]
              Portage 2.1.9.25 is installed, but if you run the command, then portage will upgrade to version 2.2.0_alpha6. In this case, the * symbol is displayed, in
              order to indicate that version 2.2.0_alpha6 is masked by missing keyword. This type of masking display is disabled by the --quiet option if the --verbose
              option is not enabled simultaneously.  The following symbols are used to indicate various types of masking:

              Symbol   Mask Type
              ──────────────────────────

                #      package.mask
                *      missing keyword
                ~      unstable keyword

              NOTE: The unstable keyword symbol (~) will not be shown in cases in which the corresponding unstable keywords have been accepted globally via ACCEPT_KEY‐
              WORDS.


#### 相关文件，配置及log ####

`/etc/portage/make.conf`:

    ACCEPT_LICENSE="* -@EULA" # 安装除EULA授权外的软件
    ACCEPT_LICENSE="-* @FREE" # 只安装自由软件

`/etc/portage/package.license`:

    app-crypt/truecrypt truecrypt-2.7
    # required by www-plugins/adobe-flash
    >=www-plugins/adobe-flash-11.2.202.394 AdobeFlash-11.x

`/etc/portage/package.use`:

    x11-base/xorg-server udev
    x11-terms/rxvt-unicode 256-color alt-font-width fading-colors focused-urgency font-styles mousewheel perl pixbuf wcwidth xft -vanilla
    app-editors/emacs gif jpeg png svg xpm tiff alsa dbus gpm inotify sound xft source athena Xaw3d -gtk -motif
    net-print/cups -usb
    dev-lang/python sqlite
    app-i18n/fcitx autostart pango table
    net-misc/openvpn iproute2 ssl

`/etc/portage/package.accept_keywords`:

    www-client/firefox

`/etc/portage/package.mask`:

`/etc/portage/package.unmask`:

`/var/log/emerge.log`:
       Contains a log of all emerge output. This file is always appended to, so if you want to clean it, you need to do so manually.

`/var/log/emerge-fetch.log`:
       Contains a log of all the fetches in the previous emerge invocation.

`/var/log/portage/elog/summary.log`:
       Contains the emerge summaries. Installs /etc/logrotate/elog-save-summary.

  

