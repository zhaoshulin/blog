# Monitor-tools' file-system
------------
##`PandoraFMS`
- `extras`: 外部扩展包
	- `chrome_extension`
	- `firefox_extension`
	- `fedora_official_specs`
	- ...
- `pandora_agents`: collects and sends data to server，最核心的模块
	- `android`
	- `embedded`
	- `pc`
	- `unix`:
		- `pandora_agent.sh`: main script, collecting data.
		- `pandora_agent.conf`: including 2 parts:
			- General Parameters:
				- `server_ip`
				- `server_path`
				- `temporal`
				- `interval`
				- `agent_name`
				- `checksum`
			- Module Definitions:
				- `module_name <name>`
				- `module_type <type>`
				- `module_exec <command>`
				- `module_description <text>`
	- `win32`
	- ...
- `pandora_console`:前端 dash board
- `pandora_server`:receives data from agent, 用这些数据干一些杂活，比如分析、打包等
- `tests`：测试用

## `Icinga`
- `etc`
	- `initsystem`：前期准备，检查必要的依赖
	- `icinga2`：配置文件
		- `conf.d`
			- `app.conf`
			- `commands.conf`
			- `hosts.conf`
			- ...
- `icinga-app`：开启监控进程，C++ 实现
- `icinga-installer`：安装，C++ 实现
- `lib`：C++ 库
- `plugins`：外部插件
	- `check_disk.cpp`
	- `check_network.cpp`
	- ...
- `third-party`：用到的第三方的东西，比如 `cmake`, `socket`

##`Zabbix`
- `conf`：配置文件
	- `zabbix_agentd.conf`
	- `zabbix_proxy.conf`
	- `zabbix_server.conf`
- `database`
	- `mysql`
	- `oracle`
- `frontends`：前端实现
- `include`：C语言头文件
- `man`：帮助文件
- `misc`：最小指令集：
	- `images`：用到的图片
	- `init.d`：启动监控进程
		- `fedora`
		- `ubuntu`: 启动客户端和服务器端
			- `agentd`
			- `server`
	- `snmptrap`：启动 SNMPtrap 服务：
		- `snmptrap.sh`
		- `trap_receiver.pl`
- `src`：具体实现，大部分是C语言：
	- `zabbix_agent`
	- `zabbix_proxy`
	- `zabbix_server`
	- `zabbix_sender`
	- ...
- `upgrades`：升级包
	- `dbpatches`

##`Naemon`：最值得我们借鉴！
> The Naemon core is a network monitoring tool based on the Nagios 4 core, but with many bug fixes, new features, and performance enhancements. If you today use Nagios, you should switch to Naemon to get bugfixes, new features, and performance enhancements.

- `Naemon` 与 `Nagios` 框架对应关系图

![](file:///Users/zhaoshulin/Desktop/Lenovo/多款监控工具框架分析/fig.png) 

## 其他可供参考的开源软件
- `Munin`
- `StatsD`
- `Collectd`
- `Graphite`

