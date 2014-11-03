#Makefile简明笔记

----------
##背景知识
1. Makefile相当于Windows下的IDE。
2. .c文件 ---编译---> .o文件
3. .o文件 ---链接---> .out文件
4. 链接时，很多的.o文件不好管理，为了方便，可以把它们一股脑打包，windows下叫.lib文件，Linux下叫.a文件
5. 链接的内容是函数和全局变量，所以如果函数未声明，可以编译（出警告），但无法链接，因为链接程序找不到函数。

##Makefile简介

###Makefile的规则

	target : prerequisites
		command

1. target：目标文件
2. prerequisites：生成target所需要的文件（依赖规则）
3. command：make需要执行的shell命令（生成规则）

注意：command以TAB开头！

###使用变量

Makefile中的变量类似于c语言中的宏。

	OBJS = c1.o c2.o c3.o ... cn.o 定义变量OBJS
	$(OBJS)  使用变量OBJS

###自动推导

	if make看到一个 whatever.o
	then 自动(1)把whatever.c加入依赖 (2)cc -c whatever.c

以下代码中，.PHONY 表示clean是个伪目标文件：

	PHONY : clean
	clean:
	-rm ...

rm前面的减号作用：也许某些文件出现问题，但是我不管，继续执行我该做的事情（删除）！

###引用其他的Makefile

	include <filename>

###Makefile查找文件的位置顺序

1. 当前目录
2. -I 指定的目录
3. <prefix>/include目录（一般是/usr/local/bin/或者/usr/include）
 
##书写规则

Makefile中只有一个最终目标（即Makefile中的第一个目标），其他目标都是被这个目标连带出来的。

###文件搜索

背景：多个源文件分别存放在不同的目录中。

（1）使用特殊变量VPATH指定目录：

	VPATH = dir1:dir2:dir3:../dir4

不同的目录由冒号隔开。

（2）使用关键字vpath指定目录：

	vpath <pattern> <directories>为符合模式的文件指定一个搜索目录
	vpath <pattern> 清除该模式的搜索目录
	vpath 清除所有模式的搜索目录

*pattern* 中需要包含 %，%的意思是：匹配若干个字符。
	
	%.h 表示所有以.h结尾的文件。

所以，

	vpath %.h ../headers

表示：make如果在当前目录下没有找到.h文件的话，就在../hearders目录下继续搜索。

###伪目标

以clean为例，make clean并不会生成clean这个目标，所以，伪目标并不是一个文件，只是一个标签（label）。再所以，伪目标不能与文件名重名，除非使用 .PHONY 显示声明。

	.PHONY : clean

###多目标

自动化变量 "$@"

	bigoutput littleoutput : text.g
	generate text.g -$(subst output, , $@) > $@

subst是一个函数，$@ 表示多个目标的集合，类似数组，遍历该数组。

所以，上述代码等价于：

	bigoutput : text.g
	generate text.g -big > bigoutput
	littleoutput : text.g
	generate text.g -little > littleoutput

###静态模式

静态模式可以更加容易地定义多目标。

代码举例：

	OBJS = foo.o bar.o
	all: $(OBJS)
	$(OBJS): %.o: %.c
	$(CC) -c $(CFLAGS) $< -o $@

1. "%.o"表明所有以.o结尾的目标，即：foo.o和bar.o
2. 依赖模式"%.c"取模式"%.o"的"%"，也就是 foo和bar，并为其加 .c的后缀，所以，依赖目标是 foo.c和bar.c
3. $< ：所有的依赖目标集（foo.c bar.c）
4. $@ ：所有的目标集（foo.o bar.o）

###自动生成依赖性

使用GCC的 -MM参数：自动寻找源文件中包含的头文件，并形成一个依赖关系。

##来写命令吧

1. 每条命令的开头必须以 Tab 键开头
2. 在命令行之间的空格或空行会被忽略，但如果该空格或空行是以Tab键开头的，那么make会认为其是一个 空命令
3. make的命令默认是被 "/bin/sh" 解释执行的

###显示命令

1. 默认下：make会在执行一条命令之前，把将要被执行的命令行输出到终端
2. 使用 "@" ：将要执行的命令行不会被输出到终端，可以看做 shut up！
3. 使用参数 "-n"：just print，只显示命令，不会执行。可用于调试Makefile，查看其执行顺序
4. 使用参数 "-s"：silent，全面禁止命令的显示，默默执行

举例1：

	echo 正在编译XXX模块...

执行结果：

	echo 正在编译XXX模块...
	正在编译XXX模块...

举例2：
	
	@echo 正在编译XXX模块...

执行结果：

	正在编译XXX模块...

说明："@" 导致echo命令行被 shut up了。

###命令执行

如果要让上一条命令的结果应用到下一条命令，那么就不能把这两条命令写在两行上，而应该把这两条命令写在一行上，用分号分隔。

举例1：(当前目录是 /home/usr1)

	exec:
	cd /home/usr2
	pwd

执行 "make exec" 的结果：
	
	/home/usr1

举例2：(当前目录是 /home/usr1)

	exec:
	cd /home/usr2; pwd

执行 "make exec" 的结果：
	
	/home/usr2

###命令出错

意味着命令退出码非零。

解决方案：

1. 命令行前加 "-"：不管命令出不出错，都认为是成功的
2. 使用make参数 "-i"：ignore errors，忽略所有的命令行的所有错误，全局！
3. 使用make参数 "-k"：keep going，如果某个目标中有个命令出错了，则只终止该目标，继续执行其他目标

###嵌套执行make，每个子目录一个Makefile

假设有一个子目录subdir，该子目录下有一个子Makefile文件，生成了一个子目标subsystem。

则总控Makefile的写法如下：
	
	subsystem:
		cd subdir && $(MAKE)

或者：
	
	subsystem:
		$(MAKE) -C subdir

注意：

1. 定义 $(MAKE) 宏的原因是：也许make需要一些参数，所以把make以及参数定义成一个变量比较利于维护
2. 这两个例子的意思都是：先进入subdir子目录，然后make
3. 如果要从总控Makefile传递某个变量到下级Makefile中，则可以在总控Makefile中写：


		export variable = value
4. 如果要传递所有的变量：


		export
5. 注意：SHELL和MAKEFLAGS这两个变量总是要被传递到下层Makefile中
6. 使用 "-w" 参数：可以看到当前的工作目录：
 
		make: Entering directory '/home/usr/...'  和 make: Leaving directory '/home/usr/...'

###定义命令包

类似于windows中的批处理，把多个命令序列封装成一个包，使用方法类似于一个变量。

定义cmd_bag:

	define cmd_bag
	cmd1
	cmd2
	...
	cmdn
	endef

使用cmd_bag:

	$(cmd_bag)



	
	








