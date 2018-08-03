---
title: "Configure F5 for Splunk"
comments: false
description: "Configure F5 for Splunk"
keywords: "F5, hsl, hish speed logging, request logging, management port logging, asm logging, apm logging"
published: false
---
#### Check network connectivity

1. Ping Splunk server from F5;
```
[mrh13j@f5san1:Active:In Sync] ~ # ping 146.201.74.20
PING 146.201.74.20 (146.201.74.20) 56(84) bytes of data.
64 bytes from 146.201.74.20: icmp_seq=1 ttl=63 time=0.838 ms
^C
```

If ping is down, it does not necessarily mean that no log will reach Splunk server because F5 will send logs to a predefined TCP/UDP port.

2. Check how F5 is reaching Splunk server;
```
[mrh13j@f5san1:Active:In Sync] ~ # ip route get 146.201.74.20
146.201.74.20 via 146.201.111.1 dev vlan_1184  src 146.201.111.253
    cache
```

If there is no route to Splunk server, add a static route and verify;
```
mrh13j@f5san1:Active:In Sync] ~ # ip route add 146.201.74.20/32 via vlan_1184
mrh13j@f5san1:Active:In Sync] ~ # route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         146.201.111.1   0.0.0.0         UG    0      0        0 vlan_1184
0.0.0.0         10.1.114.1      0.0.0.0         UG    9      0        0 mgmt
10.1.114.0      0.0.0.0         255.255.254.0   U     0      0        0 mgmt
10.1.190.0      0.0.0.0         255.255.255.128 U     0      0        0 vlan_2998
10.112.0.112    0.0.0.0         255.255.255.240 U     0      0        0 vlan_ha
127.1.1.0       0.0.0.0         255.255.255.0   U     0      0        0 tmm
127.7.0.0       127.1.1.253     255.255.0.0     UG    0      0        0 tmm
127.20.0.0      0.0.0.0         255.255.0.0     U     0      0        0 tmm_bp
146.201.74.20   146.201.111.1     255.255.255.255 UGH   9      0        0 vlan_1184
146.201.74.22   10.1.114.1      255.255.255.255 UGH   9      0        0 mgmt
146.201.111.0   0.0.0.0         255.255.255.0   U     0      0        0 vlan_1184
```




1.	Make sure in which partition you want to upload the html file. It will be easier if VIP, iRule and iFile are in the same partition.  

2.	Go to `System > File Management > iFile List`. Click on Import and upload your html file (maintenance.html) and give it a Name (maintenance.html).  

3.	Go to `Local Traffic > iRules > iFile List`. Click on Create, select the iFile you have just uploaded from File Name and give it a Name (maintenance.html).  

4.	Go to `Local Traffic > iRules > iRules List`. Create a new iRule and give it a Name (maintenance). Here is a sample iRule which will load the HTML page when all  pool members are down (this decision will be made based on `HTTP Profile` configuration);  
```
when LB_FAILED {
  if { [active_members [LB::server pool]] == 0 } {
  HTTP::respond 503 content [ifile get " maintenance.html"]}
  }
```

5.	Go to `Local Traffic > Virtual Servers > Virtual Servers List`. Select the VIP you want to apply iRule to and go to Resource Tab. In the iRule section, click on Manage, add the iRule and click Finished.  

NB: If your html file is in a different partition, then you have to use something like `/Common/maintenance.html` in the iRule. We are also using `503` because it will tell the search crawler not to cache this page.  

---

#### Create redirection page if a URI is down without iFile


1.	Go to `System > File Management > iFile List`. Click on import and upload your image (companylogo.png) and give it a Name (companylogo.png). You can also upload more files (such as css or js) to support your HTML template.  

2.	Go to `Local Traffic > iRules > iFile List`. Click on Create, select the iFile you have just uploaded from File Name and give it a Name (companylogo.png).  

3.	Go to `Local Traffic > iRules > iRules List`. Create a new iRule and give it a Name (maintenance). Here is a sample iRule which will load the html content and then redirect to another URI (`https://discovery.com`) when a user visits a particular URI;  
```
when HTTP_REQUEST {
    if { [HTTP::query] equals "providerId=https://www.example.com/" }{  
    HTTP::respond 200 content {

    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
        <html xmlns="http://www.w3.org/1999/xhtml">
            <head>
                <title>Title goes here</title>
                <META http-equiv="refresh" content="15;URL=https://discovery.com">
            </head>
            <body>
                <div align="center">
                    <div id="maintenanceHeader" align="center">
                        <img src="companylogo.png"
                    </div>
                    <div id="maintenanceBody" align="center">
                        <strong>This site is in maintenance now.</strong>  
                        <br /><br />
                        You will be redirected to https://discovery.com automatically in 15 seconds.
                    </div>
                </div>
            </body>
        </html>
    }
  }
}
```

4.	Go to `Local Traffic > Virtual Servers > Virtual Servers List`. Select the VIP you want to apply iRule to and go to Resource Tab. In the iRule section, click on Manage, add the iRule and click Finished.  

NB: If you do not want to be redirected, remove `<META http-equiv="refresh" content="15;URL=https://discovery.com">` from head.
