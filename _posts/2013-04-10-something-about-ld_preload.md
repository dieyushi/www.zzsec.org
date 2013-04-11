---
layout: post
title: "LD_PRELOAD的那些事"
description: "something about LD_PRELOAD"
category: coding
tags: [archlinux]
published: 
---
{% include JB/setup %}

### 前言

Sublime text 2在linux下的汉语输入一直是个问题，虽然网上流传着各种方法，但是一直没有一个完美的解决方案。尝试了几种方法，在ArchLinux+Xfce下一直是无效的，最近看到有人在aur上提交了[sublime-text-imfix](https://aur.archlinux.org/packages/sublime-text-imfix),通过LD_PRELOAD重新实现了`gtk_im_context_set_client_window`。这种方法可以成功的使用输入法了。

### 原理

`LD_PRELOAD`可以影响程序运行时的链接，它允许你定义在程序运行前优先加载动态链接库。这个功能主要用来有选择性的载入不同动态链接库中相同函数。通过这个环境变量，我们可以在主程序和其动态链接库的中间加载别的动态链接库，甚至覆盖正常的函数库。一方面，我们可以以此功能来使用自己的或是更好的函数（无需别人的源码），而另一方面，我们也可以以向别人的程序注入恶意程序，从而达到那不可告人的罪恶的目的。

我们知道，Linux的用的都是glibc，有一个叫libc.so.6的文件，这是几乎所有Linux下命令的动态链接中，其中有标准C的各种函数。对于GCC而言，默认情况下，所编译的程序中对标准C函数的链接，都是通过动态链接方式来链接libc.so.6这个函数库的。

<!--more-->

OK,还是让我用一个例子来看一下用LD_PRELOAD来hack别人的程序。我们写一段下面的代码。

``` c
/* 文件名：verifypasswd.c */
/* 这是一段判断用户口令的程序，其中使用到了标准C函数strcmp*/

#include <stdio.h>
#include <string.h>

int main(int argc,char **argv)
{
    char passwd[] = "password";

    if (argc < 2)
    {
        printf("usage: %s <password>\n",argv[0]);
        return;
    }

    if (!strcmp(passwd,argv[1]))
    {
        printf("Correct Password!\n");
        return;
    }

    printf("Invalid Password!\n");
}
```

在上面这段程序中，我们使用了strcmp函数来判断两个字符串是否相等。下面，我们使用一个动态函数库来重载strcmp函数：

``` c
/* 文件名：hack.c */

#include <stdio.h>
#include <string.h>

int strcmp(const char *s1, const char *s2)
{
    printf("hack function invoked. s1= s2=/n", s1, s2);
    /* 永远返回0，表示两个字符串相等 */
    return 0;
}
```

编译程序

     $ gcc -o verifypasswd verifypasswd.c
     $ gcc -shared -o hack.so hack.c

我们可以用 readelf -s hack.so 来查看其导出函数.测试一下程序：（得到正确结果）

    $ ./verifypasswd asdf
    Invalid Password!

设置LD_PRELOAD变量：（使我们重写过的strcmp函数的hack.so成为优先载入链接库）

    $ export LD_PRELOAD="./hack.so"

再次运行程序：

    $ ./verifypasswd asdf
    hack function invoked. s1= s2=
    Correct Password!

我们可以看到，1）我们的hack.so中的strcmp被调用了。2）主程序中运行结果被影响了。如果这是一个系统登录程序，那么这也就意味着我们用任意口令都可以进入系统了。编译时指定使用的动态链接库:

    $ gcc -c hack.c
    $ gcc -shared -fPCI -o libhack.so hack.o
    $ gcc -o verifypasswd verifypasswd.c -L. -lhack -Wl,-rpath,./

再次运行程序:

    $ ./verifypasswd asdf
    hack function invoked. s1= s2=
    Correct Password!

### 总结

动态库的搜索路径搜索的先后顺序是:

1. 编译目标代码时指定的动态库搜索路径
2. 环境变量`LD_LIBRARY_PATH`指定的动态库搜索路径
3. 配置文件`/etc/ld.so.conf`中指定的动态库搜索路径
4. 默认的动态库搜索路径`/lib`
5. 默认的动态库搜索路径`/usr/lib`

在上述1、2、3指定动态库搜索路径时，都可指定多个动态库搜索路径，其搜索的先后顺序是按指定路径的先后顺序搜索的。
