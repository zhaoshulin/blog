# Install and Setup Nagios3 in Ubuntu15.04
------
To be clear:

- `father` is the monitor-server.
- `son` is the one who is monitored.
	- so, `father` monitors `son`
- `$` means: user
- `#` means: root

## Install
- on `father`:
	- `$ sudo apt-get install apache2 nagios3 nagios-nrpe-plugin`
- on `son`:
	- `$ sudo apt-get install nagios-nrpe-server`

## Start Services
- on `son`:
	- `$ sudo /etc/init.d/nagios-nrpe-server start`
- on `father`:
	- `$ sudo /etc/init.d/nagios3 start`
	- open dashboard-link to observe: `http://localhost/nagios3/`


## Add a new-host
- `father: $ sudo cp /etc/nagios3/conf.d/localhost_nagios2.cfg /etc/nagios3/conf.d/my_host.cfg`
- `father: $ vim /etc/nagios3/conf/my_host.cfg`
	- rewrite: from `hostname localhost` to `hostname my_host`
- `son: $ sudo /etc/init.d/nagios-nrpe-server restart`
- `father: $ sudo /etc/init.d/nagios3 restart`

##Add a new-service
- `father: $ sudo vim /usr/lib/nagios/plugins/check_xxx.py` to coding your plugin
- add to `/etc/nagios3/commands.cfg`:
	
		define command{
			command_name check_xxx
			command_line $USER1$/check_xxx.py
		}
- add to `/etc/nagios3/conf.d/my_host.cfg`:

		define service{
			host_name my_host
			service_description xxxDetails
			check_command check_xxx
			use generic-service
			notification_interval 0
		}
		
- `son: $ sudo /etc/init.d/nagios-nrpe-server restart`
- `father: $ sudo /etc/init.d/nagios3 restart`