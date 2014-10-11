---
layout: post
title: "linux kernel module version check"
disqus: y
date: 2014-10-11 00:34:14 +0800
---

关于Linux内核模块加载时的版本检查，看了一些资料，记录一下。原文地址：http://www.skynet.ie/~mark/home/kernel/symbols.html

### Exporting Symbols

   By default, any global variables or functions defined in a module are exported to the kernel symbol table when the module is loaded. However, there are ways which you may control which symbols are exported.

   If you only require that none of the module's symbols are exported you can use the EXPORT_NO_SYMBOLS macro.

   If however, you require that only some of your global symbols are exported you will need to use the EXPORT_SYMBOL macro to export it. If CONFIG_MODVERSIONS is turned on a further step is required in the build process, but that will be explained later.

### So How Does This Work?

   A kernel module that explicitly exports symbols will have two special sections in its object file: the symbol table '__ksymtab' and the string table '.kstrtab'. When a symbol is exported by a module using EXPORT_SYMBOL, two things happen:

> - a string, that is either the symbol name or, in the case that CONFIG_MODVERSIONS is turned on, the symbol name with some extra versioning info attached, is defined in the string table.

> - a module_symbol structure is defined in the symbol table. This structure contains a pointer to the symbol itself and a pointer to the entry in the string table.

   When a module is loaded this info is added to the kernels symbol table and these symbols are now treated like any of the kernel's exported symbols.

   To take a peek at your module's symbol table do

	$> objdump --disassemble -j __ksymtab sunrpc.o

   or the string table do

	$> objdump --disassemble -j .kstrtab sunrpc.o

### CONFIG_MODVERSIONS et al.

   CONFIG_MODVERSIONS is a notion thought up to make people's lives easier. In essence, what it is meant to achieve is that if you have a module you can attempt to load that module into any kernel, safe in the knowledge that it will fail to load if any of the kernel data structures, types or functions that the module uses have changed.

   If your kernel is not compiled with CONFIG_MODVERSIONS enabled you will only be able to load modules that were compiled specifically for that kernel version and that were also compiled without MODVERSIONS enabled.

   However, if your kernel is compiled with CONFIG_MODVERSIONS enabled you will be able to load a module that was compiled for the same kernel version with MODVERSIONS turned off. But - here's the important part folks - you will also be able to load any modules compiled with MDOVERSIONS turned on, as long as the kernel API that the module uses hasn't changed.

### So How Does This Work?

   When CONFIG_MODVERSIONS is turned on the kernel then a special piece of versioning info is appended to every symbol exported using EXPORT_SYMBOL.

   This versioning info is calculated using the genksyms command whose man page has this to say about how the info is calculated :

> When a symbol table is found in the source, the symbol will be expanded to its full definition, where  all  struct's,  unions, enums  and  typedefs will be expanded down to their basic part, recursively.  This final string will then be used as input to a CRC algorithm that will give an integer that will change as soon as any of the included definitions changes, for this symbol.

> The  version information in the kernel normally looks like: symbol_R12345678, where 12345678 is the hexadecimal representation of the CRC. representation of the CRC.

   What this means is that the versioning info is calculated in such way as that it will only change when the definition of that symbol changes.

   The versioning string is 'appended' by the use of a #define in linux/modversions.h for every exported symbol. The #define usually winds up looking something like this (simplified):

	#define printk printk_R1b7d4074

   What this does is effectively get rid of the function 'printk' - alas, poor printk - and replace it with the much more handsome 'printk_R1b7d4074'.

   If you have a look at linux/modversions.h you'll notice that it just includes loads of .ver files. These are generated using a command similar to

	$> gcc -E -D__GENKSYMS__ ${c-file} | genksyms -k ${kernel-ver} > ${ver-file}

   Notice that the c file if first passed through the c preprocessor before being passed to genksyms. This is to collapse all macros and stuff beforehand.

### What does this mean for modules?

   When modules are being compiled for a kernel with CONFIG_MODVERSIONS turned on, the header file linux/modversions.h must be included at the top of every c file. This you be an awful pain to do, so we just do it with the gcc flag '-include'.

       ifdef CONFIG_MODULES
       ifdef CONFIG_MODVERSIONS
       MODFLAGS += -DMODVERSIONS -include $(HPATH)/linux/modversions.h
       endif

   The extra MODVERSIONS flag is used to indicate that this is a module being compiled with CONFIG_MODVERSIONS turned on as opposed to the kernel being compiled with CONFIG_MODVERSIONS enabled.
