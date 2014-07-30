---
layout: post
title: "GNU Gentoo Linux 启动过程"
description: "yuandeyou,袁德优，简述GNU Gentoo Linux 的启动过程。"
keywords: "yuandeyou,袁德优，GNU Gentoo Linux, boot, 启动"
category: gentoo
tags: [boot,init,runlevel]
datee: "2014-07-27 09:38:28 +0800"
---
{% include JB/setup %}

### 总体过程 ###


开机，系统引导器 `grub` 载入Linux内核文件，内核启动 `init` 进程，根据 `/etc/inittab` 加载文件系统分区及文件系统，启动相应的 `runlevel`。根据 `/etc/inittab` 文件，通常先 `/etc/runlevel/boot` ，然后是 `/etc/runlevel/default`，接着开启进程 `agetty` 用于启用虚拟终端供用户登录，虚拟终端通过 `Alt+Fn` 切换。

<!-- more -->

### 细节分析 ###

<table class="ntable" border="0" cellpadding="0" cellspacing="0" width="100%">
<tbody>
<tr><td bgcolor="#7a5ada" padding="0px">
<p class="codetitle">文件： /ect/inittab </p>
</td></tr>
<tr><td dir="ltr" align="left" padding="0px"><pre>
<span class="comment"># Default runlevel. 默认运行<strong>级别3</strong></span>
id:3:initdefault:

<span class="comment"># System initialization, mount local filesystems, etc. 挂载文件系统</span>
si::sysinit:/sbin/rc sysinit

<span class="comment"># Further system initialization, brings up the boot runlevel.启动<strong>级别3</strong> /etc/init.d/boot</span>
rc::bootwait:/sbin/rc boot

l0:0:wait:/sbin/rc shutdown 
l0s:0:wait:/sbin/halt -dhp
l1:1:wait:/sbin/rc single
l2:2:wait:/sbin/rc nonetwork
l3:3:wait:/sbin/rc default <span class="comment"># <strong>级别3</strong></span>
l4:4:wait:/sbin/rc default
l5:5:wait:/sbin/rc default
l6:6:wait:/sbin/rc reboot
l6r:6:wait:/sbin/reboot -dk
#z6:6:respawn:/sbin/sulogin

# new-style single-user
su0:S:wait:/sbin/rc single
su1:S:wait:/sbin/sulogin

<span class="comment"># TERMINALS</span>
c1:12345:respawn:/sbin/agetty 38400 tty1 linux
c2:2345:respawn:/sbin/agetty 38400 tty2 linux
c3:2345:respawn:/sbin/agetty 38400 tty3 linux
c4:2345:respawn:/sbin/agetty 38400 tty4 linux
c5:2345:respawn:/sbin/agetty 38400 tty5 linux
c6:2345:respawn:/sbin/agetty 38400 tty6 linux

# SERIAL CONSOLES
#s0:12345:respawn:/sbin/agetty -L 115200 ttyS0 vt100
#s1:12345:respawn:/sbin/agetty -L 115200 ttyS1 vt100

# What to do at the "Three Finger Salute".
ca:12345:ctrlaltdel:/sbin/shutdown -r now

# Used by /etc/init.d/xdm to control DM startup.
# Read the comments in /etc/init.d/xdm for more
# info. Do NOT remove, as this will start nothing
# extra at boot if /etc/init.d/xdm is not added
# to the "default" runlevel.
x:a:once:/etc/X11/startDM.sh
</pre></td></tr>
</tbody></table>

所有启动服务脚本都位于目录 `/etc/init.d/`，是否启用取决于 `/etc/inittab` 及 `/etc/runlevels/`。

    ydy@ydyby ~ $ ls /etc/init.d/ | head -5
    alsasound
    bootmisc
    busybox-ntpd
    busybox-watchdog
    consolefont
    ydy@ydyby ~ $ ls /etc/runlevels/
    boot  default  shutdown  sysinit

