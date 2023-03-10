# Makefile

# 1.为什么需要Makefile

在上一章节的例子中，我们都是在终端执行 gcc 命令来完成源文件的编译。
感觉挺方便的，这是因为工程中的源文件只有一两个，在终端直接执行编译命令，确实快捷方便。
但是现在一些项目工程中的源文件不计其数，其按类型、功能、模块分别放在若干个目录中，如果仍然在终端输入这些命令来编译，那显然不切实际，开发
效率极低。
我们需要一个工具来管理这些编译过程，这就是“make”。make 是一个应用程序，它根据 Makefile 来做事。Makefile 负责管理整个
编译流程：要编译哪些文件？怎么编译这些文件？怎么把它们链接成一个可执行程序。Makefile 定义了一系列的规则来实现这些管理。

# 2.makefie的引入

Makefile 的引入是为了简化我们编译流程，提高我们的开发进度。下面我们用一个例子来说明 Makefile 如何简化我们的编译流程。我们创建一个工程内
容分别 main.c，sub.c，sub.h，add.c，add.h 五个文件。sub.c 负责计算两个数减法运算，add.c 负责计算两个数加法运算，然后编译出可执行文件。其源
文件内容如下：

```c
main.c：
#include <stdio.h>
#include "add.h"
#include "sub.h"
int main()
{
printf("100 ask, add:%d\n", add(10, 10));
printf("100 ask, sub:%d\n", sub(20, 10));
return 0;
}
add.c：
#include "add.h"
int add(int a, int b)
{
return a + b;
}
```

```c
add.h：
#ifndef __ADD_H
#define __ADD_H
int add(int a, int b);
#endif
```

```c
sub.c：
#include "sub.h"
int sub(int a, int b)
{
return a - b;
}
```

```c
sub.h：
#ifndef __SUB_H
#define __SUB_H
int sub(int a, int b);
#endif
```

我们使用 gcc 对上面工程进行编译及生成可执行程序，在终端输入如下命令，如下：

```bash
$ gcc main.c sub.c add.c -o ouput
$ ls
add.c add.h main.c output sub.c sub.h
$ ./output
100 ask, add:20
100 ask, sub:10
```

上面的命令是通过 gcc 编译器对 main.c sub.c add.c 这三个文件进行编译，及生成可执行程序 output，并执行可执行文件产生结果。
从上面的例子看起来比较简单，一条命令完成三个源程序的编译并产生结果，这是因为目前只有三个源文件。

如果有上千个源文件，即使你只修改了其中一个文件，你执行这样的命令时，
它会把所有的源文件都编译一次。这样消耗的时间是非常恐怖的。
我们想实现：哪个文件被修改了，只编译这个被修改的文件即可，其它没有
修改的文件就不需要再次重新编译了。可以使用下列命令，先单独编译文件，再最后链接，如下：

```bash
$ gcc -c main.c
$ gcc -c sub.c
$ gcc -c add.c
$ gcc main.o sub.o add.o -o output
```

我们将上面一条命令变成了四条，分别编译出源文件的目标文件，最后再将所有的目标文件链接成可执行文件。
当其中一个源文件的内容发生了变化，我们只需要单独编译它，然后跟其他文件重新链接成可知执行文件，不再需要重新编译其他文件。
假如我们修改了 add.c 文件，只需要重新编译生成 add.c 的目标文件，然后再将所有的.o 文件链接成可执行文件，如下：

```bash
$ gcc -c add.c
$ gcc main.o sub.o add.o -o output
```

这样的方式虽然可以节省时间，但是仍然存在几个问题，如下：

\1. 如果源文件的数目很多，那么我们需要花费大量的时间，敲命令执行。
\2. 如果源文件的数目很多，然后修改了很多文件，后面发现忘记修改了什么。
\3. 如果头文件的内容修改，替换，更换目录，所有依赖于这个头文件的源文件全部需要重新编译。
这些问题我们不可能一个一个去找和排查，引入 Makefile 可以解决上述问题。我们先扔出一个 Makefile，如下：



