---
title: "PowerShell scripts"
comments: false
description: "PowerShell scripts"
keywords: "sharepoint, 2016, smtp, nintex, workflow, email, farm"
published: true
---
#### Find operating system, OS architecture and service pack version of remote computer from text file;

```
$server= get-content C:\serverlist.txt
    foreach ($i in $server)
    {
    get-wmiobject win32_operatingsystem -computer $i | Format-Table csname, caption,OSArchitecture,ServicePackMajorVersion,ProductType -AutoSize
    }
```
Sample output:
```
csname caption                  OSArchitecture ServicePackMajorVersion ProductType
------ -------                  -------------- ----------------------- -----------
HAL9   Microsoft Windows 10 Pro 64-bit                               0           1
```
Here,
ProductType 1 -> WorkStation  
ProductType 2 -> Domain Controller  
ProductType 3 -> Server


#### Testing if a server can send email using the SMTP server;
```
$cre = get-credential
Send-MailMessage -To you@example.com -From me@example.com -SmtpServer smtp.example.com -Credential $cre -Subject "Testing SMTP from $hostname " -Body "Did you got it?"
```

#### Print remote machine Name, Device ID, Volume Name, Total Size and Free Size (in GB)
```
clear 
$cre = Get-Credential
$file = Get-Content  C:\serverlist.txt
 
foreach ($args in $file) { 
Get-WmiObject win32_logicaldisk -Credential $cre -ComputerName $args -Filter "Drivetype=3"  |  
ft SystemName,DeviceID,VolumeName,@{Label="Total Size";Expression={$_.Size / 1gb -as [int] }},@{Label="Free Size";Expression={$_.freespace / 1gb -as [int] }} -AutoSize 
}
```

Sample output:
```
SystemName DeviceID VolumeName    Total Size Free Size
---------- -------- ----------    ---------- ---------
Server1    C:                             60        22
Server1    D:       Data and Logs         60        43
Server1    F:       SysState              30        30
```

#### Find users in the Administrators group of remote machines
```
clear 
$cre = Get-Credential 

function Get-LocalUsers { 
    param( 
    [Parameter(Mandatory=$true,valuefrompipeline=$true)] 
    [string]$strComputer) 
    begin {} 
    Process { 
        $adminlist ="" 
        $computer = [ADSI]("WinNT://" + $strComputer + ",computer") 
        $AdminGroup = $computer.psbase.children.find("Administrators") 
        $Adminmembers= $AdminGroup.psbase.invoke("Members") | %{$_.GetType().InvokeMember("Name", 'GetProperty', $null, $_, $null)} 
        foreach ($admin in $Adminmembers) { $adminlist = $adminlist + $admin + "," } 
        $Computer = New-Object psobject 
        $computer | Add-Member noteproperty ComputerName $strComputer 
        $computer | Add-Member noteproperty Administrators $adminlist 
        Write-Output $computer
        } 
end {} 
} 
Get-Content C:\serverlist.txt | Get-LocalUsers | Export-Csv C:\server-admin.csv
```




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
