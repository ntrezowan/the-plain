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

##### Install required packages

1. Install packages necessary for Nagios (such as Apache, GCC etc.);
```
# sudo apt-get install apache2
# sudo apt-get install libapache2-mod-php5
# sudo apt-get install build-essential
# sudo apt-get install libgd2-dev
# sudo apt-get install libgd2-xpm-dev
```

___

#### Create user/group for Nagios

1. Create a new Nagios user and set the password;
```
# /usr/sbin/useradd -m -s /bin/bash nagios
# passwd nagios
```

2 Create a new Nagios group;
```
# /usr/sbin/groupadd nagios
```

3. Make Nagios user as a member of Nagios group;
```
# /usr/sbin/usermod -G nagios nagios
```

4. Create a new `nagcmd` group for allowing external commands to be submitted through the web interface. Add both the Nagios user and the Apache user to the group;
```
# /usr/sbin/groupadd nagcmd
# /usr/sbin/usermod -a -G nagcmd nagios
# /usr/sbin/usermod -a -G nagcmd www-data
```

___

#### Install Nagios

1. Create a directory and download Nagios in there;
```
# mkdir ~/downloads
# cd ~/downloads
# wget http://prdownloads.sourceforge.net/sourceforge/nagios/nagios-3.2.3.tar.gz
# wget http://prdownloads.sourceforge.net/sourceforge/nagiosplug/nagios-plugins-1.4.11.tar.gz
# cd ~/downloads
```

2. Extract the Nagios source code tarball;
```
# tar xzf nagios-3.x.x.tar.gz
# cd nagios-3.x.x
```

3. Run the Nagios configure script;
```
# ./configure --with-command-group=nagcmd
# make all
# make install
# make install-init
# make install-config
# make install-commandmode
```

4. Edit the `/usr/local/nagios/etc/objects/contacts.cfg` config file according to your requirement;
```
# vi /usr/local/nagios/etc/objects/contacts.cfg
```

5. Install the Nagios web config file in the apache `conf.d` directory;
```
# make install-webconf
```
6. Create a `nagiosadmin` account for logging into the Nagios web interface using `.htaccess`;
```
# htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

7. Restart Apache;
```
# /etc/init.d/apache2 reload
```

9. Extract the Nagios plugins source code tarball;
```
# cd ~/downloads
# tar xzf nagios-plugins-1.x.xx.tar.gz
# cd nagios-plugins-1.x.xx/
```

10. Compile and install Nagios plugins;
```
#./configure --with-nagios-user=nagios --with-nagios-group=nagios
# make
# make install
```

11. Set Nagios to automatically start when the system boots;
```
# ln -s /etc/init.d/nagios /etc/rcS.d/S99nagios
```

12. Verify Nagios configuration files;
```
# /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

There should no be any error in the output, otherwise check your configuration.

___


#### Start Nagios

```
# /etc/init.d/nagios start
```

___


Now browse to http://localhost/nagios and log in as `nagiosadmin` using password `nagios`.
