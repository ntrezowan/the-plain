---
title: "Configuring SMTP in SharePoint 2016"
comments: false
description: "Configuring SMTP in SharePoint 2016"
keywords: "sharepoint, 2016, smtp, email, farm"
published: true
---
1. Go to `CA > System Settings > Configure outgoing e-mail settings` and configure as following;  
```
Outbound SMTP server: smtp@example.com
From address: sharepoint@example.com
Reply-to-address: sharepoint@example.com
Use TLS connection encryption: Yes (if you want to encrypt emails)
SMTP server port: 25 (Default)
Character set: 65000 (Unicode UTF-8)
```

2. Go to `CA > Application Management > Manage Web Application` to view all the web applications. Select a web application and click on `General Settings > Outgoing E-Mail settings` from the ribbon to verify web application specific SMTP setting;  
```
Outbound SMTP server: smtp@test.com
From address: sharepoint@test.com
Reply-to-address: sharepoint@test.com
Use TLS connection encryption: No (emails will be send at plain text)
SMTP server port: 25 (Default)
Character set: 65000 (Unicode UTF-8)
```
You can use different SMTP server for each web application as shown above. 
3. Save the following script as `SMTPTest.ps1` and run it from `SharePoint 2016 Management Shell`;  
```
$sd = New-Object System.Collections.Specialized.StringDictionary
$sd.Add("to","xyz@example.com")
$sd.Add("from","sharepoint@xyz.com")
$sd.Add("subject","Test Email from WA: SharePoint1")
$w = Get-SPWeb http://SharePoint1
$body = "Test email sent from SharePoint1"
try {
    [Microsoft.SharePoint.Utilities.SPUtility]::SendEmail($w,$sd,$body)
}
finally {
    $w.Dispose()
}
```  
Run the script from `SharePoint 2016 Management Shell`;
```
PS > .\SMTPTest.ps1
True
```
If it returns `False`, reboot the server. 

4. Remove the web fronts from load balancer and install the update on the web fronts first. This way, if we encounter any issue after patching, it will not break any of the application servers. Reboot the web fronts if necessary.  
After reboot, check if the updates are properly installed by going to `Control Panel > Program and Features > View Installed Updates`. Also, open CA from the web fronts to see if IIS is working or not
