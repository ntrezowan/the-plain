---
title: "Random SharePoint 2016 Troubleshooting"
comments: false
description: "Random SharePoint 2016 Troubleshooting"
keywords: "sharepoint, 2016, blob, cache, web application, availability, monitor, start, stop, distributed cache, os, version, architecture, hostname, domain, cname, processors, ram, disks, ip address, licensing status, uac, firewall, snmp, rdp scan, administrators, users, disk, spacec, smtp"
published: true
---
#### Database is in compatibility range and upgrade is recommended
1. Go to `CA > Upgrade and Migration > Review database status` and find if the database `Status` is showing as `Database is in compatibility range and upgrade is recommended`.
2. It can also be checked by running the following command in `SharePoint 2016 Management Shell`;
```

```
3. Run the following command in `SharePoint 2016 Management Shell` to upgrade the database;
```
Get-SPWebApplication "http://WebApplication1:9999" | Get-SPContentDatabase | Upgrade-SPContentDatabase
```
Sample output:
```
Confirm
Are you sure you want to perform this action?
Performing the operation "Upgrade-SPContentDatabase" on target
"SharePoint_AdminContent".
[Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "Y"):Y
100.00% : SPContentDatabase Name=SharePoint_AdminContent
Finalizing the upgrade...
```
4. Go to `CA > Upgrade and Migration > Review database status` and now it will show as `No action required`


---

#### Testing if a server can send email using SMTP server
```
$cre = Get-Credential
$comp= hostname
Send-MailMessage -To you@example.com -From me@example.com -SmtpServer smtp.example.com -Credential $cre -Subject "Testing SMTP from $comp" -Body "Did you got it?"
```