```makefile
Makefile：
output: main.o add.o sub.o
	gcc -o output main.o add.o sub.o
main.o: main.c
	gcc -c main.c
add.o: add.c
	gcc -c add.c
sub.o: sub.c
	gcc -c sub.c
clean:
	rm *.o output
```

Makefile 编写好后只需要执行 make 命令，就可以自动帮助我们编译工程。注意，make 命令必须要在 Makefile 的当前目录执行，如下：

```bash
$ ls
add.c add.h main.c Makefile sub.c sub.h
$ make
gcc -c main.c
gcc -c add.c
gcc -c sub.c
gcc -o output main.o add.o sub.o
$ ls
add.c add.h add.o main.c main.o Makefile output sub.c sub.h sub.o
```

通过 make 命令就可以生成相对应的目标文件.o 和可执行文件。
如果我们再次使用 make 命令编译，如下，它说你的程序已经是最新的了，不需要再做什么事情：

```bash
$ make
make: 'output' is up to date.
```

我们可以修改一下 add.c，然后再执行 make，如下：

```bash
$ make
gcc -c add.c
gcc -o output main.o add.o sub.o
```

会发现，它重新编译了 add.c，并且重新生成了可执行程序。
通过上述例子，Makefile 将我们上面的三个问题都解决了，无需手工输入复杂的命令，只编译修改过的文件。在一个很庞大的工程中，只有第一次编译时间比较长，第二次编译时会大大缩短时间，节省了我们的开发周期。
下面我们来学习 Makefile 的知识。

# 3.Makefile 的规则

命名规则：
一般来说将 Makefile 文件取名为“Makefile”或“makefile”都可以，
惯 例 是 使 用 首 字 母 大 写 的 “ Makefile ”。 也 可 以 使 用 其 他 名 字 ， 比 如makefile.linux，但你需要用“-f”参数指定，示例如下

```
make -f makefile.linux
```

基本语法规则：

