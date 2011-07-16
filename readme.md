# Automatically manage OVH Seocondary server #

Ovh-dnsupdater is a small utility to automate the addition and removal of domains to the (free) secondary DNS server offered by OVH.

## Requirements ##

The utility is written in php (so you need the cli version of php). Additionally, you must have the soap extension. In short:

	php >= 5.1
	php-soap >= 5.1

## Installing ##

To install ovh-dnsupdater it is necessary to perform three basic steps:

### Install the required files ###

First of all, you should copy the required files into appropiate locations in your filesystem. You can change the routes displayed here so long as you update the configuration file accordingly.

	cp etc/ovh-dnsupdater /etc/ovh-dnsupdater
	chmod 640 /etc/ovh-dnsupdater
	cp ovh-dnsupdater /usr/sbin/ovh-dnsupdater
	chmod 755 /usr/sbin/ovh-dnsupdater
	mkdir /var/lib/ovh-dnsupdater
	chmod 750 /var/lib/ovh-dnsupdater

### Setup the utility ###

The next step is to modify the configuration file to reflect your ovh soapi credentials, and the details of your server. All these settings are found in the /etc/ovh-dnsupdater (or wherever you have put the configuration) file.

### First run and cronjob setup ###

Next, you have to "hard" run the utility manually so that it catches all your already setup domains. It is recommended that you perform a dry run at this (the utility logs what would be done, but does nothing) to avoid breaking anything:

	ovh-dnsupdater --hard --dry-run

Inspect your /var/log/syslog to see what the script would do. If everything looks fine and there are changes, you can re-run the utility without the dry run option:

	ovh-dnsupdater --hard

Finally, you should setup cron jobs to automate the execution of the utility. The recommended way is to setup a frequently-running (every 5mins or so) task *without* the "--hard" option (this avoids any soapi request unless changes are detected), with a way less frequent (daily) task running with the "--hard" option (that ensures our cached information is up to date with the ovh servers).

The previous ideas can be easily specified in /etc/crontab by adding the following lines:

	*/5	*	*	*	*  root /usr/sbin/ovh-dnsupdater
	7	2	*	*	*  root /usr/sbin/ovh-dnsupdater --hard

Alternatively, some systems allow you to define cronjob tasks by creating files inside the /etc/cron.d folder. If this is your case, then you can just copy the included sample file:

	cp cron/ovh-dnsupdater /etc/cron.d/ovh-dnsupdater
	chmod 755 /etc/cron.d/ovh-dnsupdater

## Ovh-dnsupdater usage ##

	Usage: 
	  ./ovh-dnsupdater [options]
	Options:
	  --conf-file=<file>
		Load the configuration from file <file> instead of /etc/ovh-dnsupdater
	  --dry-run    
		Just show what would be done, without actually doing anything.
	  -h, --help   
		Show this help message.
	  --hard       
		Foce downloading the list of already setup domains using the API.

## How it works ##

Ovh-dnsupdater works by monitoring your bind's zone folder, and updating ovh's secondary server setup whenever a change is detected. Essentially, each tiem that the utility runs, it performs the following steps:

 1. Fetch the list of domains that ovh knows about
 2. Fetch the list of localy setup domains
 3. Compute the difference between these lists
 4. Add/Remove domains so that both lists end up synchronized

The first step may be done in two different ways, depending on if the "hard" option has been specified or not. When in "hard" mode, the list of domains that ovh knows about is fetched using the SOAPI. In contrast, when "hard" is not specified, the utility simply loads the list as cached by the last time it ran.

## To Do ##

The major limitation known at the time of this writing is that ovh-dnsupdater is not able to work with multiple secondary dns servers. Therefore, if you have domains setup using different secondary servers (sdns1.ovh.net and sdns2.ovh.net) and want to use this utility, you will have to re-configure the domains so that they all use the same backup server. This limitation will be fixed if someone requests me to do so.

Another possible point of improvement is that of logging. Specifically, it would be interesting to log to stdout when configuring/testing the utility, to avoid the additional step of having to look at the system's log file. Additionally, it would be interesting to give a configuration option that made the utility output any changes done to stdout, so that the admin is notified by the resulting cron mail.
