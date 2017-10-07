---
title: "PowerShell scripts"
comments: false
description: "PowerShell scripts"
keywords: "sharepoint, 2016, smtp, nintex, workflow, email, farm"
published: true
---
#### Find operating system, OS architecture and service pack version of remote computer from text file;

```
$server= Get-Content C:\serverlist.txt
    foreach ($i in $server)
    {
    Get-WmiObject win32_operatingsystem -computer $i | Format-Table csname, caption,OSArchitecture,ServicePackMajorVersion,ProductType -AutoSize
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

---

#### Testing if a server can send email using the SMTP server;
```
$cre = Get-Credential
$comp= hostname
Send-MailMessage -To you@example.com -From me@example.com -SmtpServer smtp.example.com -Credential $cre -Subject "Testing SMTP from $comp" -Body "Did you got it?"
```

---

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

---

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

---

#### Check if RDP is running in a remote machine
```
param(
     [parameter(Mandatory=$true,ValueFromPipeline=$true)][string[]]$Computer
     )
$results = @()
foreach($name in $Computer){
        $result = "" | select Name,RDP
        $result.name = $name
        try{
           $socket = New-Object Net.Sockets.TcpClient($name, 3389)
           if($socket -eq $null){
                 $result.RDP = $false
           }else{
                 $result.RDP = $true
                 $socket.close()
           }
        }
        catch{
                 $result.RDP = $false
        }
        $results += $result
}
return $results
```

Sample out:

```
Computer[0]: Server1
Computer[1]: 

Name        RDP
----        ---
Server1     True
```

If it returns `TRUE`, then default RDP port 3389 is open to accept connection.

---

