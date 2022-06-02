+++
title = "Hello World"
outputs = ["Reveal"]
date = 2022-05-29T16:30:51+08:00
draft = false
author = "杨武杰"
categories = [ "演示文稿" ]
tags = [ "Hugo", "C", "Hello Wrold" ]
year = "2022"
month = "2022/05"
+++

{{< slide transition="zoom" >}}

# [Hello World](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program)

in C

及C程序的编译和链接过程

---

hello.c
```C
#include <stdio.h>

int main()
{
	printf("Hello World!\n");
	return 0;
}
```

---

```bash
yang@ubuntu:~$ gcc hello.c -o hello
yang@ubuntu:~$ ./hello
Hello World!
yang@ubuntu:~$
```

- 建造程序：[gcc](https://gcc.gnu.org/) hello.c -o hello
- 运行程序：[./](https://www.computerhope.com/jargon/d/dotslash.htm)hello
- 运行结果：`$ Hello World!`

---

[Why do you need ./ (dot-slash) before executable or script name to run it in bash?](https://stackoverflow.com/questions/6331075/why-do-you-need-dot-slash-before-executable-or-script-name-to-run-it-in-bas)

---

## 建造（Build）过程含多个步骤

1. {{% fragment %}}预处理{{% /fragment %}}
2. {{% fragment %}}编译{{% /fragment %}}
3. {{% fragment %}}链接{{% /fragment %}}

---

{{< slide transition="zoom" >}}

# 预处理

---

## 预处理的主要任务

- {{% fragment %}}纳入头文件（`#include`）{{% /fragment %}}
- {{% fragment %}}宏扩展（`#define`）{{% /fragment %}}
- {{% fragment %}}条件编译（`#ifdef`）{{% /fragment %}}

---

一个虚构的例子
```C
#include "foo.h"

#define BAR_PARAM 7 

int main()
{
	foo();

#ifdef BAR_PARAM
	bar(BAR_PARAM);
#endif

	return 0;
}
```

---

foo.h

```C
void foo();
void bar(int index);
```

---

执行预处理命令
```bash
$ gcc -E main.c
```
结果如下：
```C
# 1 "main.c"
# 1 "<built-in>"
# 1 "<command-line>"
# 31 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 32 "<command-line>" 2
# 1 "main.c"
# 1 "foo.h" 1
void foo();
void bar(int index);
# 2 "main.c" 2



int main()
{
 foo();


 bar(7);


 return 0;
}
```

---

> 请忽略那些色调暗淡的行，那是行号信息，不是源码的一部分。

---

现在把`main.c`中的“`#define BAR_PARAM 7`”行删除
```C
#include "foo.h"

int main()
{
	foo();

#ifdef BAR_PARAM
	bar(BAR_PARAM);
#endif

	return 0;
}
```
猜猜看预处理后会得到什么结果？

---

结果如下：
```C
# 1 "main.c"
# 1 "<built-in>"
# 1 "<command-line>"
# 31 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 32 "<command-line>" 2
# 1 "main.c"
# 1 "foo.h" 1
void foo();
void bar(int index);
# 2 "main.c" 2

int main()
{
 foo();





 return 0;
}
```
请对比一下结果有什么变化，请问你猜对了吗？

---

结论

预处理后得到的仍然是C源程序文件，只不过和程序员原来的程序相比有些变化而已。

---

预处理
```goat {font="12px"}
                    .-------.  
                   |         | 
                   |  .h文件   | 
                   |         | 
                    '---+---'  
                        |
                        v
  .-------.       .-----+-----.        .-------.  
 |         |      |           |       |         | 
 |  .c文件   +----->|   C预处理器   +------>|  C源代码   | 
 |         |      |           |       |         | 
  '-------'       '-----------'        '-------'  

```

---

## 预处理的目的

- 提高程序的可读性
- 方便程序的维护

---

{{< slide transition="zoom" >}}

# 编译

---

执行编译命令
```bash
$ gcc -c main.c -o myapp.o
```
命令完成后会在当前目录生成一个名为`myapp.o`的目标文件（包含目标平台的机器指令）。

---

借助工具[`objdump`](https://www.geeksforgeeks.org/objdump-command-in-linux-with-examples/)对其反汇编后，进行观察：
```bash
$ objdump -d myapp.o
```
```nohighlight
myapp.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <main>:
   0:   f3 0f 1e fa             endbr64 
   4:   55                      push   %rbp
   5:   48 89 e5                mov    %rsp,%rbp
   8:   b8 00 00 00 00          mov    $0x0,%eax
   d:   e8 00 00 00 00          callq  12 <main+0x12>
  12:   b8 00 00 00 00          mov    $0x0,%eax
  17:   5d                      pop    %rbp
  18:   c3                      retq   
```
大致可以看到`main`函数编译后的样子（其中包含一个函数调用指令`callq`）。

---

编译
```goat {font="12px"}
  .-------.       .-----------.        .-------.  
 |         |      |           |       |         | 
 |  .c文件   +----->|    编译器    +------>|  目标代码   | 
 |         |      |           |       |         | 
  '-------'       '-----------'        '-------'  

```

---

{{< slide transition="zoom" >}}

# 链接

---

用`gcc`一次性完成预处理、编译和链接，这是我们通常的用法：
```bash
$ gcc foo.c -o myapp
```

喔，出错了：
```nohighlight
/usr/bin/ld: /tmp/ccqQ3C81.o: in function `main':
main.c:(.text+0xe): undefined reference to `foo'
collect2: error: ld returned 1 exit status
```

---

这是什么意思？

```nohighlight
main.c:(.text+0xe): undefined reference to `foo'
```

---

噢，是因为我们没有编写`main`里调用的`foo`函数。现在创建一个名为`foo.c`的文件，在里面写入如下内容：

```bash
foo.c:
```

```C
void foo()
{
	printf("Function foo is called.\n");
}
```

为了编写`foo`函数，未必要创建新文件。我在这里为了演示怎样编写多个文件组成的程序，创建了一个新文件。

---

建造这个由多个源程序文件（`main.c、foo.c、foo.h`）组成的程序也很简单，一个命令即可搞定：

```bash
$ gcc main.c foo.c -o myapp
```

命令执行后，磁盘上会生成一个名为`myapp`的文件。but...

---

屏幕上显示了这样的信息：

```nohighlight
foo.c: In function ‘foo’:
foo.c:3:2: warning: implicit declaration of function ‘printf’ [-Wimplicit-function-declaration]
    3 |  printf("Function foo is called.\n");
      |  ^~~~~~
foo.c:3:2: warning: incompatible implicit declaration of built-in function ‘printf’
foo.c:1:1: note: include ‘<stdio.h>’ or provide a declaration of ‘printf’
  +++ |+#include <stdio.h>
    1 | void foo()
```

---

原因

`foo`函数调用了`printf`，编译器在编译`foo.c`的时候，它不知道`printf`需要带什么样的参数以及返回什么样的值，所以它就瞎猜了一个。

链接的时候，链接器在运行时库里找到了`printf`，但样式和编译器猜测的不一样，所以它给出了警告，并提示了修正的方法。

---

这种情况在某些编译器下可能会是个错误。在`gcc`下，虽然`myapp`已经生成了，但给出了警告。这终归是个隐患，我们这就根据编译器的提示修正这个警告。在`foo.c`里加一行代码：

```C
#include <stdio.h>

void foo()
{
	printf("Function foo is called.\n");
}
```

然后，重新建造:
```bash
$ gcc main.c foo.c -o myapp
```

---

这次，建造命令执行完毕后，没有显示任何提示信息。没有消息就是好消息。

用如下命令运行我们编写的程序：
```bash
$ ./myapp
```
屏幕上显示了如下信息：
```nohighlight
Function foo is called.
```

---

## `printf`
程序调用的`printf`函数并不是我们编写的，它是一个库函数。

---

## C标准函数库

`printf`函数包含在C标准函数库中。

标准库中包含了程序员经常用到的大量的函数，秉承“不需要重新发明轮子”的原则，开发工具的提供者预先编制好这些函数供程序员使用。

请参考 [C Standard Library](https://en.wikipedia.org/wiki/C_standard_library)

---

链接
```goat {font="12px"}
                     .-------.  
                    |         | 
                    |  运行时库   | 
                    |         | 
                     '---+---'  
                         |
                         v
   .-------.       .-----+-----.        .-------.  
  |         |      |           |       |         | 
  |  目标文件   +----->|    链接器    +------>|  可执行程序  | 
  |         |      |           |       |         | 
   '-------'       '-----------'        '-------'  

```

---

{{< slide transition="zoom" >}}

# [GNU make](https://www.gnu.org/software/make/)

[![GNU logo](246px-The_GNU_logo.png)](https://www.gnu.org/software/make/)

---

如果程序规模较大，包括大量源程序文件，建造程序时，用上面的方法岂不很繁琐？

---

## Makefile

在源程序所在目录创建一个名为`Makefile`的文件，在里面写入如下内容：
```nohighlight
myapp: main.c foo.c foo.h
	gcc main.c foo.c -o myapp
```
请注意，第二行开头有个制表符，不能省略，否则会出错。保存文件，之后需要建造程序时，只需执行一个简单的命令即可：
```bash
$ make
```

---

这个`makefile`简明但不专业。有关[`GNU make`](https://www.gnu.org/software/make/)的更详细信息请参考：
- [Makefile Tutorial](https://makefiletutorial.com/)
- [GNU make](https://www.gnu.org/software/make/manual/make.html)

---

## [IDE(集成开发环境)](https://en.wikipedia.org/wiki/Integrated_development_environment)
多数人更喜欢IDE，这里也推荐几个：
- [Visual Studio](https://visualstudio.microsoft.com/zh-hans/)
- [Visual Studio Code](https://code.visualstudio.com/)
- [Code::Blocks](https://www.codeblocks.org/)
- 其他还有很多，例如：[20+ Best C IDE for Windows, Mac & Linux (2022 Editors)](https://www.guru99.com/best-c-ide.html)

---

你没有看错，`Visual Studio`和`Visual Studio Code`不仅名字相近，实际上他们是同一家公司的产品。

个人推荐[`Visual Studio Code`](https://code.visualstudio.com/)，但它对新手不是很友好......但还是要推荐

[![vscode icon](Visual_Studio_Code_icon.svg)](https://code.visualstudio.com/)

---

## [Why Visual Studio Code?](https://code.visualstudio.com/docs/editor/whyvscode)

[![Visual Studio Code](https://code.visualstudio.com/assets/docs/editor/whyvscode/macwinlinux2.png)](https://code.visualstudio.com/docs/editor/whyvscode)

---

如果你选择vscode，那么这个链接或许对你有用：

[C/C++ for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)

---

{{% section %}}
# Happy Hacking

---

![hacking](what-is-the-purpose-of-hacking.png)

{{% /section %}}

