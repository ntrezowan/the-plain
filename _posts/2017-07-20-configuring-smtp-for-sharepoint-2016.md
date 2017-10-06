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
Go to `CA > General Application Settings > Farm Search Administration > Search Server Application` and now `Administrative Status` will show as `Paused: for external request`.

4. Remove the web fronts from load balancer and install the update on the web fronts first. This way, if we encounter any issue after patching, it will not break any of the application servers. Reboot the web fronts if necessary.  
After reboot, check if the updates are properly installed by going to `Control Panel > Program and Features > View Installed Updates`. Also, open CA from the web fronts to see if IIS is working or not

5. Install the update in Application Servers and reboot if necessary.  
After reboot, check if the updates are properly installed by going to `Control Panel > Program and Features > View Installed Updates`. Also, open CA from the application servers to see if IIS is working or not 

6. Remove `WSS_Logging` database from Availability Group if there are two SQL servers in a cluster with Always On Availability

7. At first, run `SharePoint 2016 Products Configuration Wizard` at the server where CA is installed. When it finishes, open CA to verify the server status.  
You can also run Configuration Wizard from Management Shell using the following command;   
```
PS > psconfig.exe -cmd helpcollections -installall -cmd secureresources -cmd services -install -cmd installfeatures -cmd applicationcontent -install -cmd upgrade -inplace b2b -force -wait
```

8. Run Configuration Wizard in the web fronts and other application servers

9. Run the following command to check `Configuration Database version`;  
```
PS > (get-spfarm).buildversion
```

10. Go to `CA > Upgrade and Migration > Check product and patch installation status` and verify the installed `Version` for each server in the farm

11. Go to `CA > Upgrade and Migration > Check Upgrade Status` and verify if all the servers in the farm shows as `Succeeded`

12. Add `WSS_Logging` database to the Availability Group if there are two SQL servers in a cluster with Always On Availability

13. Start `Search Service Application` using the following command on the Search servers in the farm;  
```
PS > Resume-SPEnterpriseSearchServiceApplication –Identity “Search Service Application”
```

14. Go to `CA > General Application Settings > Farm Search Administrator > Search Service Administration` and verify that `Administrative status` is shows as `Running`

15. Go to `CA > General Application Settings > Farm Search Administrator > Search Service Administration > Content Source` and start a `Full Crawl`

