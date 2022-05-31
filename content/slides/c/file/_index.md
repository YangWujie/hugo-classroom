+++
title = "File"
outputs = ["Reveal"]
date = 2022-05-30T09:19:36+08:00
draft = false
author = "杨武杰"
categories = [ "演示文稿", "编程" ]
tags = [ "C", "文件I/O" ]
year = "2022"
month = "2022/05"
+++

{{< slide transition="zoom" >}}

# [C文件输入输出](https://en.wikipedia.org/wiki/C_file_input/output)

---

- 文件无处不住，这包括你的头文件，你的源程序文件，编译生成的目标文件，链接得到的可执行文件
- 不涉及文件操作的C程序几乎不存在
- 即便那个最简单的`Hello World`程序也涉及文件操作，只不过穿了个马甲

---

`printf`就是个穿了马甲的文件操作

```C
#include <stdio.h>

int main()
{
	printf("Hello World!\n");
	return 0;
}
```

---

马甲之下的真相：
```C
#include <stdio.h>

int main()
{
	fprintf(stdout, "Hello World!\n");
	return 0;
}
```
这个程序和原来的`Hello World`程序等价。

---

## [标准输出设备](https://en.wikipedia.org/wiki/Standard_streams#Standard_output_(stdout))

- stdout即标准输出设备，程序运行时系统自动为你打开
- 除了stdout之外，系统还自动打开了stdin（标准输入设备）和stderr（标准错误设备）
- 文件、设备、流几个术语在某些场合下经常互换使用

---

```C
int printf ( const char *format, ... );
int fprintf( FILE *stream, const char *format, ... );
```
看了它们的函数原型，了解它们为什么有那样的名字了吧？

- stream − This is the pointer to a FILE object that identifies the stream.
- format − This is the C string that contains the text to be written to the stream. It can optionally contain embedded format tags that are replaced by the values specified in subsequent additional arguments and formatted as requested. 

---

