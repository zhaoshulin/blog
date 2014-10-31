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




