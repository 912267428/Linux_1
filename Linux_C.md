## 编写C程序

vim或者vscode
VIM设置Tab4个空格和显示行号：
sudo vim /etc/vim/vimrc
然后在末尾加上：
set ts=4
set nu

## 编译C程序

使用gcc编译器

gcc <文件> <选项>

#### 直接编译产生可执行文件

-o <文件名>  在选项-o的后面跟着输出文件名
**eg. gcc main.c -o hello**
gcc分别执行预处理、编译、汇编、链接四个阶段，生成可执行文件hello（如果不加上-o hello，Unix系统的默认输出文件名是a.out）
**gcc main.c plus.c -o program** （多个文件用空格分开）
gcc main.c plus.c -o program （多个文件用空格分开）

-On(n为1到3的数)	 对代码进行优化（减少程序运行时间和程序大小）(n的数字越大，优化等级越高）
-Wall		生成所有警告信息。(gcc默认只会生成错误信息，不会生错误信息)
-I		指定包含的头文件的路径

指定生成各阶段的文件

| 选项 | 说明                             |
| ---- | -------------------------------- |
| -E   | 预处理后的文件（.i）             |
| -S   | 编译后得到的汇编代码文件（.s）   |
| -c   | 汇编后得到的二进制目标文件（.o） |

1. gcc main.c -E -o main.i （使用-E选项必须指定输出文件名）

生成main.i文件

2. gcc main.c plus.c -S

生成main.s和plus.s汇编代码文件

3. gcc main.c plus.c -c

生成main.o和plus.o目标文件

## Makefile

通过在终端执行 gcc 命 令来完成 C 文件的编译，如果我们的工程只有一两个 C 文件还好，需要输入的命令不多，当文 件有几十、上百甚至上万个的时候用终端输入 GCC 命令的方法显然是不现实的。如果我们能够 编写一个文件，这个文件描述了编译哪些源码文件、如何编译那就好了，每次需要编译工程的 时只需要使用这个文件就行了。这种问题怎么可能难倒聪明的程序员，为此提出了一个解决大 工程编译的工具：make，描述哪些文件需要编译、哪些需要重新编译的文件就叫做 Makefile， Makefile 就跟脚本文件一样，Makefile 里面还可以执行系统命令。使用的时候只需要一个 make命令即可完成整个工程的自动编译，极大的提高了软件开发的效率。

在要使用make编译时只需要输入“make”这个命令即可，其会自动在当前目录下寻找Makefile文件（必须叫这个名字）并根据其定义的规则进行编译。

### Makefile语法

#### Makefile规则格式：

```makefile
目标 : 依赖文件集合……
	命令 1
	命令 2
	……
eg：
main : main.o input.o calcu.o
 gcc -o main main.o input.o calcu.o
```

这条规则的目标是 main，main.o、input.o 和 calcu.o 是生成 main 的依赖文件，如果要更新目标 main，就必须先更新它的所有依赖文件，如果依赖文件中的任何一个有更新，那么目标也 必须更新，“更新”就是执行一遍规则中的命令列表。

**命令列表中的每条命令必须以 TAB 键开始，不能使用空格！**

make 命令会为 Makefile 中的每个以 TAB 开始的命令创建一个 Shell 进程去执行。

```makefile
main: main.o input.o calcu.o
	gcc -o main main.o input.o calcu.o
main.o: main.c
	gcc -c main.c
input.o: input.c
	gcc -c input.c
calcu.o: calcu.c
	gcc -c calcu.c
 
clean:
	rm *.o
	rm main
```

上述代码中一共有 5 条规则，1~2 行为第一条规则，3~4 行为第二条规则，5~6 行为第三条 规则，7~8 行为第四条规则，10~12 为第五条规则，make 命令在执行这个 Makefile 的时候其执行步骤如下： 首先更新第一条规则中的 main，第一条规则的目标成为默认目标，只要默认目标更新了那么就认为 Makefile 的工作。在第一次编译的时候由于 main 还不存在，因此第一条规则会执行， 第一条规则依赖于文件 main.o、input.o 和 calcu.o 这个三个.o 文件，这三个.o 文件目前还都没有，因此必须先更新这三个文件。make 会查找以这三个.o 文件为目标的规则并执行。以 main.o 为例，发现更新 main.o 的是第二条规则，因此会执行第二条规则，第二条规则里面的命令为“gcc –c main.c”，这行命令很熟悉了吧，就是不链接编译 main.c，生成 main.o，其它两个.o 文件同理。 最后一个规则目标是 clean，它没有依赖文件，因此会默认为依赖文件都是最新的，所以其对应 的命令不会执行，当我们想要执行 clean 的话可以直接使用命令“**make clean**”，执行以后就会删 除当前目录下所有的.o 文件以及 main，因此 clean 的功能就是完成工程的清理。

**Make 的执行过程：**

1. make 命令会在当前目录下查找以 Makefile(makefile 其实也可以)命名的文件。 
2. 当找到 Makefile 文件以后就会按照 Makefile 中定义的规则去编译生成最终的目标文件。
3. 当发现目标文件不存在，或者目标所依赖的文件比目标文件新(也就是最后修改时间比 目标文件晚)的话就会执行后面的命令来更新目标。

#### Makefile变量

跟 C 语言一样 Makefile 也支持变量的

```makefile
main: main.o input.o calcu.o
	gcc -o main main.o input.o calcu.o
```

上述 Makefile 语句中，main.o input.o 和 calcue.o 这三个依赖文件，我们输入了两遍，我们 这个 Makefile 比较小，如果 Makefile 复杂的时候这种重复输入的工作就会非常费时间，而且非 常容易输错，为了解决这个问题，Makefile 加入了变量支持。不像 C 语言中的变量有 int、char 等各种类型，Makefile 中的变量都是字符串！类似 C 语言中的宏。使用变量将上面的代码修改， 修改以后如下所示：

```makefile
#Makefile 变量的使用
objects = main.o input.o calcu.o
main: $(objects)
	gcc -o main $(objects)

```

第 1 行是注释，Makefile 中可以写注释，注释开头要 用符号“#”，第 2 行我们定义了一个变量 objects，并且 给这个变量进行了赋值，其值为字符串“main.o input.o calcu.o”，第 3 和 4行使用到了变量 objects， Makefile 中变量的引用方法是“**$(变量名)**”，比如本例中的“$(objects)”就是使用变量 objects。

在定义变量 objects 的时候使用“=”对其进行了赋值，Makefile 变量的赋值符还有其它两个“:=”和“?=”

##### 赋值符号“=”

使用“=”在给变量的赋值的时候，不一定要用已经定义好的值，也可以使用后面定义的值，

```makefile
name = zzk
curname = $(name)
name = zuozhongkai

print:
	@echo curname: $(curname)
```

第 1 行定义了一个变量 name，变量值为“zzk”，第 2 行也定义 了一个变量curname，curname的变量值引用了变量name，按照我们C写语言的经验此时curname 的值就是“zzk”。第 3 行将变量 name 的值改为了“zuozhongkai”，第 5、6 行是输出变量 curname 的值。在 Makefile 要输出一串字符的话使用“echo”，就和 C 语言中的“printf”一样，第 6 行 中的“echo”前面加了个“@”符号，因为 Make 在执行的过程中会自动输出命令执行过程，在 命令前面加上“@”的话就不会输出命令执行过程。使用 命令“make print”来执行上述代码，可以发现curname的值竟然是name修改之后的zuozhongkai，所以“=”借助另外一个变量，可以将变量的真实 值推到后面去定义。也就是变量的真实值取决于它所引用的变量的最后一次有效值。

##### 赋值符号“:=”

赋值符“:=”不会使用后面定义的变量，只能使用前面已经定义好的（也就是说复制之后原来的变量再改变对已经复制的变量没有用），这就是“=”和“:=”两个的区别。

##### 赋值符“?=”

curname ?= zuozhongkai

上述代码的意思就是，如果变量 curname 前面没有被赋值，那么此变量是“zuozhongkai”， 如果前面已经赋过值了，那么就使用前面赋的值。

##### 变量追加“+=”

Makefile 中的变量是字符串，有时候我们需要给前面已经定义好的变量添加一些字符串进 去，此时就要使用到符号“+=”

```makefile
objects = main.o inpiut.o
objects += calcu.o
```

一开始变量 objects 的值为“main.o input.o”，后面我们给他追加了一个“calcu.o”，因此变 量 objects 变成了“main.o input.o calcu.o”，这个就是变量的追加。

#### Makefile模式规则

前面所见的Makefile文件中每一个 C 文件都要写一个对 应的规则，如果工程中 C 文件很多的话显然不能这么做。这是可以使用Makefile的模式规则，通过模式规则我们就可以使用一条规则来将所有的.c 文件编译为对应的.o 文件。

模式规则中，至少在规则的目标定定义中要包涵“%”，否则就是一般规则，目标中的“%” 表示对文件名的匹配，“%”表示长度任意的非空字符串，比如“%.c”就是所有的以.c 结尾的 文件，类似与通配符，a.%.c 就表示以 a.开头，以.c 结束的所有文件。

当“%”出现在目标中的时候，目标中“%”所代表的值决定了依赖中的“%”值，使用方 法如下：

```makefile
%.o: %.c
	#命令
```

```makefile
objects = main.o input.o calcu.o
main: $(objects)
	gcc -o main $(objects)
	
%.o : %.c
	#命令

clean:
	rm *.o
	rm main
```

修改以后的 Makefile 还不能运行，因为第 6 行的命令我们还没写呢，第 6 行的命令我们需要借 助另外一种强大的变量—自动化变量。

#### Makefile 自动化变量

上面的模式规则中，目标和依赖都是一系列的文件，每一次对模式规则进行解析的时候 都会是不同的目标和依赖文件，而命令只有一行，如何通过一行命令来从不同的依赖文件中生 成对应的目标？自动化变量就是完成这个功能的！所谓自动化变量就是这种变量会把模式中所 定义的一系列的文件自动的挨个取出，直至所有的符合模式的文件都取完，自动化变量只应该出现在**规则的命令**中，常用的自动化变量如表 ：

| 自动化变量 | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| $@         | 规则中的目标集合，在模式规则中，如果有多个目标的话，“$@”表示匹配模 式中定义的目标集合。 |
| $%         | 当目标是函数库的时候表示规则中的目标成员名，如果目标不是函数库文件， 那么其值为空。 |
| $<         | 依赖文件集合中的第一个文件，如果依赖文件是以模式(即“%”)定义的，那么 “$<”就是符合模式的一系列的文件集合。 |
| $?         | 所有比目标新的依赖目标集合，以空格分开。                     |
| $^         | 所有依赖文件的集合，使用空格分开，如果在依赖文件中有多个重复的文件， “$^”会去除重复的依赖文件，值保留一份。 |
| $+         | 和“$^”类似，但是当依赖文件存在重复的话不会去除重复的依赖文件。 |
| $*         | 这个变量表示目标模式中"%"及其之前的部分，如果目标是 test/a.test.c，目标模 式为 a.%.c，那么“$*”就是 test/a.test。 |

```makefile
objects = main.o input.o calcu.o
main: $(objects)
	gcc -o main $(objects)
	
%.o : %.c
	gcc -c $<

clean:
	rm *.o
	rm main
```

上述代码代码就是修改后的完成的 Makefile

####  Makefile 伪目标

Makefile 有一种特殊的目标——伪目标，一般的目标名都是要生成的文件，而伪目标不代 表真正的目标名，在执行 make 命令的时候通过指定这个伪目标来执行其所在规则的定义的命 令。

使用伪目标主要是为了避免 Makefile 中定义的执行命令的目标和工作目录下的实际文件出 现名字冲突，有时候我们需要编写一个**规则用来执行一些命令**，但是这个规则不是用来创建文件的，比如在前面的代码中的clean目标。
为了避免文件夹中存在了clean这个名字的文件，导致make clean不能达到预期效果，可以将clean声明为伪目标：

```makefile
.PHONY : clean
```

```makefile
objects = main.o input.o calcu.o
main: $(objects)
	gcc -o main $(objects)
	
.PHONY : clean

%.o : %.c
	gcc -c $<

clean:
	rm *.o
	rm main
```

第 5 行声明 clean 为伪目标，声明 clean 为伪目标以后不管当前目录下是否存在名 为“clean”的文件，输入“make clean”的话规则后面的 rm 命令都会执行。

#### Makefile 条件判断

Makefile的条件判断语法有两种：

```makefile
<条件关键字>
	<条件为真时执行的语句>
endif

#以及
<条件关键字>
	<条件为真时执行的语句>
else
	<条件为假时执行的语句>
endif

```

其中条件关键字有 4 个：ifeq、ifneq、ifdef 和 ifndef，这四个关键字其实分为两对、ifeq 与 ifneq、ifdef 与 ifndef，先来看一下 ifeq 和 ifneq，ifeq 用来判断是否相等，ifneq 就是判断是否不 相等，**ifeq 用法如下：**

```makefile
ifeq (<参数 1>, <参数 2>)
ifeq ‘<参数 1 >’,‘ <参数 2>’
ifeq “<参数 1>”, “<参数 2>”
ifeq “<参数 1>”, ‘<参数 2>’
ifeq ‘<参数 1>’, “<参数 2>”
```

上述用法中都是用来比较“参数 1”和“参数 2”是否相同，如果相同则为真，“参数 1”和 “参数 2”可以为函数返回值。ifneq 的用法类似，只不过 ifneq 是用来了比较“参数 1”和“参 数 2”是否不相等，如果不相等的话就为真。

**ifdef 和 ifndef** 的用法如下：

ifdef <变量名>
如果“变量名”的值非空，那么表示表达式为真，否则表达式为假。“变量名”同样可以是 一个函数的返回值。ifndef 用法类似，但是含义用户 ifdef 相反。

#### Makefile 函数使用

##### 函数 subst

![image-20230829192226777](image\1 Makefile_subst函数.jpg)

##### 函数 patsubst

![image-20230829192324477](image\2 Makefile_patsubst函数.jpg)

##### 函数 dir

![image-20230829192401032](image\3 Makefile_dir函数.jpg)

##### 函数 notdir

![image-20230829192437092](image\4 Makefile_notdir函数.jpg)

##### 函数 foreach

![image-20230829192606114](image\5 Makefile_foreach函数.jpg)

##### 函数 wildcard

![image-20230829192644996](image\6 Makefile_wildcard函数.jpg)