[What does 'stream' mean in C?](https://stackoverflow.com/questions/38652953/what-does-stream-mean-in-c)

---

```C {linenos=inline}
#include <stdio.h>

int main() {
	FILE *fp;

	fp = fopen("test.txt", "a");
	if (fp == NULL) {
		perror("Error: ");
		return -1;
	}

	fprintf(fp, "This is testing for fprintf...\n");
	fputs("This is testing for fputs...\n", fp);
	fclose(fp);

	return 0;
}
```
---

这个程序运行后，屏幕上不会显示信息。如果查看当前目录，则会发现一个名为`test.txt`的文件，其中包含如下信息：
```nohighlight
This is testing for fprintf...
This is testing for fputs...
```
---

再运行一次，`test.txt`中的信息则变为：
```nohighlight
This is testing for fprintf...
This is testing for fputs...
This is testing for fprintf...
This is testing for fputs...
```
请解释这个现象的原因，可参考[C File Handling](https://www.programiz.com/c-programming/c-file-input-output)

---

- 请把`test.txt`文件的存取属性设置为“只读”，然后再次运行这个程序，请问得到了什么结果？

- 如果把程序的第7-10行删除，运行时又会得到什么结果？

**教训**

C没有异常处理机制，I/O操作时，请记得检查函数的返回值。否则，这可能成为程序的一个bug。

---

{{< slide transition="zoom" >}}

# 文本方式打开

vs

# 二进制方式打开

请参考[Binary vs Text](https://stackoverflow.com/questions/26966715/differences-between-writing-reading-binary-text-in-c)

---

**没有区别，几乎没有！**

现在仍然强调这种区别，更多是因为历史原因。

请参考[difference between text file and binary file](https://stackoverflow.com/questions/6039050/difference-between-text-file-and-binary-file)

---

你可能会质疑我的说法，但我会事实捍卫我的观点。

- 在Linux下，无论是以二进制方式打开还是以文本方式打开，只要程序其他方面相同，你会得到完全相同的结果
- 在Windows下，你会得到几乎同样的结果，其中只有一点点细微的差别。我如果不提醒，你可能都注意不到

---

程序一：以文本方式打开
```C {linenos=inline,hl_lines=5}
#include <stdio.h>

int main() {
	FILE *fp;
	fp = fopen("test.txt", "w");
	if (fp == NULL) {
		perror("Error: ");
		return -1;
	}
	fprintf(fp, "A\nB");
    fclose(fp);

	return 0;
}
```

---

程序二：以二进制方式打开
```C {linenos=inline,hl_lines=5}
#include <stdio.h>

int main() {
	FILE *fp;
	fp = fopen("test.txt", "wb");
	if (fp == NULL) {
		perror("Error: ");
		return -1;
	}
	fprintf(fp, "A\nB");
	fclose(fp);
	return 0;
}
```

---

- 这两个程序在Linux下运行结果完全相同
- 这两个程序在Windows下运行，其中一个向文件中写了3个字节，另一个向文件中写了4个字节
- 之所以有这种区别，是因为以文本方式打开，写入文件时把换行符转换成了回车和换行两个字符
- 结论：**打开文件时，根本不用纠结如何选择打开方式，它们几乎没有区别。即便有些微区别，它对你几乎没有影响。**

---

{{< slide transition="zoom" >}}

# [编码](https://en.wikipedia.org/wiki/Code)

[![Braille chart](Russian_Braille_chart.jpg)](https://en.wikipedia.org/wiki/Braille)

---

本质上

**所有文件均以二进制方式存在**

重点是

**数据是以何种编码方式生成的**

以及

**我们如何对这些数据进行解读**

---

我们可以把一个文件解释为
- 文本文件
- 压缩包
- 可执行程序
- 音频
- ...

解释成功的前提是

**文件中的数据以我们期望的格式编码**

---

书籍推荐

[Code: The Hidden Language of Computer Hardware and Software](https://en.wikipedia.org/wiki/Code:_The_Hidden_Language_of_Computer_Hardware_and_Software)

---

一个例子
```C {linenos=inline,hl_lines=12}
#include <stdio.h>

int main() {
	FILE *fp;

	int i = 12345678;
	fp = fopen("test.txt", "wb");
	if (fp == NULL) {
		perror("Error: ");
		return -1;
	}
	fprintf(fp, "%d", i);
	fclose(fp);
	return 0;
}
```
---

- 这个程序输出的文字信息使用了UTF-8编码规范（在我的电脑上），在你的电脑上也许使用的是不同的规范。
- 同一个文本文件，在有的电脑上能正常显示，在有的电脑上却是乱码，原因就在于解读时使用了错误的编码规范。

---

- 这个程序是以二进制方式打开文件的，请问得到的`test.txt`文件中包含什么内容？
- 假设把打开方式改为文本方式，结果会有不同吗？
- 在Linux和Windows下运行，会有不同吗？
- 这个程序生成的`test.txt`应该怎样解读？请写代码验证。

---

另一个例子
```C {linenos=inline,hl_lines=12}
#include <stdio.h>

int main() {
	FILE *fp;

	int i = 12345678;
	fp = fopen("test.txt", "wb");
	if (fp == NULL) {
		perror("Error: ");
		return -1;
	}
	fwrite(&i, sizeof(int), 1, fp);
	fclose(fp);
	return 0;
}
```
请参考[C library function - fwrite()](https://www.tutorialspoint.com/c_standard_library/c_function_fwrite.htm)

---

这个程序输出的数据信息遵循[ANSI C](https://en.wikipedia.org/wiki/ANSI_C)规范中的数据表示标准。

---

- 这个程序和上一个程序的区别仅在于第12行，请问得到的`test.txt`文件中包含什么内容？和上一个程序相同吗？
- 假设把打开方式改为文本方式，结果会有不同吗？
- 在Linux和Windows下运行，会有不同吗？
- 这个程序生成的`test.txt`应该怎样解读？请写代码验证。
- 这个程序通过修改打开方式，或者修改文件名后缀，能生成文本文件吗？

---

{{< slide transition="zoom" >}}

# 文件路径

---

- 操作文件的第一个步骤是打开文件。
- 打开文件时需要指明文件位置，即文件路径。
- 在计算机系统里，文件系统是一个倒立的树状结构。
- Linux的根目录用“/”表示。
- Windows的根目录用盘符表示，例如“C:\”。从这个意义上，我们把Windows的文件系统看作森林更合适（因为多个盘没有共同的根）。

---

## 绝对路径
- 在Windows下，以盘符开始的路径
- 在Linux下，以“/”开始的路径

绝对路径的概念很清晰，用起来也很简单，但并不常用。为什么？

---

举个生活中的例子。如果要告诉别人自己的位置，我们并不直接说出经纬度，而是先说一个地标，然后说出相对地标的方位和距离，这样更方便。

程序也一样。在不同的计算机系统，程序有可能被安装到不同的目录，所以访问自己安装目录里的文件时，如果要使用绝对路径，首先需要了解自己安装目录，然后再拼装出完整路径，这并不方便。

如果把程序的安装目录当作参照物，说出一个相对这个参照物的路径，然后由系统自动拼装出绝对路径不是很方便吗？

---

## 相对路径
- 顾名思义，相对路径就是相对于某个目录的路径。
- 我们把这个作为参照的路径叫做当前工作目录。
- 系统为每个运行的程序都设定一个当前工作目录。
- 程序也可根据需要自行改变当前工作目录。

---

## 如何了解程序的当前工作目录？
在不同情况下，系统给程序设定的当前工作目录可能不一样。
- 在Visual Studio里
- 在命令行窗口直接键入程序名
- 把程序拖入命令行窗口

这些不同情形，因为设定的当前工作目录不同，可能会给大家带来困惑。如果需要，可以通过程序的方式获得。

---

Windows下获取当前工作目录的程序：
```C
#include <stdio.h>
#include <direct.h>

int main()
{
	char buff[FILENAME_MAX];
	_getcwd(buff, FILENAME_MAX);
	printf("Current working dir: %s\n", buff);
	return 0;
}
```

---

Linux下获取当前工作目录的程序：

```C
#include <unistd.h>
#include <stdio.h>
#include <limits.h>

int main()
{
	char cwd[PATH_MAX];
	if (getcwd(cwd, sizeof(cwd)) != NULL) {
		printf("Current working dir: %s\n", cwd);
	}
	return 0;
}
```

---

## 挑战

用宏定义和条件编译预处理指令把两个程序合二为一。

---

## 相对路径的形式

- Linux下，不以“/”开头的路径
- Windows下，不以盘符开头的路径
- 路径中可以使用“.”代表当前目录
- 路径中可以使用“..”代表父目录

---

## 绝对路径和相对路径的关系

**当前工作目录 + 相对路径 = 绝对路径**

---

## 建议

除非是访问系统文件，否则一般不要使用绝对路径，特别是在访问和当前程序相关的文件时。

比如，在你的系统里，你用绝对路径在D盘创建了文件。但在另一个系统里，可能根本不存在D盘。这时，你的程序就无法正常工作了。

---

**Happy Hacking**