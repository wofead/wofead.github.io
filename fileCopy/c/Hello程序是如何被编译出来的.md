# hello程序的编译

[toc]

## Hello World

hello world程序我们再熟悉不过：

```
/*include head file*/
#include<stdio.h>
/*the main function*/
int main(int argc,char *argv[])
{
    printf("Hello World!\n");
    return 0 ;
}
```

编译并运行：

```
 gcc -o helloWorld helloWorld.c 
 ./helloWorld
 Hello World!
```

整个过程一气呵成，但是实际上上面的过程并非像看起来那么简单。它可以大体分为4个步骤：**预处理，编译，汇编，链接**。接下来我们一一简单介绍这四个步骤做了什么。

## 预处理

预处理主要是处理源代码中以#开头的指令（#pragma 除外），例如本文hello world程序中的#include，预处理之后会将stdio.h的内容插入到预处理指令的位置。
想要只生成预处理之后的内容，可以使用下面的方式：

```
gcc -E -o helloWorld.i helloWorld.c #-E参数表示只进行预处理
```

生成的helloWorld.i即为预处理之后的内容，有兴趣的可以打开文件查看里面的内容，会发现stdio.h的位置被其实际内容所替代。预处理之后，注释内容也会被删除，宏定义会被展开。

## 编译

预处理之后就需要对生成的预处理文件进行词法分析，语法分析，语义分析，最终产生**汇编代码**文件，说白点可以简单理解为将C代码“翻译”成汇编代码。该过程是核心同时也是较复杂的一个过程。我们可以通过命令：

```
gcc -S -o helloWorld.s helloWorld.c #-S参数表示只到生成汇编为止
cat helloWorld.s
    .file   "helloWorld.c"
    .section    .rodata
.LC0:
    .string "Hello World!"
    .text
    .globl  main
    .type   main, @function
main:
.LFB0:
    .cfi_startproc
    pushq   %rbp
    .cfi_def_cfa_offset 16
    .cfi_offset 6, -16
    movq    %rsp, %rbp
    .cfi_def_cfa_register 6
    subq    $16, %rsp
    movl    %edi, -4(%rbp)
    movq    %rsi, -16(%rbp)
    movl    $.LC0, %edi
    call    puts
    movl    $0, %eax
    leave
    .cfi_def_cfa 7, 8
    ret
    .cfi_endproc
.LFE0:
    .size   main, .-main
    .ident  "GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.10) 5.4.0 20160609"
    .section    .note.GNU-stack,"",@progbits
```

上面的内容即为编译之后得到的汇编代码。

## 汇编

汇编是将汇编代码翻译成机器可执行的指令，生成目标文件。整个过程较为简单，几乎只是按照汇编指令和机器指令进行一一翻译。我们可以用下面的命令获得汇编后的内容：

```
gcc  -o  helloWorld.o   -c helloWorld.c
od helloWorld.o  #查看二进制内容
0000000 042577 043114 000402 000001 000000 000000 000000 000000
0000020 000001 000076 000001 000000 000000 000000 000000 000000
0000040 000000 000000 000000 000000 001260 000000 000000 000000
0000060 000000 000000 000100 000000 000000 000100 000015 000012
0000100 044125 162611 101510 010354 076611 044374 072611 137760
（其他内容未显示）
```

## 总结

- 我们总结整个编译过程大致如下：



![图片](https://mmbiz.qpic.cn/mmbiz_png/wdJvTnIClft75VgrsTUbY5zmmW5gDDutAue4bKuhD4yTkx0UERsTQaMK4jYc4iaxBn5ZHY8lnKguAmQPKDf4iagw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



而正是由于整个编译过程分阶段进行，我们可以看到不同类型的问题在不同阶段出现并且有先后顺序。正因如此，链接问题在编译的最后阶段才会出现。

- gcc编译系统本身调用了很多其他相关工具，可以加上--verbose观察其详细编译过程，发现gcc命令调用了预处理器，编译器，汇编器，链接器等命令。