In Gentoo, there are seven runlevels defined: three internal runlevels, and four user-defined runlevels. The internal runlevels are called sysinit, shutdown and reboot and do exactly what their names imply: initialize the system, powering off the system and rebooting the system.

The user-defined runlevels are those with an accompanying /etc/runlevels subdirectory: boot, default, nonetwork and single. The boot runlevel starts all system-necessary services which all other runlevels use. The remaining three runlevels differ in what services they start: default is used for day-to-day operations, nonetwork is used in case no network connectivity is required, and single is used when you need to fix the system.

### /etc/init.d/ 服务管理 ###

<table class="ntable" border="0" cellpadding="0" cellspacing="0" width="100%">
<tbody>
<tr><td bgcolor="#7a5ada">
<p class="codetitle">/etc/init.d/ssh {start, stop, restart, zap, status, ineed, iuse, needsme, usesme or broken}</p>
</td></tr>
<tr><td dir="ltr" align="left"><pre>
/etc/init.d/A --nodeps stop <span class="comment"># 停止A服务，但保留依赖A服务的服务，不停止依赖：</span>
/etc/init.d/A status <span class="comment"># 服务状态</span>
/etc/init.d/A zap <span class="comment"># 服务关显示为runnig，但实际没有运行，重置状态为stop</span>
/etc/init.d/A ineed/iuse <span class="comment"># 服务需要的，可以使用的其它服务。可以使用的不一定要用，但需要的一定要有，否则服务无法正常运行</span>
/etc/init.d/A needsme/usesme <span class="comment"># 类同</span>
/etc/init.d/A broken <span class="comment"># 服务需要但又缺失的依赖服务</span>
</pre></td></tr>
</tbody></table>

服务之间有相互依赖关系，比如A服务依赖B服务，那么在启动A之前需启动B，处理这样的工作是枯燥及愚蠢的。我们可以使用 `rc-update` 工具来完成这项工作，将服务添加/删除至指定运行级别。

    rc-update show -v
    rc-update add/del A default # 从default中添加/删除服务A

目录 `/etc/init.d/` 下的init 脚本本身是很复杂的，也会因系统升级而变化，我们一般不会直接去修改这些文件。为了配置服务，为服务指定选项，我们可以为单个服务在目录 `/etc/conf.d/` 中创建用户配置文件。

<table class="ntable" border="0" cellpadding="0" cellspacing="0" width="100%">
<tbody>
<tr><td bgcolor="#7a5ada">
<p class="codetitle">文件： /etc/conf.d/apache2</p>
</td></tr>
<tr><td dir="ltr" align="left"><pre>
APACHE2_OPTS="-D PHP5"
</pre></td></tr>
</tbody></table>

### 新建运行级别示例 ###

{% highlight bash  %}
mkdir /etc/runlevels/offline
(Copy all services from default runlevel to offline runlevel)
# cd /etc/runlevels/default
# for service in *; do rc-update add $service offline; done
(Remove unwanted service from offline runlevel)
# rc-update del net.eth0 offline
(Display active services for offline runlevel)
# rc-update show offline
(Partial sample Output)
               acpid | offline
          domainname | offline
               local | offline
            net.eth0 |
{% endhighlight %}
			
 Even though net.eth0 has been removed from the offline runlevel, udev might want to attempt to start any devices it detects and launch the appropriate services, a functionality that is called hotplugging. By default, Gentoo does not enable hotplugging.

If you do want to enable hotplugging, but only for a selected set of scripts, use the rc_hotplug variable in /etc/rc.conf:

{% highlight bash  %}
emacs /etc/rc.conf
# Allow net.wlan as well as any other service, except those matching net.*
# to be hotplugged
rc_hotplug="net.wlan !net.*"
{% endhighlight %}

修改 `grub` /boot/grub/grub.conf，添加新的启动项:
{% highlight bash  %}
title Gentoo Linux Offline Usage
  root (hd0,0)
  kernel (hd0,0)/kernel-2.4.25 root=/dev/hda3 softlevel=offline
{% endhighlight %}

