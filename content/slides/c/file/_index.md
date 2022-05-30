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

你可能会质疑我的说法，但我会捍卫我的观点。

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
- 结论：**打开文件时，根本不用纠结如何选择打开方式**

---

**如有不同意见敬请告知**

谢谢

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
- 压缩文件
- 可执行程序
- 音频文件
- ...

解释成功的前提是

**文件中的数据必须以我们期望的格式编码**

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

**Happy Hacking**