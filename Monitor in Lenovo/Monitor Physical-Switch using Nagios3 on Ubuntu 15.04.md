#Monitor Physical-Switch using Nagios3 on Ubuntu 15.04
------
## Install and Setup Nagios3
<https://github.com/zhaoshulin/blog/blob/master/Install%20and%20Setup%20Nagios3%20in%20Ubuntu15.md>

## Install and Setup snmpd
	sudo apt-get install snmpd snmp-mibs-downloader
	
	sudo vi /etc/default/snmpd 
		# change 2 lines:
		export MIBS=UCD-SNMP-MIB
		SNMPDOPTS='-Lsd -Lf /dev/null -u snmp -I -smux -p /var/run/snmpd.pid -c /etc/snmp/snmpd.conf'
		
## Monitor Switch using check_traffic.sh
	1. sudo cp check_traffic.sh /usr/lib/nagios/plugins/
        
	2. sudo vi /etc/nagios3/conf.d/lenovo_switch.cfg
		# new file:
		define host{				use                     generic-host                   
				host_name               lenovo_switch				alias                   Lenovo Switch				address                 10.240.253.174				max_check_attempts      10
				retry_interval          1        }		define service{				use                             generic-service				host_name                       lenovo_switch				service_description             Check Traffic on Index-193				check_command                   check_traffic				notifications_enabled            1				normal_check_interval           5				retry_interval                  1        }      
        
    3. sudo vi /etc/nagios3/command.cfg
		# add:
		define command{        command_name    check_traffic        command_line   $USER1$/check_traffic.sh -V 2c -C public -H 10.240.253.174 -I 193 -w 1200,1500 -c 1700,1800 -K -b        }  
      
    4. sudo /etc/init.d/nagios3 restart

Monitoring-result demo:
   
![](file:///Users/zhaoshulin/Desktop/Lenovo/Monitor Physical-Switch using Nagios3 in Ubuntu 15.04/fig.png) 