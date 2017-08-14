---
title: "Installing Cacti on CentOS"
comments: false
description: "Installing Cacti on CentOS"
keywords: "cacti, install, centos, apache, mysql"
---

#### Installing prerequisites

1. Define a repo from [dag.wieers.org](http://www.blogger.com/dag.wieers.org) to install Cacti dependencies;
```
# vi /etc/yum.repos.d/dag.repo
```
Add the following lines in `dag.repo` file;
```
[dag] name=Dag RPM Repository for Red Hat Enterprise Linux
baseurl=http://apt.sw.be/redhat/el5/en/i386/dag
gpgcheck=1
gpgkey=http://dag.wieers.com/rpm/packages/RPM-GPG-KEY.dag.txt
enabled=1
```

2. Update the OS;
```
# yum update
```

3. Now install prerequisite packages (such as Apache, MySQL, rrdtool etc.);
```
# yum install php httpd mysql mysql-server php-mysql vim-enhanced initscripts perl-rrdtool rrdtool initscripts
```

___

#### Install Cacti

1. Download [Cacti](http://www.cacti.net) and untar;
```
# tar xzvf cacti-0.8.7e.tar.gz
# mv cacti-0.8.7e cacti
# mv cacti /var/www/html
```

2. Now create a Cacti user and give appropriate permission for a Cacti group;
```
# /usr/sbin/groupadd cacti
# /usr/sbin/useradd -g cacti cactiuser
# passwd cactiuser
```

3. Change the ownership of the `/var/www/html/cacti/rra/` and `/var/www/html/cacti/log/` directories to the Cacti user;
```
# cd /var/www/html/cacti
# chown -R cactiuser rra/ log/
```

4. Create a MySQL database for Cacti;
```
# mysqladmin --user=root --password=123456 create cacti
```

5. Go to `/var/www/html/cacti`, and use the `cacti.sql` to create tables for the database;
```
# cd /var/www/html/cacti
# mysql --user=root --password=blink182 cacti < cacti.sql
```

6. Now create a mySQL username and password for Cacti;
```
# mysql --user=root --password=blink182
mysql> GRANT ALL ON cacti.* TO cactiuser@xyz.com IDENTIFIED BY 'blink182';
mysql> flush privileges;
mysql> exit
```

7. Edit `/var/www/html/cacti/include/config.php` file;
```
$database_type = "mysql";
$database_default = "cacti";
$database_hostname = "xyz.com";
$database_username = "cactiuser";
$database_password = "blink182";
$database_port = "3306";
```

8. Edit php config file at `/etc/php.ini` to allow more memory usage for Cacti;
```
# vi /etc/php.ini
```
Change the `memory_limit` to `128M`;
```
memory_limit = 128M
```

___

#### Install SNMP

1. Insall SNMP using the following command;
```
# yum install net-snmp net-snmp-utils php-snmp net-snmp-libs
```

2. To Configure SNMP, open the `/etc/snmp/snmpd.conf` file;
```
# vi /etc/snmp/snmpd.conf
```
The following lines should be included in the `snmpd.conf` file;
```
com2sec local localhost public
group MyRWGroup v1 local
group MyRWGroup v2c local
group MYRWGroup usm local
view all include .1 80
access MYRWGroup "" any noauth exact all all none
syslocation Unknown
syscontact admin@xyz.com
pass .1.3.6.1.4.1.4413.4.1 /usr/bin/ucd5820stat
```

___

#### Configure Apache for Cacti

1. Open `/etc/httpd/conf.d/cacti.conf` file;
```
# vi /etc/httpd/conf.d/cacti.conf
```
The following lines should be included in `cacti.conf` file;
``` 
Alias /cacti /usr/share/cacti
< Directory /usr/share/cacti/>
Order Deny,Allow
Deny from all
Allow from 172.0.0.0/8
< /Directory>
```

2. Open `/etc/cron.d/cacti` to set a cronjob;
```
# vi /etc/cron.d/cacti
```
The following line should be included in the crontab;
```
*/5 * * * * cacti /usr/bin/php /usr/share/cacti/poller.php > /dev/null 2>&1
```

3. Now restart all the services;
```
# service httpd restart
# service mysqld restart
# service snmpd restart
```

___

#### Backup/restore Cacti database and rrd

1. To backup the Cacti MySQL database;
```
# mysqldump -p cacti > cacti.mysql
```

2. To backup the Cacti rrd data;
```
# mkdir rra
# cd rra
# cp /var/lib/cacti/rra/*.rrd ./
# chown rezowan:rezowan *.rrd
# for i in *.rrd; do rrdtool dump "$i" "$i".xml; done
```

3. To restore the Cacti MySQL database;
```
# mysql -p cacti < cacti.mysql
```

4. To restore the Cacti rrd data;
```
# cd rra
# for i in *.xml; do rrdtool restore "$i" "$i".rrd; done
# for i in *.rrd.xml.rrd; do mv "$i" `echo "$i" | sed s/.xml.rrd//g`; done
# cp *.rrd /var/lib/cacti/rra/
# chown www-data:www-data /var/lib/cacti/rra/*.rrd
```

___

Now browse to http://localhost/nagios and log in as `admin` using password `admin`.
