+++
title = "内存布局"
outputs = ["Reveal"]
date = 2022-05-30T17:54:23+08:00
draft = false
author = "杨武杰"
categories = [ "演示文稿" ]
tags = [ "C", "Memory" ]
year = "2022"
month = "2022/05"
+++

{{< slide transition="zoom" >}}

C语言程序的
# 内存布局

---

- `memory-layout.c:`一个极简C程序框架
```C
int main()
{
	return 0;
}
```

---

{{< slide transition="zoom" >}}

# 数据段和代码段

---

- 用`gcc`建造该程序，并用GNU工具`size`查看可执行程序中各部分的大小：

```bash
$ gcc memory-layout.c -o memory-layout
$ size memory-layout
```

```nohighlight
   text    data     bss     dec     hex filename
   1418     544       8    1970     7b2 memory-layout
```

---

- 给程序增加一个全局变量：
```C {linenos=inline,hl_lines=1}
double global;

int main()
{
	return 0;
}
```

---

```bash
$ gcc memory-layout.c -o memory-layout
$ size memory-layout
```
```nohighlight
   text    data     bss     dec     hex filename
   1418     544      16    1978     7ba memory-layout
```

- 再次查看，比较两次的数据差异可知，变量`global`位于`bss`段。

---

## [bss](https://en.wikipedia.org/wiki/.bss)

- 这是一个被沿用的古老的名字。
- 是block starting symbol的缩写。
- 其中的数据未被初始化，所以在目标文件中只记录大小，但并不占用空间。
- 程序运行时，在内存为bss中的数据分配空间，默认值为0。

---

## 问题
- 请想办法验证bss中的数据在目标文件中真的不占用空间，并说明为什么可以这样？

---

- 给程序增加两个静态变量：
```C {linenos=inline,hl_lines=5}
double global;

int main()
{
	static int i, j;
	return 0;
}
```

---

```bash
$ gcc memory-layout.c -o memory-layout
$ size memory-layout
```
```nohighlight
   text    data     bss     dec     hex filename
   1418     544      24    1986     7c2 memory-layout
```

- 再次查看，比较两次的数据差异可知，静态变量亦位于`bss`段。
- 这里之所以加了两个静态变量是因为在64位系统中，数据通常是8字节对齐的，单个4字节的整形变量可能不足以导致数字上可观察的变化。

---

- 给全局变量赋初值：
```C {linenos=inline,hl_lines=1}
double global = 7;

int main()
{
	static int i, j;
	return 0;
}
```

---

```bash
$ gcc memory-layout.c -o memory-layout
$ size memory-layout
```
```nohighlight
   text    data     bss     dec     hex filename
   1418     552      16    1986     7c2 memory-layout
```

- 再次查看，发现`bss`段便小了，`data`段变大了
- 由此可知，变量`global`从`bss`段移到了`data`段

---

- 再增加一个全局变量：
```C {linenos=inline,hl_lines=1}
char *greetings = "Hello World!";
double global = 7;

int main()
{
	static int i, j;
	return 0;
}
```

---

```bash
$ gcc memory-layout.c -o memory-layout
$ size memory-layout
```
```nohighlight
   text    data     bss     dec     hex filename
   1455     560      16    2031     7ef memory-layout
```

- 再次查看，发现`data`段变大了，这是因为增加了全局变量`greetings`的结果
- 但`text`也增加了。为什么？

---

原因
- `greetings`的数据类型是字符指针，变量本身位于`data`段。
- `greetings`指向的目标，即字符串内容本身被放在`text`段。
- 所以，`data`段和`text`段同时增大了。
- 程序指令也处于`text`段。
- 和`data`段不同，`text`段只允许读，不允许写。

---

```goat
      data  |                  |
            .------------------.          
        .---+--------*         |  greetings
        |   .------------------.
        |   |                  |       
        |       
        |
        |   |                  |
        .-->.------------------.          
            |  "Hello World!"  | 
            .------------------.
      text  |                  |
```

---

- 试图把“Hello World!”中的大写H改为小写：
```C {linenos=inline,hl_lines=8}
char *greetings = "Hello World!";
double global = 7;

int main()
{
	static int i, j;

	*greetings = 'h';
	return 0;
}
```

---

```bash
$ gcc memory-layout.c -o memory-layout
$ ./memory-layout
```
```nohighlight
Segmentation fault (core dumped)
```

- 这个程序运行时会发生如上错误。你能指出错误原因吗？

---

```C
#include <stdio.h>

void foo()
{
	static int no;
	printf("%d\n", no++);
}

int main()
{
	for (int i = 0; i < 5; i++)
		foo();
	return 0;
}
```
- 请猜测上面的程序输出的信息是什么，然后运行程序验证你的猜测。

---

总结
- 未初始化的全局变量和静态变量位于bss段。bss段在目标文件中不占空间，程序运行时被分配内存并初始化为0。
- 已初始化的全局变量和静态变量位于data段。
- 程序指令和字符串位于text段。
- bss段和data段可读可写，text段只可读不可写。
- 这些段中的数据的生命周期等同于程序的生命周期，即这些段中的内存不需要分配和回收管理。

---

{{< slide transition="zoom" >}}

# 栈

---

- 一个先进后出的数据结构
- 非常契合函数之间的调用关系：函数A调用函数B，函数B调用函数C。返回时的顺序则是从C返回B，然后从B返回A。
- 把函数调用时的返回地址顺次压栈，即可实现我们期望的函数调用和返回关系。
- 函数内定义的局部变量，在函数被调用时在栈里分配空间，函数返回时回收。
- 因为局部变量在栈里分配空间，管理起来也方便，所以也较栈变量、自动变量。

