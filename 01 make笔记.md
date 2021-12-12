[toc]



# **Makefile 笔记**



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