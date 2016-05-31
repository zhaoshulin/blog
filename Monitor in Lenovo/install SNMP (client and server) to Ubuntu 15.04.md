# install SNMP (client and server) to Ubuntu 15.04
-------
- First, Let's define:
	- `server`: it is monitored. 
	- `client`: it is monitoring `server`.

## install:
- `server`:
 - `sudo apt-get install snmpd`
- `client`:
	- `sudo apt-get install snmp snmp-mibs-downloader`
	- `ufw disable`

##setup:
- `server`:
	- `sudo cp /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf-backup`
	- `sudo vim /etc/snmp/snmpd.conf` to change these lines:

			# agentAddress udp:127.0.0.1:161
			agentAddress udp:161,udp6:[::1]:161
			
			# view systemonly included .1.3.6.1.2.1.1
			# view systemonly included .1.3.6.1.2.1.25.1
			view systemonly included .1
			
			# rocommunity public localhost
			rocommunity lenovo default -V systemonly
			rocommunity6 lenovo default -V sysetmonly
	- `sudo /etc/init.d/snmpd restart`	
- `client`:
	- `sudo vim /etc/snmp/snmp.conf` to change 1 line:

			# mibs :
			
##test:
- `client`:
	- `snmpwalk -v2c -c public <your-server-ip-address>`: that would be `Timeout` error. Because there is no `public` in `snmpd.conf`
	- `snmpwalk -v2c -c lenovo <your-server-ip-address>`: that would be successful.