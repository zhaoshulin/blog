## Setup your network in Ubuntu
- Let network can be managed:
	- `sudo vi /etc/NetworkManager/NetworkManager.conf`
		- change from `managed=false` to `managed=true`
- Edit your `ethx`:
	- `sudo vi /etc/network/interfaces`
- Edit your DNS:
	- `sudo vi /etc/resolv.conf`
- Restart your network:
	- `sudo /etc/init.d/networking restart`

##Check out network information
- IP + netmask:
			
		ifconfig
- Gateway:

		netstat -nr
		
- DNS:

		nslookup www.baidu.com