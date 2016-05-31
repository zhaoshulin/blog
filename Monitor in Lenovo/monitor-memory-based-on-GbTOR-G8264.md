##Monitor all links using `check_traffic.sh`
- Check out which links is up:
	- `snmpwalk -v 2c -c public -H 10.240.253.174 ifOperStatus`
	- We'll find out only 4 links are up: `145, 146, 147, 148`
	- OK, so we'll monitor all these links.
- `sudo vi /etc/nagios3/command.cfg`

		# add these 4 links:
		define command{        command_name    check_traffic        command_line   $USER1$/check_traffic.sh -V 2c -C public -H 10.240.253.174 -I 145 -w 1200,1500 -c 1700,1800 -K -b        } 
        
        define command{        command_name    check_traffic        command_line   $USER1$/check_traffic.sh -V 2c -C public -H 10.240.253.174 -I 146 -w 1200,1500 -c 1700,1800 -K -b        } 
        
        define command{        command_name    check_traffic        command_line   $USER1$/check_traffic.sh -V 2c -C public -H 10.240.253.174 -I 147 -w 1200,1500 -c 1700,1800 -K -b        } 
        
        define command{        command_name    check_traffic        command_line   $USER1$/check_traffic.sh -V 2c -C public -H 10.240.253.174 -I 148 -w 1200,1500 -c 1700,1800 -K -b        } 
        
- result figure:

![](file:///Users/zhaoshulin/Desktop/Lenovo/monitor-memory/fig1.jpg)       
        
##What can we get from `GbTOR-GB8264.mib`
- I found `GbTOR-GB8264.mib` from internet, this file is provided to monitor our switch using SNMP.
- A few steps to load this additionl `.mib` file:
	- `sudo apt-get install snmp-mibs-downloader`
	- `download-mibs`
	- `sudo chmod 777 GbTOR-GB8264.mib`
	- `sudo cp GbTOR-GB8264.mib /usr/share/snmp/mibs/`
	- `sudo vi /etc/snmp/snmp.conf` to add one line:
		- `mibs +ALL`
	- `sudo snmptranslate -Tp`
		- Maybe there will be 3 errors, but don't worry. That's because these files are not IETF official files. And we won't use them. So we can simply delete them. (According to <http://docs.linuxconsulting.mn.it/notes/net-snmp-errors>)
	- `sudo /etc/init.d/snmpd restart`
- Check out what can we get about Memory Statisitics Group (just for our first-try) from `GbTOR-GB8264.mib`:
	- there are 8 objects about Memory Statisitics Group:
		- `totalMemoryStats`: The total memory in bytes.
		- `memoryFreeStats`: The free memory in bytes.
		- `memorySharedStats`: The shared memory in bytes.
		- `memoryBufferStats`: The buffer memory in bytes.
		- `swapTotalStats`: The total swap memory in bytes.
		- `swapFreeStats`: The free swap memory in bytes.
		- `highTotalStats`: The total high memory in bytes.
		- `highFreeStats`: The free high memory in bytes.
	- So, I write a `check_lenovo_switch_memory` bash script, result figure:

![](file:///Users/zhaoshulin/Desktop/Lenovo/monitor-memory/fig2.png)

- Besides of memory, there are much more information we can get from `GbTOR-GB8264.mib`, such as:
	- CPU util percent
	- NTP Statisitics
	- Stats for per thread CPU utilization
	- failoverInfo
	- VLAN 
	- ... (I'll try more parameters)