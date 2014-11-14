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

##使用变量

- 使用 ":=" 的方式给变量赋值的好处：只能使用前面已定义好的变量，不会造成死循环调用
- 系统变量“MAKELEVEL”的作用：记录当前make的调用层数
- “#”注释符可用于表示变量定义的终止：

		定义Empty变量
		nullstring :=
	
		使用#来表示变量定义的终止	
		space := $(nullstring)#  

		dir := /foo/bar# 这个#非常重要啦！不会出现粗心的空格

- “？=”操作符的作用：如果变量FOO之前没有被定义过，就定义它为bar；否则什么也不干（人家有男友了，你就别动人家了，道德点）

		FOO ?= bar

- $(var:a=b) 作用是把变量var中所有以“a”结尾的“a”替换为“b”：

		foo := a.o b.o c.o
		bar := $(foo:.o=.c)
		等价于：
		foo := a.o b.o c.o
		bar := a.c b.c c.c
		等价于：
		foo := a.o b.o c.o
		bar := $(foo:%.o=%.c)

- 变量命名的组合：

		first_second = Hello
		a = first
		b = second
		all = $($a_$b)

		所以 $(all) 是 "Hello"

- "+=" 可以追加变量值：

		objects = src1.c src2.c
		objects += src3.c 

		现在 $(objects) 是 src1.c src2.c src3.c

- 定义在文件中的变量，如果要向下层Makefile传递，需要使用export关键字
- 为某个目标单独设置自己的局部变量：

		prog : CFLAGS = -g
		prog : prog.o foo.o bar.o
		$(CC) $(CFLAGS) prog.o foo.o bar.o

		prog.o : prog.c
		$(CC) $(CFLAGS) prog.c

		foo.o : foo.c
		$(CC) $(CFLAGS) foo.c
		
		bar.o : bar.c
		$(CC) $(CFLAGS) bar.c

		在这个例子中，不管全局的$(CFLAGS)是什么，在prog目标（以及其所引发的所有规则：prog.o foo.o bar.o）中，
		$(CFLAGS)都是 "-g"

- override针对：
	- 系统环境传入的变量
	- make命令行指定的变量

##使用条件判断

6个关键字：

1. ifeq
2. ifneq
3. ifdef
4. ifndef
5. else
6. endif

举例如下：

	libs_for_gcc = -lgcc
	normal_libs =

	ifeq ($(CC), gcc)
		libs = $(libs_for_gcc)
	else
		libs = $(normal_libs)
	endif

	foo : $(objects)
	$(CC) -o foo $(objects) $(libs)

##使用函数

- 使用方法：

		$(subst a,b,c) 相当于c语言中的 subst(a, b, c)

- 字符串处理函数：
	
		$(subst <from>,<to>,<text>) 字符串替换
		$(patsubst <pattern>,<replacement>,<text>) 模式字符串替换
		$(strip <string>) 去掉开头和结尾的空格
		$(findstring <find>,<in>) 在字符串<in>中查找<find>字符
		$(filter <pattern...>,<text>) 从<text>中过滤出（保留）满足<pattern...>的单词
		$(filter-out <pattern...>,<text>) 从<text>中反过滤出（去掉）满足<pattern...>的单词
		$(sort <list>) 升序排序<list>中的单词
		$(word <n>,<text>) 从<text>中取出第<n>个单词
		$(wordlist <s>,<e>,<text>) 从<text>中取出从第<s>个到第<e>个单词组合
		$(words <text>) 统计<text>中的单词个数
		$(firstword <text>) 取<text>的首个单词
		
- 字符串处理函数典型例子分析：

		VPATH = src:../headers
		override CFLAGS += $(patsubst %, -I%, $(subst :, ,$(VPATH)))

		1，$(subst :, ,$(VPATH)))的结果是：(把，变成空格)
			src ../headers
		2，$(patsubst %, -I%, src ../headers)的结果是：
			-Isrc -I../headers

