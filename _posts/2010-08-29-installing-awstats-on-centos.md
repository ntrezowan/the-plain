---
title: "Installing AWStats on CentOS"
comments: false
description: "Installing AWStats on CentOS"
keywords: "awstats, install, centos, apache, mysql"
---
> Operating System: _CentOS_  
> Web Server: _Apache_  
> Database: _MySQL_  

___

1. Install EPEL software repository, and then install AWStats;
```
# yum install epel-release
# yum install awstats
```

2. Download [GeoIP](http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz) and [GeoLiteCity](http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz) from maxmind.com and move it to `/tmp`
```
# mkdir /var/www/html/GeoIP
# cd /var/www/html/GeoIP
# mv /tmp/GeoIP.dat.gz GeoIP.dat.gz
# mv /tmp/GeoLiteCity.dat.gz GeoLiteCity.dat.gz
# gunzip GeoIP.dat.gz
# gunzip GeoLiteCity.dat.gz
```

3. Edit default AWStats config;
```
# cd /etc/awstats
# cp awstats.model.conf awstats.model.conf.orig
# cp awstats.model.conf.orig awstats.stats.xyz.com.conf
```

3. Now a create directories for the new subdomain `stats.xyz.com`;
```
# mkdir /var/www/html/stats.xyz.com
# mkdir /var/www/html/stats.xyz.com/cgi-bin
```

4. Copy the AWStats program files to newly created `stats.xyz.com/cgi-bin` directory;
```
# cd /usr/share/awstats/wwwroot/
# cp -R * /var/www/html/stats.xyz.com/
```

5. Open Apache configuration file and add an entry for this new subdomain;
```
# vi /etc/httpd/conf/httpd.conf  
```
Add the following lines in `httpd.conf`;
```
<virtualhost *:80>
ServerAdmin admin@xyz.com
ServerName xyz.com
ServerAlias www.xyz.com
DocumentRoot /var/www/html/xyz.com
ScriptAlias /cgi-bin/ /var/www/html/xyz.com/cgi-bin/
CustomLog logs/xyz.com_access_log combined
ErrorLog logs/xyz.com_error_log
</VirtualHost>
```
For AWStats subdomain, add the following in `httpd.conf`;
```
<virtualhost *:80>
ServerAdmin admin.awstats@xyz.com
ServerName stats.xyz.com
DocumentRoot /var/www/html/stats.xyz.com
ScriptAlias /cgi-bin/ /var/www/html/stats.xyz.com/cgi-bin/
CustomLog logs/xyz.com.stats_access_log combined
ErrorLog logs/xyz.com.stats_error_log
Alias /classes "/var/www/html/stats.xyz.com/classes/"
Alias /css "/var/www/html/stats.xyz.com/css/"
Alias /icon "/var/www/html/stats.xyz.com/icon/"
ScriptAlias /awstats/ "/var/www/html/stats.xyz.com/cgi-bin/"
</VirtualHost>
```

6. Next, edit `/etc/httpd/conf.d/awstats.conf` to include directory path for new subdomain;
```
# vi /etc/httpd/conf.d/awstats.conf
```
Add the following lines;
```
<Directory "/var/www/html/stats.xyz.com">
DirectoryIndex awstats.pl
Options ExecCGI
Options FollowSymLinks
Options None
AllowOverride None
Order allow,deny
Allow from all
```

7. Now edit the `awstats.stats.xyz.com.conf` and add the following lines;
```
# vi /etc/awstats/awstats.stats.xyz.com.conf
```
Add/edit the following lines;
```
LogFile="/var/log/httpd/xyz.com_access_log"
SiteDomain="stats.xyz.com"
HostAliases="stats.xyz.com"
DirData="/var/www/html/stats.xyz.com/"
LoadPlugin="geoip GEOIP_STANDARD /var/www/html/GeoIP/GeoIP.dat"
LoadPlugin="geoip_city_maxmind GEOIP_STANDARD /var/www/html/GeoIP/GeoLiteCity.dat"
```

8. Restart Apache web server;
```
# service httpd restart
```

9. Run AWStats perl script to automatically update stats;
```
# perl /var/www/html/stats.xyz.com/cgi-bin/awstats.pl -config=stats.xyz.com -update
```

10. To update AWStats automatically every hour, create a cronjob;
```
# cd /etc/cron.hourly && vi 01awstats
```
Add the following line;
```
perl /var/www/html/stats.xyz.com/cgi-bin/awstats.pl -config=stats.xyz.com -update > /dev/null 2>&1
```
Now, set permission and restart crond;
```
# chmod 755 01awstatsservice crond restart
```

___

Browse to `http://stats.xyz.com/cgi-bin/awstats.pl` to see site stats for  `xyz.com`.
