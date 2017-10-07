---
title: "PowerShell scripts"
comments: false
description: "PowerShell scripts"
keywords: "sharepoint, 2016, smtp, nintex, workflow, email, farm"
published: true
---
#### Find operating system, OS architecture and service pack version of remote computers

```
$server= get-content C:\serverlist.txt
    foreach ($i in $server)
    {
    get-wmiobject win32_operatingsystem -computer $i | select ` csname,caption,osarchitecture,servicepackmajorversion
    }
```
Sample output:
```
csname caption                  osarchitecture servicepackmajorversion
------ -------                  -------------- -----------------------
HAL9   Microsoft Windows 10 Pro 64-bit                               0
```

2. Generate a new CSR for `example.com` domain;

2. Go to `CA > Application Management > Manage Web Application` to view all the web applications. Select a web application and click on `General Settings > Outgoing E-Mail settings` from the ribbon to verify web application specific SMTP setting;  
```
Outbound SMTP server: smtp@test.com
From address: sharepoint@test.com
Reply-to-address: sharepoint@test.com
Use TLS connection encryption: No (emails will be send in plain text)
SMTP server port: 25 [Default]
Character set: 65000 (Unicode UTF-8) [Default]
```
You can use different SMTP server for each web application as shown above.

3. Save the following script as `SMTPTest.ps1`. This script will allow us to test if a particular web application can send email using SMTP;  
```
$sd = New-Object System.Collections.Specialized.StringDictionary
$sd.Add("to","xyz@example.com")
$sd.Add("from","sharepoint@appserver")
$sd.Add("subject","Test Email from WebApp1)
$w = Get-SPWeb http://SharePoint1
$body = "Test email sent from WebApp1"
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
If it returns `False`, reboot the server. Sometimes SMTP changes does not reflect on the farm even if we restart `SharePoint Timer Service`.

___

#### Configuring SMTP for Nintex Workflow
If you are using `Nintex Workflow` in your farm, then go to `CA > Nintex Administration > Messaging and notifications` and configure as following;
```
Outbound SMTP server: smtp@example.com
SMTP server requires authentication -> Tick if you have a AD service account for Nintex to send/receive emails
From Address: sharepoint-workflow@example.com
Reply To Address: sharepoint-workflow@example.com
Character set: 65000 (Unicode UTF-8) [Default]
Use css styles in HTML emails: Yes (if Nintex is using HTML formatting in emails)
Location of stylesheet containing email styles: /_layouts/NintexWorkflow/htmleditorstyle.css [Default]
```