---

- 在`main`里定义两个局部变量：
```C {linenos=inline,hl_lines=7}
char *greetings = "Hello World!";
double global = 7;

int main()
{
	static int i, j;
	int m, n;

	return 0;
}
```

---

```bash
$ gcc memory-layout.c -o memory-layout
$ size memory-layout
```
```nohighlight
   text    data     bss     dec     hex filename
   1455     560      16    2031     7ef memory-layout
```

- 再次查看，没有任何变化
- 说明局部变量不在`data`和`bss`段中分配内存

---

- 栈有大小限制。
- 当函数调用层次过深时，有可能会发生栈溢出现象。

---

- 这是一个用递归算法写的计算阶乘的程序；
```C
#include <stdio.h>

int factorial(int n)
{
        if (n == 1)
                return 1;
        else
                return n * factorial(n - 1);
}

int main()
{
        int n;

        printf("Please input a number: ");
        scanf("%d", &n);
        printf("Factorial of %d is %d\n", n, factorial(n));
        return 0;
}
```

---

- 运行该程序，得到的结果如下。请解释这些现象。

```nohighlight
输入10时，得到的结果：

Factorial of 10 is 3628800


输入100时，得到的结果：

Factorial of 10 is 0


输入1000000时，得到的结果：

Segmentation fault (core dumped)
```

---

- 根据你的理解，请问你能找到一种简单的方法判定在你的系统里，栈是向地址增大或是减小的方向扩展吗？

---
- 一个古怪的程序
```C
#include <stdio.h>

void foo()
{
        int f;
        printf("%X\n", f);
        f = 0xdeadbeef;
        printf("%X\n", f);
}

void bar()
{
        int b;
        printf("%X\n", b);
        b = 0xcafebabe;
        printf("%X\n", b);
}

int main()
{
        foo();
        bar();
        foo();
        return 0;
}
```

---

```nohighlight
5577
DEADBEEF
DEADBEEF
CAFEBABE
CAFEBABE
DEADBEEF
```

- 程序的输出是上面那个样子。你能解释原因吗？

---

{{< slide transition="zoom" >}}

# 堆

heap

---

- 用`malloc`在堆里分配内存。
- 在堆里分配的内存，最终需要用`free`释放返回。
- 全局变量、局部变量的生命周期不受程序员控制，但堆变量的生命周期完全受控于程序员，可以提供最大的灵活性。
- 你可以把堆简单想象成一个仓库，里面储满了内存。需要时从里面取一块，用完了还回去。如果用了不还，会导致可用内存越来越少，最终可能导致程序出错。

---

- 堆变量的生命周期相互独立，先分配的未必先释放。经过一段时间的使用之后，堆可能变成下面这样的筛子样：已分配出去的内存块和空闲的内存块交错排列。

```goat
      .---------------.      
      |▉▉  ▉▉▉▉▉▉▉▉   |      
      |  ▉▉  ▉▉     ▉▉|      
      |▉▉  ▉▉  ▉▉▉▉▉  |      
      |  ▉▉  ▉▉     ▉▉|      
      |▉▉▉▉▉  ▉▉▉▉▉   |      
      .---------------.      

```

---

## 堆内存的应用场景

- 函数返回后仍然要用到的数据应该放到堆里，如果你不希望它和程序具有同样的生命周期的话。

---

- 堆变量在使用之前需要先用`malloc`分配
- 堆变量用完之后用`free`释放
- 释放之后不允许继续使用
- 只能释放一次

---

- 一个使用堆的简单例子：
```C
#include <stdio.h>
#include <stdlib.h>

int main()
{
	// This pointer will hold the
	// base address of the block created
	int* ptr;
	int n, i;

	// Get the number of elements for the array
	printf("Enter number of elements:");
	scanf("%d",&n);
	printf("Entered number of elements: %d\n", n);

	// Dynamically allocate memory using malloc()
	ptr = (int*)malloc(n * sizeof(int));

	// Check if the memory has been successfully
	// allocated by malloc or not
	if (ptr == NULL) {
		printf("Memory not allocated.\n");
		exit(0);
	}
	else {

		// Memory has been successfully allocated
		printf("Memory successfully allocated using malloc.\n");

		// Get the elements of the array
		for (i = 0; i < n; ++i) {
			ptr[i] = i + 1;
		}

		// Print the elements of the array
		printf("The elements of the array are: ");
		for (i = 0; i < n; ++i) {
			printf("%d, ", ptr[i]);
		}
	}

	free(ptr);

	return 0;
}
```

---

- 堆变量听起来和看起来是不是特简单？
- 不要被它人畜无害的样子骗了。C程序中的大量bug都和它有关。
- 正是由于C的这个痛点，才催生了Java、Python、Go等采用[Garbage Clloection](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science))技术的语言。

---

## 常见错误

- 请指出以下各代码片段中的错误

---

```C
void foo()
{
	int *p;
	*p = 100;
}
```

---

```C
void foo()
{
	int n = 5;
	int *p = &n;
	(*p)++;
	free(p);
}
```

---

```C
void foo()
{
	int *p;
	free(p);
}
```

---

```C
void foo()
{
	char *p = malloc(128);
}
```

---

```C
void foo()
{
	char *p;
	p = (char*)malloc(100);
	free(p);
	free(p);
}
```

---

```C
struct x {
	int data;
};

int foo()
{
	struct x *p;
	p = (struct x *) malloc(sizeof (struct x));
	p->data = 10;
	free(p);
	return p->data;
}
```

---

## Happy Hacking