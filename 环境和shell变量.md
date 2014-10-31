#环境和shell变量

----------
##本地shell变量
本地变量：只在用户现在的shell生命期的脚本中使用。
###自定义一个本地变量
在终端内输入：
	
	variable-name = value

###查看该本地变量

	echo ${variable-name}

###清除一个本地变量

	unset variable-name

###显示所有的本地变量

	set

###设置只读变量

	variable-name = value
	readonly variable-name

###查看所有的只读变量

	readonly

##环境变量
环境变量用于所有的用户，必须用 export 命令导出。

1. 命令行设置 => 注销时丢失
2. 修改/etc/profile => 永久保存，开机即执行

###自定义一个环境变量

	VARIABLE-NAME = value; export VARIABLE-NAME (一般大写)

###查看某一个环境变量

	echo VARIABLE-NAME

###清除某一个环境变量

	unset VARIABLE-NAME

###查看所有的环境变量

	env

###嵌入shell变量


