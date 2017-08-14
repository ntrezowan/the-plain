---
title: "Installing IPplan on CentOS"
---
Operating System: _CentOS_  
Web Server: _Apache_  
Database: _MySQL_  

___

1. Download [ipplanv4.92a](http://iptrack.sourceforge.net/) and extract the tarball in `/var/www/html`;
```
# cd /var/www/html
# tar zxvf ipplan-4.92a.tar.gz
```

2. Change the ownership and permissions of the IPplan directory;
```
# chown -R root:apache /var/www/html/ipplan/
# chmod -R 750 /var/www/html/ipplan/
```

3. Now create the IPplan database;
```
# mysqladmin -u root -p create ipplan
```

4. And a user for IPplan database;
```
# mysql -u root -p ipplan
```

5. Give all the rights to the IPplan user on the IPplan database;
```
mysql> GRANT SELECT,INSERT,UPDATE,DELETE on ipplan.* \ TO ipplan@xyz.com IDENTIFIED by ‘blink182’;
mysql> flush privileges;
mysql> exit
```

6. Change the IPplan `config.php` file and insert the IPplan mysql connection settings:
```
define("DBF_TYPE", 'maxsql');
define("DBF_HOST", 'xyz.com');
define("DBF_USER", 'ipplan');
define("DBF_NAME", 'ipplan');
define("DBF_PASSWORD", 'blink182');
```

7. Install Nmap and SNMP for `ipplan-poller` to work;
```
# yum -y install nmap
# yum -y install snmpd
```

8. The IPplan poller uses a custom file to know which IP addresses the scan. Create a `.txt` file and the following output show the syntax for the `xyz-networks.txt` file;
```
172.16.0.0/16
192.168.0.0/24
```

9. Now set a crontab for IPplan poller within a defined interval;
```
# crontab -e
```
Add the following lines in crontab;
```
0 9,12,15 * * * php /var/www/html/ipplan/contrib/ipplan-poller.php - hostname -c 1 -f /var/www/html/ipplan/ingane-networks.txt
```

10. Finally configure `snmpd.conf`;
```
# vi /etc/snmp/snmpd.conf
```
The following lines should be included in `snmpd.conf` file;
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

11. Restart MySQL, Apache and SNMP service;
```
# service httpd restart
# service mysqld restart
# service snmpd restart
```

12. To backup the IPplan database;
```
# mysqldump -u ipplan -pblink182 ipplan > ipplan.sql
```

13. To restore the IPplan database;
```
# mysql -u ipplan -pblink182 ipplan < ipplan.sql
```

___

Go to `http://xyz.com/ipplan/admin/install.php` to continue with the further web based installation.
