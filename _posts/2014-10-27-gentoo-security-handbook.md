---
layout: post
title: "GNU Gentoo Linux 安全手册"
description: "yuandeyou,袁德优，gentoo 安全手册"
keywords: "yuandeyou,袁德优，gentoo 安全手册"
category: gentoo
tags: [gentoo,security]
datee: "2014-10-27 09:02:16 +0800"
---
{% include JB/setup %}

A 系统安全

* 安装之前要考虑的

1. 要启动的daemon/service
2. 硬盘分区规划
3. root 用户
a. su to root, "su -" root enviroment variables
b. normal user to "wheel" group, PAM
c. sudo
4. 安全政策

<!-- more -->

* 安全加固

1. boot 密码
2. 限制登录控制台个数，/etc/securetty, devfs vc/1, udev tty1
3. 日志Syslog-ng
{% highlight bash  %}
options {
        chain_hostnames(no);

        # The default action of syslog-ng is to log a STATS line
        # to the file every 10 minutes.  That's pretty ugly after a while.
        # Change it to every 12 hours so you get a nice daily update of
        # how many messages syslog-ng missed (0).
        stats_freq(43200);
};

source src {
    unix-stream("/dev/log" max-connections(256));
    internal();
};

source kernsrc { file("/proc/kmsg"); };

# define destinations
destination authlog { file("/var/log/auth.log"); };
destination syslog { file("/var/log/syslog"); };
destination cron { file("/var/log/cron.log"); };
destination daemon { file("/var/log/daemon.log"); };
destination kern { file("/var/log/kern.log"); };
destination lpr { file("/var/log/lpr.log"); };
destination user { file("/var/log/user.log"); };
destination mail { file("/var/log/mail.log"); };

destination mailinfo { file("/var/log/mail.info"); };
destination mailwarn { file("/var/log/mail.warn"); };
destination mailerr { file("/var/log/mail.err"); };

destination newscrit { file("/var/log/news/news.crit"); };
destination newserr { file("/var/log/news/news.err"); };
destination newsnotice { file("/var/log/news/news.notice"); };

destination debug { file("/var/log/debug"); };
destination messages { file("/var/log/messages"); };
destination console { usertty("root"); };

# By default messages are logged to tty12...
destination console_all { file("/dev/tty12"); };

# ...if you intend to use /dev/console for programs like xconsole
# you can comment out the destination line above that references /dev/tty12
# and uncomment the line below.
#destination console_all { file("/dev/console"); };

# create filters
filter f_authpriv { facility(auth, authpriv); };
filter f_syslog { not facility(authpriv, mail); };
filter f_cron { facility(cron); };
filter f_daemon { facility(daemon); };
filter f_kern { facility(kern); };
filter f_lpr { facility(lpr); };
filter f_mail { facility(mail); };
filter f_user { facility(user); };
filter f_debug { not facility(auth, authpriv, news, mail); };
filter f_messages { level(info..warn)
        and not facility(auth, authpriv, mail, news); };
filter f_emergency { level(emerg); };

filter f_info { level(info); };
filter f_notice { level(notice); };
filter f_warn { level(warn); };
filter f_crit { level(crit); };
filter f_err { level(err); };
filter f_failed { message("failed"); };
filter f_denied { message("denied"); };

# connect filter and destination
log { source(src); filter(f_authpriv); destination(authlog); };
log { source(src); filter(f_syslog); destination(syslog); };
log { source(src); filter(f_cron); destination(cron); };
log { source(src); filter(f_daemon); destination(daemon); };
log { source(kernsrc); filter(f_kern); destination(kern); };
log { source(src); filter(f_lpr); destination(lpr); };
log { source(src); filter(f_mail); destination(mail); };
log { source(src); filter(f_user); destination(user); };
log { source(src); filter(f_mail); filter(f_info); destination(mailinfo); };
log { source(src); filter(f_mail); filter(f_warn); destination(mailwarn); };
log { source(src); filter(f_mail); filter(f_err); destination(mailerr); };