- 文件名操作函数:

		$(dir <names...>) 从文件名序列<names...>中取出目录部分
		$(notdir <names...>) 从文件名序列<names...>中取出非目录部分
		$(suffix <names...>) 取后缀，比如 .c .h .o
		$(basename <names...>) 取前缀
		$(addsuffix <suffix>,<names...>) 加后缀
		$(addprefix <prefix>,<names...>) 加前缀
		$(join <list1>,<list2>) 链接，举例：
			$(join aaa bbb, 111 222 333) => "aaa111 bbb222 333"
		$(foreach <var>,<list>,<text>) 循环体，举例：
			names := a b c d
			files := $(foreach n, $(names), $(n).o)
				=>
			a.o b.o c.o d.o
		$(if <condition>, <then-part>)
		$(if <condition>, <then-part>, <else-part>)
		$(call <expression>,<parm1>,<parm2>,<parm3>...) 函数调用
		$(origin <variable>) 变量来源：
			undefined default environment file command_line override automatic
		
- shell函数
		
		$(shell <cmd>)	
	
- 控制make的函数

		$(error <text>) 调用error，打印错误信息<text>
		$(warning <text>) 调用warning，打印警告信息<text>

##make 的运行

- make的退出码：
	- 0：成功执行
	2. 1：出现错误
	3. 2：使用了make的"-q"选项，有些目标不需要更新

- 指定make
	- 默认下的顺序：
		1. GNUmakefile
		2. makefile
		3. Makefile
	- "-f": 人为指定一个自定义的文件名字（比如：my_Makefile.mk）

		 	make -f my_Makefile.mk 

- 指定目标
	- MAKECMDGOALS：存放着你所指定的目标，是一个list
	- 举例1：

				sources = foo.c bar.c
				ifneq($(MAKECMDGOALS), clean)
					include $(sources:.c=.d)
				endif

	- 结果：
		- 只要我们输入的命令不是 "make clean"，那么Makefile会自动包含 "foo.d" 和 "bar.d" 这两个Makefile

	- 举例2：

				.PHONY: all
				all: prog1 prog2 prog3 prog4

	- 结果：
		- 这个Makefile需要编译四个程序：prog1 prog2 prog3 prog4
		- 可以使用 "make all" 来编译所有的目标
		- 可以使用 "make prog2" 来单独编译目标 "prog2"

	- GNU官方Makefile中的目标：
		1. “all”：编译所有的目标
		2. “clean”：删除所有被make创建的文件
		3. “install”：安装已经编译好的程序
		4. “print”：列出改变过的源文件
		5. “tar”：把源程序打包备份（.tar）
		6. “dist”：把源程序打包备份（.gz）
		7. “TAGS”：更新所有目标，以备重编译
		8. “clean”和“test”：测试Makefile的流程

- make的参数

		-b：忽略兼容性
		-m：忽略兼容性
		-B：重编译所有目标
		-C <dir>：指定Makefile所在的目录
		-d：输出所有的调试信息
		-e：用环境变量的变量值覆盖掉Makefile中的变量值
		-f=<file>：指定需要执行的Makefile
		-h：help
		-i：忽略所有错误
		-I <dir>：包含进一个搜索目标
		-j 4:4线程工作
		-k：出错也不停止运行（如果一个目标失败了，那么依赖这个目标的相关目标就不会被执行了）
		-n：只按顺序输出命令序列，并不执行
		-q：检查目标是否需要更新
		-r：禁用隐含规则
		-R：禁用变量隐含规则
		-s：不输出命令的输出
		-S：取消 "-k" 的作用
		-t：touch某个目标，使之时间戳变为最新
		-v：版本

##隐含规则

- 编译c程序的隐含规则
	
		<xyz>.o ---> <xyz>.c #.o的依赖 自动推导出 .c
		$(CC) -c $(CPPFLAGS) $(CFLAGS)

- 编译c++程序的隐含规则

		<xyz>.o ---> <xyz>.cc
		$(CXX) -c $(CPPFLAGS) $(CFLAGS)

- 汇编的隐含规则

		1. <xyz>.o ---> <xyz>.s
			$(AS) $(ASFLAGS)

		2. <xyz>.s ---> <xyz>.S
			$(AS) $(ASFLAGS)	

- 链接 .o 文件的隐含规则（ld命令：链接）

		<xyz> ---> <xyz>.o
		$(CC) $(LDFLAGS) <xyz>.o $(LOADLIBS) $(LDLIBS)

- 一个目标只能出现一次：防止无限递归

##自动化变量

1. $@：目标
2. $%：库文件 (.a .lib) 中的目标

		举例：
			foo.a (bar.o)
			$% 是 bar.o

3. $^：依赖（all）
4. $?：依赖（newer than 目标）
5. $<：第一个依赖（类似于c++中的迭代器）
6. $+：依赖（all，并且不去除重复的）