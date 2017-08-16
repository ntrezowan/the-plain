---
title: "Installing Nagios on Ubuntu Server"
comments: false
description: "Installing Nagios on Ubuntu Server"
keywords: "nagios, install, ubuntu, apache, mysql"
---
> Operating System: _Ubuntu Server_  
> Web Server: _Apache_  
> Database: _MySQL_  

___

#### A. Install dependencies

1. Install packages necessary to run Nagios (such as Apache, GCC etc.);
```
# sudo apt-get install apache2
# sudo apt-get install libapache2-mod-php5
# sudo apt-get install build-essential
# sudo apt-get install libgd2-dev
# sudo apt-get install libgd2-xpm-dev
```

#### B. Create user and group for Nagios

1. Create a new Nagios user and set the password;
```
# /usr/sbin/useradd -m -s /bin/bash nagios
# passwd nagios
```

2. Create a new Nagios group;
```
# /usr/sbin/groupadd nagios
```

3. Make Nagios user as a member of Nagios group;
```
# /usr/sbin/usermod -G nagios nagios
```

4. Create a new `nagcmd` group for allowing external commands to pass through the web interface. Add both the Nagios user and the Apache user into this group;
```
# /usr/sbin/groupadd nagcmd
# /usr/sbin/usermod -a -G nagcmd nagios
# /usr/sbin/usermod -a -G nagcmd www-data
```

#### C. Install Nagios

1. Create a directory and download Nagios;
```
# mkdir ~/downloads
# cd ~/downloads
# wget http://prdownloads.sourceforge.net/sourceforge/nagios/nagios-3.2.3.tar.gz
# wget http://prdownloads.sourceforge.net/sourceforge/nagiosplug/nagios-plugins-1.4.11.tar.gz
# cd ~/downloads
```

2. Extract Nagios core;
```
# tar xzf nagios-3.x.x.tar.gz
# cd nagios-3.x.x
```

3. Run Nagios configure script;
```
# ./configure --with-command-group=nagcmd
# make all
# make install
# make install-init
# make install-config
# make install-commandmode
```

4. Edit `/usr/local/nagios/etc/objects/contacts.cfg` config file according to your requirement;
```
# vi /usr/local/nagios/etc/objects/contacts.cfg
```

5. Install Nagios web config file in apache `conf.d` directory;
```
# make install-webconf
```

6. Create `nagiosadmin` account for logging into the Nagios web interface using `.htaccess`;
```
# htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

7. Restart Apache web server;
```
# /etc/init.d/apache2 reload
```

8. Extract Nagios plugins;
```
# cd ~/downloads
# tar xzf nagios-plugins-1.x.xx.tar.gz
# cd nagios-plugins-1.x.xx/
```

9. Compile and install Nagios plugins;
```
#./configure --with-nagios-user=nagios --with-nagios-group=nagios
# make
# make install
```

10. Set Nagios daemon to automatically start when the system boots;
```
# ln -s /etc/init.d/nagios /etc/rcS.d/S99nagios
```

11. Verify Nagios configuration;
```
# /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```  
If there is any error in the output, check your configuration.

12. Start Nagios;
```
# /etc/init.d/nagios start
```

___


Browse to [http://localhost/nagios](http://localhost/nagios) and log in as `nagiosadmin` using password `nagios`.
