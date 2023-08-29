## Shell脚本

shell脚本就是将多个命令放到一个文件中，然后运行这个文件就会一行一行地执行

Shell脚本的第一行一定要是：

**#!/bin/bash**  			表示使用bash

新建的shell脚本文件没有可执行权限，只有使用chmod给脚本文件添加执行权限之后才能执行

如果想在PATH变量中 追加一个路径写法如下：
		export PATH=$PATH:/需要添加的路径

### 基本语法

#### 注释

单个"#"号代表注释当前行

#### 执行方式

三种执行方式 

1. ./xxx.sh
2. bash xxx.sh
3. . xxx.sh

##### 区别：

./xxx.sh :	先按照 文件中#!指定的解析器解析。如果#！指定指定的解析器不存在 才会使用系统默认的解析器

bash xxx.sh:	指明先用bash解析器解析，如果bash不存在 才会使用默认解析器

. xxx.sh：	直接使用默认解析器解析（不会执行第一行的#！指定的解析器）但是第一行还是要写的

#### 变量

##### 定义变量：

变量名=变量值

相关命令：

**unset**  		清空变量中的值
**read**    		从键盘中获取值   
						在一行上显示和添加提示 需要加上-p  eg：read -p "num：" num
**readonly** 	定义只读变量	再次赋值无效
**env**				查看环境变量	

```shell
#!/bin/bash
num=100
echo $num
unset num
echo $num
```

![image-20230829202734742](image\7 Shell_变量1.jpg)

```shell
#!/bin/bash

num=10
echo "num=$num"
echo "请输入num的值请输入num的值"
read num
echo "num=$num"
```

![image-20230829203120187](image\8 Shell_变量2.jpg)

```shell
#!/bin/bash

readonly num=10
echo "num=$num"
read -p "输入num的值" num
echo "num=$num"
```

![image-20230829203653737](D:\Program Files(x86)\Linux\Linux_1\image\9 Shell_变量3.jpg)

**source命令**用法:source在当前bash环境下执行命令，而scripts是启动一个子shell来执行命令。这样如果把设置环境变量(或alias等等)的命令写进scripts中，就只会影响子shell,无法改变当前的BASH,所以通过文件(命令列)设置环境变量时，要用source 命令。

##### 注意事项：

1. 变量名只能包含英文字母下划线，不能以数字开头
2. 等号两边不能直接接空格符，若变量中本身就包含了空格，则整个字符串都要用双引号、或单引号括起来
3. 双引号 单引号的区别：**双引号可以解析变量的值，单引号不能解析变量的值**

#### 预设变量

##### shell直接提供无需定义的变量

- $$:	传给shell脚本参数的数量
- $*:    传给shell脚本参数的内容
- $1、$2...$9:   运行脚本时传递给其的参数，用空格隔开
- $?:    命令执行后返回的状态
               用于检查上一个命令执行是否正确（0表示正确，非0表示错误）    
- $0:    当前执行的进程名
- $$:    当前执行的进程号

```shell
#!/bin/bash
echo "参数的个数：$#"
echo "参数的内容：$*"
echo "第一个参数：$1"
echo "第二个参数：$2"
echo "第三个参数：$3"
readonly data=10
data=250
echo "data=250的运行结果：$?"
echo "进程名：$0"
echo "进程号：$$"
```

![image-20230829205246285](image\10 Shell_预设变量1.jpg)

##### 脚本标量的特殊用法

- ""（双引号）：包含的变量会被解释
- ''（单引号）：包含的变量会被当做字符串处理
- ``(反引号 数字键1上)：其中的内容作为系统命令，并执行其内容，可以替换输出为一个变量
- \ 转义字符： 同c一样\n\t\r\a等使用echo命令需要加-e转义
- (命令序列)：由子shell来完成，不会影响当前shell中的变量
- {命令序列}：在当前shell中执行，会影响当前变量

```shell
#!/bin/bash
echo "today is `date`."
echo "##\n##"
echo -e "##\n##"

data=10
(#子shell中完成
	data=100
	echo "()里面data=$data"
)
echo "当前data=$data"

{ #当前shell中完成
	data=100
	echo "{}里面data=$data"
}
echo "当前data=$data"
```

