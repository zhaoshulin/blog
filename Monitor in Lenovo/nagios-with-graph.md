# adding nagiosgraph to nagios3
## install
- `./install.pl --check-prereq`:
	- `sudo apt-get install rrdtool librrds-perl libgd-gd2-perl libcgi-pm-perl libnagios-object-perl`
	- until it doesn't say this: `one or more problems were detected!`
- `sudo ./install.pl`
	- just keep pressing `Enter`
- `sudo vi /etc/nagios3/nagios.cfg` to add this:

		# process nagios performance data using nagiosgraph		process_performance_data=1		service_perfdata_file=/tmp/perfdata.log		service_perfdata_file_template=$LASTSERVICECHECK$||$HOSTNAME$||		$SERVICEDESC$||$SERVICEOUTPUT$||$SERVICEPERFDATA$		service_perfdata_file_mode=a		service_perfdata_file_processing_interval=30		service_perfdata_file_processing_command=process-service-perfdata-for-nagiosgraph 
		
- `sudo vi /etc/nagios3/command.cfg` to add this:

		# command to process nagios performance data for nagiosgraph		define command { 		  command_name process-service-perfdata-for-nagiosgraph		  command_line /usr/local/nagiosgraph/bin/insert.pl		}
		
- `sudo vi /etc/apache2/apache2.conf` to add this:

		Include /usr/local/nagiosgraph/etc/nagiosgraph-apache.conf
		
- to check configure:
	- `/usr/sbin/nagios3 -v /etc/nagios3/nagios.cfg`
- `sudo /etc/init.d/nagios3 restart`
- `sudo /etc/init.d/apach2 restart`

> ps: check out your `cgi` file-location is: `/usr/local/nagiosgraph/cgi/` or `/usr/local/nagiosgraph/cgi-bin`. This might be the reason of most of your problems.


## setup
- `sudo vi /etc/nagios3/conf.d/graphed-service.cfg` to add:

		define service{
			name graphed-service
			action_url /nagiosgraph/cgi/show.cgi?host=$HOSTNAME$&service=$SERVICEDESC$
			register 0
		}
- if you want to plot a service, just change `use xxx` to `use xxx,graphed-service`. For example, I want to plot `lenovo_server`'s `check_dall_disks` service, I should do this: `sudo vi /etc/nagios3/conf.d/lenovo_server.conf` to change this line:

		-- use generic-service
		++ use generic-service,graphed-service

## Figures
- open <http://localhost/nagios3/> :

![](file:///Users/zhaoshulin/Desktop/Lenovo/monitor-graphed/fig1.png)

- open <http://localhost/nagiosgraph/cgi/show.cgi?host=lenovo_server&service=Current%20Load> :

![](file:///Users/zhaoshulin/Desktop/Lenovo/monitor-graphed/fig2.png)