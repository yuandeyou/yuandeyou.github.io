---
layout: post
title: "Gentoo Fcitx Firefox 光标跟随问题"
description: "yuandeyou,袁德优，Gentoo Fcitx Firefox 光标跟随问题"
keywords: "yuandeyou,袁德优，Gentoo Fcitx Firefox 光标跟随问题"
category: gentoo
tags: [fcitx,firefox,问题]
datee: "2014-08-06 18:43:52 +0800"
---
{% include JB/setup %}

在 GNU Gentoo Linux 中安装好 fcitx，一下就遇到两个问题： 1. [emacs 中不能调出](http://www.yuandeyou.org/blog/gentoo/2014-07/gnu-gentoo-install.html)； 2. firefox 中光标不能跟随。原因是 `USE` 没有使用 `gtk` ，结果使用 `xim` 协议，而firefox 采用了gtk，xim 存在 bug，所以光标就不能跟随了。

<!-- more -->

**解决办法：**

文件 `/etc/portage/package.use`:

{% highlight bash  %}
app-i18n/fcitx autostart pango table gtk
{% endhighlight %}

重新编译：

    emerge -pv app-i18n/fcitx

文件 `~/.xinitrc`:
{% highlight bash  %}
[[ -f ~/.Xresources ]] && xrdb -merge ~/.Xresources
urxvtd -q -o -f
export XMODIFIERS="@im=fcitx"
export QT_IM_MODULE=fcitx
export GTK_IM_MODULE=fcitx
# export GTK_IM_MODULE=xim # if USE don't use gtk
fcitx 
exec ck-launch-session dbus-launch --sh-syntax --exit-with-session awesome
{% endhighlight %}