![image-20230829210115772](image\11 Shell_预设变量2.jpg)

#### 变量的拓展

##### 判断变量是否存在

```shell
#!/bin/bash
# ${num:-val} 如果num存在，整个表达式的值为num，否则为val
echo ${num:-100}
num=200
echo ${num:-100}

# ${num:=val} 如果num存在，整个表达式的值为num，否则为val，同时将val的值赋值给num
unset num
echo ${num:=100}
echo "num=$num"
```

![image-20230829210626686](image\12 Shell_变量拓展1.jpg)

##### 字符串的操作

```shell
#!/bin/bash
str="hehe:haha:xixi:lala"
#字符串长度${#str}
echo "len=${#str}"

#从下标n的位置提取${str:n}
echo ${str:3}

#从下标n的位置提取长度为l${str:3:l}
echo ${str:3:6}

#用new替换str中出现的第一个old ${str/old/new}
echo ${str/:/#}

#用new替换str中所有的old ${str//old/new}
echo ${str//:/#}
```

![image-20230829211200467](image\13 Shell_变量拓展2.jpg)

#### 条件测试

test命令：用于测试字符串、文件状态和数字
test命令有两种格式:
test condition 或[ condition ]
使用方括号时，要注意在条件两边加上空格。

##### 文件测试

测试文件状态的条件表达式

- -e:    是否存在
- -d:    是否目录
- -f:    是否文件
- -r:    是否可读
- -w:    是否可写
- -x:    是否可执行
- -L:    符号连接
- -c:    是否字符设备 
- -b:    是否块设备
- -s:    文件非空

```shell
#!/bin/bash
read -p "输入一个文件名" fileName

#test -e $fileName
[ -e $fileName ]
echo $?
```

![image-20230829211652541](image\14 Shell_条件测试1.jpg)

##### 字符串测试

test str_operator "str"
		test "str1" str_operator  "str2"

[ str_operator "str" ]
		[ "str1" str_operator  "str2" ]

**str_operator可以是：**

- =:     两个字符串相等
- !=:    两个字符串不相等
- -z:    空串
- -n:    非空串

![image-20230829212007706](image\15 Shell_条件测试2.jpg)

##### 数值测试

![image-20230829212136981](image\16 Shell_条件测试3.jpg)

![image-20230829212234494](image\17 Shell_条件测试4.jpg)

##### 符合语句测试

![image-20230829212331845](image\18 Shell_条件测试5.jpg)

![image-20230829212858758](image\19 Shell_条件测试6.jpg)

#### 控制语句

##### if控制语句

```shell
#格式一：
if [条件1]; then
    #执行第一段程序
else
    #执行第二段程序
fi
#格式二：
if [条件1]; then
    #执行第一段程序
elif [条件2]；then
	#执行第二段程序
else
    #执行第三段程序
fi

```

![image-20230829213215368](image\20 Shell_控制语句1.jpg)

##### case语句

![image-20230829213328645](image\21 Shell_控制语句2.jpg)

![image-20230829213429951](image\22 Shell_控制语句3.jpg)

##### for循环语句

![image-20230829213621337](image\23 Shell_控制语句4.jpg)

![image-20230829213740150](image\24 Shell_控制语句5.jpg)

##### while语句

![image-20230829213835564](image\25 Shell_控制语句6.jpg)

##### until语句

![image-20230829213928330](image\26 Shell_控制语句7.jpg)

##### break continue语句

![image-20230829214036236](D:\Program Files(x86)\Linux\Linux_1\image\27 Shell_控制语句8.jpg)

#### 函数

定义函数的两种格式

```shell
#格式一：
函数名 () {
	命令 ...
}

#格式二：
function 函数名 (){
	命令
}
```

所有函数在使用前必须定义，必须将函数放在脚本开始部分，直至shell解释器首次发现它时，才可以使用

**调用函数的格式为：**

​	函数名 para1 para2 ......

在函数中使用参数同在一般脚本中使用特殊变量

​	$1 、、、$9

函数可以使用return提前结束并带回返回值

![image-20230829214522471](image\28 Shell_控制语句9.jpg)