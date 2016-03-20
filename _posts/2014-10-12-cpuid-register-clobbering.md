---
layout: post
title: "cpuid使用中的一个注意事项"
disqus: y
date: 2014-10-12 13:49:22 +0800
---

针对特定CPU进行一些优化工作时，就需要先对CPU支持的指令集进行判断。一般的做法是使用cpuid指令，然后检查相应的标识位来判断。这部分详见Intel的手册。

在使用中，发现了一个诡异的问题，如果在程序初始化时调用封装了cpuid的函数则正常运行，但是如果每次在使用使用优化函数前调用cpuid则会发生段错误。

挂上gdb启动程序，然后在段错误前si逐指令跟踪。最终确定原因为cpuid会将结果保存到RAX,RBX,RCX,RDX，问题就出到RBX上，根据[Intel的文章](https://software.intel.com/en-us/articles/introduction-to-x64-assembly)，

> - Registers RAX, RCX, RDX, R8, R9, R10, and R11 are considered volatile and must be considered destroyed on function calls.
> - RBX, RBP, RDI, RSI, R12, R14, R14, and R15 must be saved in any function using them.

如果gcc产生的指令使用到了RBX，我们又没有对RBX进行单独的保护，就会出现段错误。正确的调用cpuid方法是在使用cpuid前将RBX寄存器的值保存一下。

```
    movq %rbx, %r8
    movl $1, %eax
    cpuid
    xchgq %rbx, %r8
```