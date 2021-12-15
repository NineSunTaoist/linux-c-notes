[toc]



# **Makefile 笔记**

原文参考：https://seisman.github.io/how-to-write-makefile/introduction.html#



## 1、编译和连接

一般来说，C、C++首先要把源文件编译成中间代码文件，在Windows下也就是 .obj 文件，UNIX下是 .o 文件，即 Object File，这个动作叫做编译（compile）。然后再把大量的Object File合成执行文件，这个动作叫作链接（link）。  

    编译时，编译器需要的是语法的正确，函数与变量的声明的正确。对于后者，通常是你需要告诉编译器头文件的所在位置（头文件中应该只是声明，而定义应该放在C/C++文件中），只要所有的语法正确，编译器就可以编译出中间目标文件。一般来说，每个源文件都应该对应于一个中间目标文件（O文件或是OBJ文件）。 
    链接时，主要是链接函数和全局变量，所以，我们可以使用这些中间目标文件（O文件或是OBJ文件）来链接我们的应用程序。链接器并不管函数所在的源文件，只管函数的中间目标文件（Object File），在大多数时候，由于源文件太多，编译生成的中间目标文件太多，而在链接时需要明显地指出中间目标文件名，这对于编译很不方便，所以，我们要给中间目标文件打个包，在Windows下这种包叫“库文件”（Library File)，也就是 .lib 文件，在UNIX下，是Archive File，也就是 .a 文件。



## 2、$@，$^，$<代表的意义 

他们三个是十分重要的三个变量，所代表的含义分别是：

* $@  目标文件
* $^  所有的依赖文件
* $<  第一个依赖文件



## 3、makefile 简介

```
1.如果这个工程没有编译过，那么我们的所有C文件都要编译并被链接。
2.如果这个工程的某几个C文件被修改，那么我们只编译被修改的C文件，并链接目标程序。
3.如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的C文件，并链接目标程序。
```



## 4、Makefile入门案列

* 1 编辑main.c文件

```
#include "add.h"

int main() {
    Add(10, 15);
    printf("hello world\n");
}
```

* 2 编辑add.h文件

```
#include <stdio.h>

void Add(int a, int b);
```

* 3 编辑add.c文件

```
#include "add.h"

void Add(int a, int b) {
    printf("a + b = %d\n", a + b);
}
```

* 4 编辑Makefile文件

```
test: main.o add.o
	cc -o test main.o add.o

main.o : main.c add.h
	cc -c main.c
add.o : add.c add.h
	cc -c add.c

clean:
	rm edit *.o
```

执行make就生成可执行文件
执行make clean就清理文件



## 5、make使用变量

将上述makefile修改为变量的方式

```
objects = main.o add.o

test: $(objects)
	cc -o test $(objects)

main.o : main.c add.h
	cc -c main.c
add.o : add.c add.h
	cc -c add.c

clean:
	rm test $(objects)
```



## 6、make自动推导

```
objects = main.o add.o

test: $(objects)
	cc -o test $(objects)

main.o : add.h
add.o : add.h

clean:
	rm test $(objects)
```



## 7、清理垃圾

写在最后

```
clean:
	rm test $(objects)
```

更稳健的写法

```
.PHONY : clean
clean :
    -rm edit $(objects)
```

`.PHONY` 表示 `clean` 是一个“伪目标”。而在 `rm` 命令前面加了一个小减号的意思就是，也许某些文件出现问题，但不要管，继续做后面的事。



## 8、Makefile文件名

通常使用Makefile和makefile

使用其他名字时`make -f Make.Linux` 或 `make --file Make.AIX`



## 9、引用其它的Makefile

在Makefile使用 `include` 关键字可以把别的Makefile包含进来，这很像C语言的 `#include` ，被包含的文件会原模原样的放在当前文件的包含位置。 `include` 的语法是：

```
include <filename>
```

`filename` 可以是当前操作系统Shell的文件模式（可以包含路径和通配符）。

在 `include` 前面可以有一些空字符，但是绝不能是 `Tab` 键开始。 `include` 和 `<filename>` 可以用一个或多个空格隔开。举个例子，你有这样几个Makefile： `a.mk` 、 `b.mk` 、 `c.mk` ，还有一个文件叫 `foo.make` ，以及一个变量 `$(bar)` ，其包含了 `e.mk` 和 `f.mk` ，那么，下面的语句：

```
include foo.make *.mk $(bar)
```

等价于：

```
include foo.make a.mk b.mk c.mk e.mk f.mk
```

make命令开始时，会找寻 `include` 所指出的其它Makefile，并把其内容安置在当前的位置。就好像C/C++的 `#include` 指令一样。如果文件都没有指定绝对路径或是相对路径的话，make会在当前目录下首先寻找，如果当前目录下没有找到，那么，make还会在下面的几个目录下找：

1. 如果make执行时，有 `-I` 或 `--include-dir` 参数，那么make就会在这个参数所指定的目录下去寻找。
2. 如果目录 `<prefix>/include` （一般是： `/usr/local/bin` 或 `/usr/include` ）存在的话，make也会去找。

如果有文件没有找到的话，make会生成一条警告信息，但不会马上出现致命错误。它会继续载入其它的文件，一旦完成makefile的读取，make会再重试这些没有找到，或是不能读取的文件，如果还是不行，make才会出现一条致命信息。如果你想让make不理那些无法读取的文件，而继续执行，你可以在include前加一个减号“-”。如：

```
-include <filename>
```

其表示，无论include过程中出现什么错误，都不要报错继续执行。和其它版本make兼容的相关命令是sinclude，其作用和这一个是一样的。