![image](https://github.com/1WMD1/script_file/blob/master/makefile/png/image-20230310110255519.png)

* target：需要生成的目标文件
* prerequisites：生成该 target 所依赖的一些文件
*  command：生成该目标需要执行的命令



三者的关系：target 依赖于 prerequisites 中的文件，其生成规则定义在command 中。举例，比如我们平时要编译一个文件：

```
$ gcc main.c -o main
```

换成 Makefile 的书写格式如下，，：

```bash
01 main:main.c
02 gcc main.c -o main
```

第 1 行表示 main 这个可执行程序依赖于 main.c，第 2 行表示需要用“gcc main.c -o main”这个命令来生成 main。
注意：Makefile 文件里的命令必须要使用 Tab，不能使用空格。

目标生成规则：

* 目标生成：

> 检查规则中的依赖文件是否存在。

> 若依赖文件不存在，则寻找是否有规则用来生成该依赖文件。

目标生成流程，如下：

![image](https://github.com/1WMD1/script_file/blob/master/makefile/png/image-20230310111353906.png)

* 目标更新：

> 检查目标的所有依赖，任何一个依赖有更新时，就要重新生成目标。
> 目标文件比依赖文件更新时间晚，则需要更新。

目标更新流程，如下：

![image](https://github.com/1WMD1/script_file/blob/master/makefile/png/image-20230310111817321.png)

我们使用上面的例子，Makefile 内容如下：

```makefile
output: main.o add.o sub.o
	gcc -o output main.o add.o sub.o
main.o: main.c
	gcc -c main.c
add.o: add.c
	gcc -c add.c
sub.o: sub.c
	gcc -c sub.c
clean:
	rm *.o output
```

编译执行：

```bash
$ make
gcc -c main.c
gcc -c add.c
gcc -c sub.c
gcc -o output main.o add.o sub.o
```

make 命令会检测寻找目标的依赖是否存在，不存在，则会寻找生成依赖的命令。
① . output 依赖于“main.o add.o sub.o”，这三个文件都没有，分别去生成它们。怎么生成？继续寻找规则，发现 main.o 依赖于 main.c，于是
使用“gcc -c main.c”来生成；其他两个文件类似处理。

② 现在 output 的依赖文件都有了，但是 output 文件还没有，所以使用“gcc -o output main.o add.o sub.o”来生成 output 文件。当我们修改某一个文件时，比如之修改 add.c 文件，然后重新 make，如下：

```bash
$ make
gcc -c add.c
gcc -o output main.o add.o sub.o
```

make 命令还是根据 Makefile 来检查、执行：
① output 依赖于“main.o add.o sub.o”，这三个文件都有了，main.o 依赖于 main.c，main.c 没改过，所以不需要重新生成 main.o；但是 add.o的依赖文件 add.c 被改过，需要重新生成 add.o，使用命令“gcc -c add.c”；sub.o 跟 main.o 类似，也不需要重新生成。
② 现在 output、它的三个依赖文件都有了，但是其中一个依赖文件 add.o 比output 文件新，所以使用“gcc -o output main.o add.o sub.o”来生成output 文件。

# 5.Makefile的语法

## 5.1 变量的定义及取值

Makefile 也支持变量定义，变量的定义也让的我们的 Makefile 更加简化，可复用。
变量的定义：一般采用大写字母，赋值方式像 C 语言的赋值方式一样，如下：

```bash
DIR = ./100ask/
```

变量取值：使用括号将变量括起来再加美元符，如下：

```bash
FOO = $(DIR)
```

变量可让 Makefile 简化可复用，上面个的 Makefile 文件，内容如下：

```bash
output: main.o add.o sub.o
gcc -o output main.o add.o sub.o
main.o: main.c
gcc -c main.c
add.o: add.c
gcc -c add.c
sub.o: sub.c
gcc -c sub.c
clean:
rm *.o output
```

我们可以将其优化，如下：

```bash
#Makefile 变量定义
OBJ = main.o add.o sub.o
output: $(OBJ)
gcc -o output $(OBJ)
main.o: main.c
gcc -c main.c
add.o: add.c
gcc -c add.c
sub.o: sub.c
gcc -c sub.c
clean:
rm $(OBJ) output
```

我们分析一下上面简化过的 Makefile，第一行是注释，Makefile 的注释采用‘#’，而且不支持像 C 语言中的多行注释。第二行我们定义了变量 OBJ，并赋值字符串”main.o，add.o，sub.o“。其中第三，四，十三行，使用这个变量。这样用到用一个字符串的地方直接调用这个变量，无需重复写一大段字符串。
Makefile 除了使用‘=’进行赋值，还有其他赋值方式，比如‘:=’和‘?=’，接下来我们来对比一下这几种的区别：

**赋值符‘=’**
我们使用一个例子来说明赋值符‘=’的用法。Makefile 内容如下：

```bash
01 PARA = 100
02 CURPARA = $(PARA)
03 PARA = ask
04
05 print:
06 @echo $(CURPARA)
```

分析代码：第一行定义变量 PARA，并赋值为“100”，第二行定义变量 CURPARA，并赋值引用变量 PARA，此时 CURPARA 的值和 PARA 的值是一样的，第三行，将变量 PARA 的变量修改为“ask”。第六行输出 CURPARA 的值，echo 前面增加@符号，代表只显示命令的结果，不显示命令本身。
通过命令“make print”执行 Makefile，如下：

```bash
$ make print
ask
```

从结果上看，变量 CURPARA 的值并不是“100”，而是 PARA 的最后一次赋值。使用赋值符“=”设置的变量，它的值在运行时才能确定，这称为“延时变量”。
其实可以理解为在 C 语言中，定义一个指针变量指向一个变量的地址。如下：

```c
01 int a = 10;
02 int *b = &a;
03 a=20;
```

代码分析：我们见上面的 Makefile 的第二行的“=”替换成“:=”，重新编译，如下：

```
$ make print
100
$
```

从结果上看，变量 CURPARA 的值为“100”。使用“:=”设置的变量被称为“即时变量”，在赋值时就确定了它的值。

**赋值符‘?=’**
我们用两个 Makefile 来说明赋值符“?=”的用法。如下：

```makefile
第一个 Makefile：
PARA = 100
PARA ?= ask
print:
	@echo $(PARA)
```

编译结果：

```bash
$ make print
100
```

```makefile
第二个 Makefile:
PARA ?= ask
print:
@echo $(PARA)
```

编译结果：

```bash
$ make print
ask
```

上面的例子说明，使用“?=”给变量设置值时，如果这个变量之前没有被设置过，那么“?=”才会起效果；如果曾经设置过这个变量，那么“?=”不会起效果。赋值符‘+=’
Makefile 中的变量是字符串，有时候我们需要给前面已经定义好的变量添加一些字符串进去,此时就要使用到符号“+=”，比如如下：

```makefile
01 OBJ = main.o add.o
02 OBJ += sub.o
```

这样的结果是 OBJ 的值为：”main.o，add.o，sub.o“。说明“+=”用作与变量的追加。

## 5.2 系统自带变量

系统自定义了一些变量，通常都是大写，比如 CC，PWD，CLFAG 等等，有些有默认值，有些没有，比如以下几种，如下：
① CPPFLAGS：预处理器需要的选项，如：-l
② CFLAGS：编译的时候使用的参数，-Wall -g -c
③ LDFLAGS：链接库使用的选项，-L -l
其中：默认值可以被修改，比如 CC 默认值是 cc，但可以修改为 gcc：CC=gcc使用的例子，如下：

```
01 OBJ = main.o add.o sub.o
02 output: $(OBJ)
03 gcc -o output $(OBJ)
04 main.o: main.c
05 gcc -c main.c
06 add.o: add.c
07 gcc -c add.c
08 sub.o: sub.c
09 gcc -c sub.c
10
11 clean:
12 rm $(OBJ) output
```

使用系统自带变量，如下：

```
01 CC = gcc
02 OBJ = main.o add.o sub.o
03 output: $(OBJ)
04 $(CC) -o output $(OBJ)
05 main.o: main.c
06 $(CC) -c main.c
07 add.o: add.c
08 $(CC) -c add.c
09 sub.o: sub.c
10 $(CC) -c sub.c
11
12 clean:
13 rm $(OBJ) output
```

在上面例子中，系统变量 CC 不改变默认值，也同样可以编译，修改的目的是为了明确使用 gcc 编译。

## 5.3 自动变量

Makefile 的语法提供一些自动变量，这些变量可以让我们更加快速地完成Makefile 的编写，其中自动变量只能在规则中的命令使用，常用的自动变量如
下：
① $@：规则中的目标
② $<：规则中的第一个依赖文件
③ $^：规则中的所有依赖文件
我们上面的例子继续完善，修改为采用自动变量的格式，如下:

```bash
01 CC = gcc
02 OBJ = main.o add.o sub.o
03 output: $(OBJ)
04 $(CC) -o $@ $^
05 main.o: main.c
06 $(CC) -c $<
07 add.o: add.c
08 $(CC) -c $<
09 sub.o: sub.c
10 $(CC) -c $<
11
12 clean:
13 rm $(OBJ) output
```

其中：第 4 行$^表示变量 OBJ 的值，即 main.o add.o sub.o，第四，第六，第八行的$<分别表示 main.c add.c sub.c。$@表示 output。

### 6.5.4 模式规则

模式规则实在目标及依赖中使用%来匹配对应的文件，我们依旧使用上面的例子，采用模式规则格式，如下：

```
01 CC = gcc
02 OBJ = main.o add.o sub.o
03 output: $(OBJ)
04 $(CC) -o $@ $^
05 %.o: %.c
06 $(CC) -c $<
07
08 clean:
09 rm $(OBJ) output
```

其中：第五行%.o: %.表示如下。
① main.o 由 main.c 生成
② add.o 由 add.c 生成
③ sub.o 由 sub.c 生成

### 6.5.5 伪目标

在前面的例子中，我们直接执行“make”命令，它的目的是去执行第 1 个规则，这跟执行“make output”的效果是一样的。在这里，“output”既是规则
的目标，也是一个实际的文件。
而伪目标是什么呢？对于以前的例子，先执行 make 命令，然后再执行“make clean”命令，如下

```
$make
gcc -c main.c
gcc -c add.c
gcc -c sub.c
gcc -o output main.o add.o sub.o
$make clean
rm *.o output
```

一切正常！接着我们做个手脚，在 Makefile 目录下创建一个 clean 的文件，然后依旧执行 make 和 make clean，如下：

```
$touch clean
$make
gcc -c main.c
gcc -c add.c
gcc -c sub.c
gcc -o output main.o add.o sub.o
$make clean
make: 'clean' is up to date.
```

为什么“make clean”时命令没有被执行？因为已经有名为 clean 的文件了，并且它的依赖是空的，执行规则的条件没满足。
伪目标就是为了解决这个问题，我们在 clean 前面增加“.PHONY:clean”，
如下：

```
01 CC = gcc
02 OBJ = main.o add.o sub.o
03 output: $(OBJ)
04 $(CC) -o $@ $^
05 %.o: %.c
06 $(CC) -c $<
07
08 .PHONY:clean
09 clean:
10 rm $(OBJ) output
```

运行结果：

```
$make
gcc -c main.c
gcc -c add.c
gcc -c sub.c
gcc -o output main.o add.o sub.o
$make clean
rm *.o output
```

当一个目标被声明为伪目标后，make 在执行规则时就会默认它满足执行条件。
这样就提高了 make 的执行效率，也不用担心由于目标和文件名重名了。
伪目标的两大好处：
① 避免只执行命令的目标和工作目录下的实际文件出现名字冲突。
② 提高执行 Makefile 时的效率

# 6 Makefile 函数

Makefile 提供了大量的函数，函数调用的格式如下：

```
$(function arguments)
```

这里`function’是函数名，`arguments’是该函数的参数。参数和函数名之间是用空格或 Tab 隔开，如果有多个参数，它们之间用逗号隔开。这些空格和逗号不是参数值的一部分。
代 码 ： GIT 下 载 后 在 “ 10_ 裸 机 开 发 /01_100ASK_IMX6ULL 裸 机 程 序/6_Makefile 与 GCC/001_Makefile_02”目录下。
我们经常使用的函数主要有两个(wildcard，patsubst)，先把它们单独拎出来讲讲。创建一个文件夹 src，在里下面创建两个文件，100.c，ask.c。如下：

```
.
├── Makefile
└── src
	├── 100.c
	└── ask.c
```

**wildcard 函数**
用于查找指定目录下指定类型的文件，函数参数：目录+文件类型，如下：

```
$(wildcard 指定文件类型)
```

其中，指定文件类型，如果不写路径，则默认为当前目录查找，例子如下：

```
01 SRC = $(wildcard ./src/*.c)
02
03 print:
04 @echo $(SRC)
```

执行命令 make，结果如下：

```
$ make
./src/ask.c ./src/100.c
```

其中，这条规则表示：找到目录./src 下所有后缀为.c 的文件，并赋值给变量SRC。命令执行完，SRC 变量的值：./src/ask.c ./src/100.c



**patsubst 函数**
用于匹配替换。函数参数：原模式+目标模式+文件列表，如下:

```
$( patsubst 原模式, 目标模式, 文件列表)
```

其中，从文件列表中查找出符合原模式文件类型的文件，然后一一替换成目标模式。举例：将./src 目录下的.c 结尾的文件，替换成.o 文件，并赋值给 obj。
如下：

```
SRC = $(wildcard ./src/*.c)
OBJ = $(patsubst %.c, %.o, $(SRC))
print:
@echo $(OBJ)
```

执行命令 make，结果如下：

```
$ make
./src/ask.o ./src/100.o
```

其中，这条规则表示：把变量中所有后缀为.c 的文件替换为.o。 命令执行完，OBJ 变量的值：./src/ask.o ./src/100.o字符串替换和分析函数

① $(subst from,to,text)
在文本`text’中使用`to’替换每一处`from’。比如：

```
$(subst ee,EE,feet on the street)
```

结果为‘fEEt on the strEEt’。

② $(patsubst pattern,replacement,text)，前面讲过。
寻找`text’中符合格式`pattern’的字，用`replacement’替换它们。
`pattern’和`replacement’中可以使用通配符。比如：

```
$(patsubst %.c,%.o,x.c.c bar.c）
```

结果为：`x.c.o bar.o’。

③ $(strip string)
去掉前导和结尾空格，并将中间的多个空格压缩为单个空格。比如：

```
$(strip a b c )
```

结果为`a b c’。

④ $(findstring find,in)
在字符串`in’中搜寻`find’，如果找到，则返回值是`find’，否则返回值为空。比如：

```
$(findstring a,a b c)
$(findstring a,b c)
```

将分别产生值`a’和`’(空字符串)。

⑤ $(filter pattern...,text)
返回在`text’中由空格隔开且匹配格式`pattern...’的字，去除不符合格式`pattern...’的字。比如

```
$(filter %.c %.s,foo.c bar.c baz.s ugh.h)
```

结果为`foo.c bar.c baz.s’。

⑥ $(filter-out pattern...,text)
返回在`text’中由空格隔开且不匹配格式`pattern...’的字，去除符合格式`pattern...’的字。它是函数 filter 的反函数。比如：

```
$(filter %.c %.s,foo.c bar.c baz.s ugh.h)
```

结果为`ugh.h’。

⑦ $(sort list)
将‘list’中的字按字母顺序排序，并去掉重复的字。输出由单个空格隔开的字的列表。比如：

```
$(sort foo bar lose)
```

返回值是‘bar foo lose’。

**文件名函数**

① $(dir names...)
抽取‘names...’中每一个文件名的路径部分，文件名的路径部分包括从文件名的首字符到最后一个斜杠(含斜杠)之前的一切字符。比如：

```
$(dir src/foo.c hacks)
```

结果为‘src/ ./’。

② $(notdir names...)
抽取‘names...’中每一个文件名中除路径部分外一切字符（真正的文件名）。比如：

```
$(notdir src/foo.c hacks)
```

结果为‘foo.c hacks’。

③ $(suffix names...)
抽取‘names...’中每一个文件名的后缀。比如：

```
$(suffix src/foo.c src-1.0/bar.c hacks)
```

结果为‘.c .c’。

④ $(basename names...)
抽取‘names...’中每一个文件名中除后缀外一切字符。比如：

```
$(basename src/foo.c src-1.0/bar hacks)
```

结果为‘src/foo src-1.0/bar hacks’。

⑤ $(addsuffix suffix,names...)
参数‘names...’是一系列的文件名，文件名之间用空格隔开；suffix 是一个后缀名。将 suffix(后缀)的值附加在每一个独立文件名的后面，完成后将
文件名串联起来，它们之间用单个空格隔开。比如：

```
$(addsuffix .c,foo bar)
```

结果为‘foo.c bar.c’。

⑥ $(addprefix prefix,names...)
参数‘names’是一系列的文件名，文件名之间用空格隔开；prefix 是一个前缀名。将 preffix(前缀)的值附加在每一个独立文件名的前面，完成后将文件
名串联起来，它们之间用单个空格隔开。比如：

```
$(addprefix src/,foo bar)
```

结果为‘src/foo src/bar’。

⑦ $(wildcard pattern) ，前面讲过
参数‘pattern’是一个文件名格式，包含有通配符(通配符和 shell 中的用法一样)。函数 wildcard 的结果是一列和格式匹配的且真实存在的文件的名
称，文件名之间用一个空格隔开。
比如若当前目录下有文件 1.c、2.c、1.h、2.h，则:

```
c_src := $(wildcard *.c)
```

结果为‘1.c 2.c’。

**其他函数**

① $(foreach var,list,text)
前两个参数，‘var’和‘list’将首先扩展，注意最后一个参数‘text’，此时不扩展；接着，‘list’扩展所得的每个字，都赋给‘var’变量；然后‘text’
引用该变量进行扩展，因此‘text’每次扩展都不相同。
函数的结果是由空格隔开的‘text’ 在‘list’中多次扩展后，得到的新‘list’，就是说：‘text’多次扩展的字串联起来，字与字之间由空格隔开，如此就产生
了函数 foreach 的返回值。
下面是一个简单的例子，将变量‘files’的值设置为 ‘dirs’中的所有目录下的所有文件的列表：

```
dirs := a b c d
files := $(foreach dir,$(dirs),$(wildcard $(dir)/*))
```

这里‘text’是‘$(wildcard $(dir)/*)’，它的扩展过程如下：

a) 第一个赋给变量 dir 的值是`a’，扩展结果为‘$(wildcard a/*)’；
b) 第二个赋给变量 dir 的值是`b’，扩展结果为‘$(wildcard b/*)’；

c) 第三个赋给变量 dir 的值是`c’，扩展结果为‘$(wildcard c/*)’；
d) 如此继续扩展。

这个例子和下面的例有共同的结果：

```
files := $(wildcard a/* b/* c/* d/*)
```

② $(if condition,then-part[,else-part])
首先把第一个参数‘condition’的前导空格、结尾空格去掉，然后扩展。如果扩展为非空字符串，则条件‘condition’为‘真’；如果扩展为空字符串，
则条件‘condition’为‘假’。
如果条件‘condition’为‘真’,那么计算第二个参数‘then-part’的值，并将该值作为整个函数 if 的值。
如果条件‘condition’为‘假’,并且第三个参数存在，则计算第三个参数‘else-part’的值，并将该值作为整个函数 if 的值；如果第三个参数不存
在，函数 if 将什么也不计算，返回空值。
注意：仅能计算‘then-part’和‘else-part’二者之一，不能同时计算。这样有可能产生副作用（例如函数 shell 的调用）。

③ $(origin variable)
变量‘variable’是一个查询变量的名称，不是对该变量的引用。所以，不能采用‘$’和圆括号的格式书写该变量，当然，如果需要使用非常量的文件名，
可以在文件名中使用变量引用。

函数 origin 的结果是一个字符串，该字符串变量是这样定义的：

* ‘undefined'：如果变量‘variable’从没有定义；
*  ‘default'：变量‘variable’是缺省定义；
*  ‘environment'：变量‘variable’作为环境变量定义，选项‘-e’没有打开；
*  ‘environment override'：变量‘variable’作为环境变量定义，选项‘-e’已打开；
*  ‘file' ：变量‘variable’在 Makefile 中定义；
*  ‘command line'：变量‘variable’在命令行中定义；
*  ‘override'：变量‘variable’在 Makefile 中用 override 指令定义；
*  ‘automatic' ：变量‘variable’是自动变量

④ $(shell command arguments)
函数 shell 是 make 与外部环境的通讯工具。函数 shell 的执行结果和在控制台上执 行‘ command arguments ’的 结果相似。 不过如果 ‘ command
arguments’的结果含有换行符（和回车符），则在函数 shell 的返回结果中将把它们处理为单个空格，若返回结果最后是换行符（和回车符）则被去掉。
比如当前目录下有文件 1.c、2.c、1.h、2.h，则：

```
c_src := $(shell ls *.c)
```

结果为‘1.c 2.c’。

# 7 Makefile 实例

代 码 ： GIT 下 载 后 在 “ 10_ 裸 机 开 发 /01_100ASK_IMX6ULL 裸 机 程 序/6_Makefile 与 GCC/001_Makefile_03”目录下。
在上面的例子中，我们都是把头文件，源文件放在同一个文件里面，这样不好用于维护，所以我们将其分类，把它变得更加规范一下，把所有的头文件放在
文件夹：inc，把所有的源文件放在文件夹：src。
代码目录如下：

```
$ tree
.
├── inc
│ ├── add.h
│ └── sub.h
├── Makefile
└── src
	├── add.c
	├── main.c
	└── sub.c
```

其中 Makefile 的内容如下：

```
01 SOURCE = $(wildcard ./src/*.c)
02 OBJECT = $(patsubst %.c, %.o, $(SOURCE))
03
04 INCLUEDS = -I ./inc
05
06 TARGET = 100ask
07 CC = gcc
08 CFLAGS = -Wall -g
09
10 $(TARGET): $(OBJECT)
11 @mkdir -p output/
12 $(CC) $^ $(CFLAGES) -o output/$(TARGET)
13
14 %.o: %.c
15 $(CC) $(INCLUEDS) $(CFLAGES) -c $< -o $@
16
17 .PHONY:clean
18 clean:
19 @rm -rf $(OBJECT) output/
```

分析：

* 行 1：获取当前目录下 src 所有.c 文件，并赋值给变量 SOURCE。
* 行 2：将./src 目录下的.c 结尾的文件，替换成.o 文件，并赋值给变量OBJECT。
* 行 4：通过-I 选项指明头文件的目录，并赋值给变量 INCLUDES。
*  行 6：最终目标文件的名字 100ask，赋值给 TARGET。
*  行 7：替换 CC 的默认之 cc，改为 gcc。
*  行 8：将显示所有的警告信息选项和 gdb 调试选项赋值给变量 CFLAGS。
*  行 11：创建目录 output，并且不再终端现实该条命令。
* 行 12：编译生成可执行程序 100ask，并将可执行程序生成到 output 目录
*  行 15：将源文件生成对应的目标文件。
*  行 17：伪目标，避免当前目录有同名的 clean 文件。
*  行 19：用与执行命令 make clean 时执行的命令，删除编译过程生成的文件。
  最后编译的结果，如下:

```
$ make
gcc -I ./inc -c src/main.c -o src/main.o
gcc -I ./inc -c src/add.c -o src/add.o
gcc -I ./inc -c src/sub.c -o src/sub.o
gcc src/main.o src/add.o src/sub.o -o output/100ask
$tree
.
├── inc
│ ├── add.h
│ └── sub.h
├── Makefile
├── output
│ └── 100ask
└── src
	├── add.c
	├── add.o
	├── main.c
	├── main.o
	├── sub.c
	└── sub.o
```

上面的 Makefile 文件算是比较完善了，不过项目开发中，代码需要不断的迭代，那么必须要有东西来记录它的变化，所以还需要对最终的可执行文件添加版本号，如下：

```
01 VERSION = 1.0.0
02 SOURCE = $(wildcard ./src/*.c)
03 OBJECT = $(patsubst %.c, %.o, $(SOURCE))
04
05 INCLUEDS = -I ./inc
06
07 TARGET = 100ask
08 CC = gcc
09 CFLAGS = -Wall -g
10
11 $(TARGET): $(OBJECT)
12 @mkdir -p output/
13 $(CC) $^ $(CFLAGES) -o output/$(TARGET)_$(VERSION)
14
15 %.o: %.c
16 $(CC) $(INCLUEDS) $(CFLAGES) -c $< -o $@
17
18 .PHONY:clean
19 clean:
20 @rm -rf $(OBJECT) output/
```

* 行 1：将版本号赋值给变量 VERSION。
* 行 13：生成可执行文件的后缀添加版本号。

编译结果：

```
$ tree
.
├── inc
│ 	├── add.h
│ 	└── sub.h
├── Makefile
├── output
│ 	└── 100ask_1.0.0
└── src
	├── add.c
		├── add.o
	├── main.c
	├── main.o
	├── sub.c
	└── sub.o
```