log { source(src); filter(f_debug); destination(debug); };
log { source(src); filter(f_messages); destination(messages); };
log { source(src); filter(f_emergency); destination(console); };

# default log
log { source(src); destination(console_all); };
{% endhighlight %}

4. mount partions options

{% highlight bash  %}

    nosuid - Will ignore the SUID bit and make it just like an ordinary file
    noexec - Will prevent execution of files from this partition
    nodev - Ignores devices

/dev/sda1 /boot ext2 noauto,noatime 1 1
/dev/sda2 none swap sw 0 0
/dev/sda3 / reiserfs notail,noatime 0 0
/dev/sda4 /tmp reiserfs notail,noatime,nodev,nosuid,noexec 0 0
/dev/sda5 /var reiserfs notail,noatime,nodev 0 0
/dev/sda6 /home reiserfs notail,noatime,nodev,nosuid 0 0
/dev/cdroms/cdrom0 /mnt/cdrom iso9660 noauto,ro 0 0
proc /proc proc defaults 0 0

{% endhighlight %}

<span class="comment"># Note: I do not set /var to noexec or nosuid, even if files normally are never executed from this mount point. The reason for this is that netqmail is installed in /var/qmail and must be allowed to execute and access one SUID file. I setup /usr in read-only mode since I never write anything there unless I want to update Gentoo. Then I remount the file system in read-write mode, update and remount again.</span>

<span class="comment"># Note: Even if you do not use netqmail, Gentoo still needs the executable bit set on /var/tmp since ebuilds are made here. But an alternative path can be setup if you insist on having /var mounted in noexec mode.</span>

5.1 资源限制: pam limits.conf
 /etc/security/limits.conf is part of the PAM package and will only apply to packages that use PAM.

5.2 资源限制: kernel quotas
内核：File systems->Quota support，**emerge quota**. Then modify your **/etc/fstab** and add usrquota and grpquota to the partitions that you want to restrict disk usage on, like in the example below:

{% highlight bash  %}
/dev/sda1 /boot ext2 noauto,noatime 1 1
/dev/sda2 none swap sw 0 0
/dev/sda3 / reiserfs notail,noatime 0 0
/dev/sda4 /tmp ext3 noatime,nodev,nosuid,noexec,usrjquota=aquota.user,grpjquota=aquota.group,jqfmt=vfsv=1 0 0
/dev/sda5 /var ext3 noatime,nodev,usrjquota=aquota.user,grpjquota=aquota.group,jqfmt=vfsv1 0 0
/dev/sda6 /home ext3 noatime,nodev,nosuid,usrjquota=aquota.user,grpjquota=aquota.group,jqfmt=vfsv1 0 0
/dev/sda7 /usr reiserfs notail,noatime,nodev,ro 0 0
/dev/cdroms/cdrom0 /mnt/cdrom iso9660 noauto,ro 0 0
proc /proc proc defaults 0 0
{% endhighlight %}

Creating the quota files:

{% highlight bash  %}
# touch /tmp/aquota.user
# touch /tmp/aquota.group
# chmod 600 /tmp/aquota.user
# chmod 600 /tmp/aquota.group
# rc-update add quota boot
{% endhighlight %}

The Linux kernel will track the quota usage as the system works. If for any reason the quota files become corrupt or you think the data is wrong, you will need to bring the system in single-user mode (or at least ensure that the file systems are not being actively written to) and then call **quotacheck -avugm**.

{% highlight bash  %}
edquota -u user1 # 为用户设置限制额度
edquota -g group1 # 为用户组设置限制额度
{% endhighlight %}

quota's for user user1:

{% highlight bash  %}
/dev/sda4: blocks in use: 2594, limits (soft = 5000, hard = 6500)
         inodes in use: 356, limits (soft = 1000, hard = 1500)
{% endhighlight %}













