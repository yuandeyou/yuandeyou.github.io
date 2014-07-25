---
layout: post
title: "GNU Gentoo Linux portage 软件包管理"
description: "gentoo portage 的使用"
keywords: "GNU Gentoo Linux, portage"
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
