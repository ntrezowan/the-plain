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
3. Run Netcat to check if you can send logs using all the configured remote ports of Splunk server;
```
mrh13j@f5san1:Active:In Sync] ~ # route echo '<0>F5SAN1-UDP' | nc -w 1 -u 146.201.74.20 9514
```
To test if you can reach a UDP remote port, run the following and then search in Splunk server with “HOST=F5san* F5SAN1-UDP”

```
mrh13j@f5san1:Active:In Sync] ~ # route echo '<0>F5SAN1-TCP' | nc -w 1 -t 146.201.74.20 9515
```
To test if you can reach a TCP remote port, run the following and then search in Splunk server with “HOST=F5san* F5SAN1-TCP”

If you have a HA environment, do this for all the F5 servers in the group.

4. You can also do a tcpdump to check if custom logs generated from F5 can reach Splunk server. 
Run the following in one terminal;
```
mrh13j@f5san1:Active:In Sync] ~ # tcpdump -A -nni vlan_1184 host 146.201.74.20 and port 9515
```

Now send a custom log using another terminal;
```
mrh13j@f5san1:Active:In Sync] ~ # logger -p local0.notice "DUMPLING”
```

To test if you can reach a remote UDP port, run the following and then search in Splunk server with “HOST=F5san* DUMPLING”

If nc or tcpdump works, it means F5 can send logs to Splunk without any issue.


---

#### Add Splunk server to F5

1. In F5, check syslog-ng global and local configuration;
```
[mrh13j@f5san1:Active:In Sync] ~ # cat /var/run/config/syslog-ng.conf

log_fifo_size(2048); - Log file size
# local0.* - syslog configuration
# local1.* - apm configuration
# local3.* - asm configuration
```

2. Check if there is any pre-configured remote log server; 
```
mrh13j@(f5san1)(cfg-sync In Sync)(Active)(/Common)(tmos)# list /sys syslog remote-servers
sys syslog {
    remote-servers none
}
```

If there is any remote log server, we need to remove it because it does not allow us to set severity level of outgoing logs. To remove remote-servers, run the following;
```
mrh13j@(f5san1)(cfg-sync In Sync)(Active)(/Common)(tmos)# modify /sys syslog remote-servers none
mrh13j@(f5san1)(cfg-sync In Sync)(Active)(/Common)(tmos)# save /sys config
```

3.	Now add the remote server as a “include” which will allow us to filter outgoing logs;
```
mrh13j@(f5san1)(cfg-sync In Sync)(Active)(/Common)(tmos)# list /sys syslog all-properties

sys syslog {
    auth-priv-from notice
    auth-priv-to emerg
    clustered-host-slot enabled
    clustered-message-slot disabled
    console-log enabled
    cron-from warning
    cron-to emerg
    daemon-from notice
    daemon-to emerg
    description none
    include "
    filter f_ssl_acc_req {
        not (facility(local6) and level(info) and filter(f_httpd_ssl_acc)) and
        not (facility(local6) and level(info) and filter(f_httpd_ssl_req)) and
        level(info..emerg);
    };
    destination d_remote_loghost {
        tcp(\"146.201.74.20\" port(9515));
        udp(\"146.201.74.20\" port(9514));
    };
    log {
        source(s_syslog_pipe);
        filter(f_ssl_acc_req);
        destination(d_remote_loghost);
    };
    "
    iso-date enabled
    kern-from debug
    kern-to emerg
    local6-from notice
    local6-to emerg
    mail-from notice
    mail-to emerg
    messages-from notice
    messages-to warning
    remote-servers none
    user-log-from notice
    user-log-to emerg
```

Here, 146.201.74.20 is the Splunk server IP and F5 will send logs to 9514/udp and 9515/tcp port of Splunk. In filter, we set the severity level from informational to emergency for syslog (/var/logs/ltm).

4.	Change the date format to iso-date;
```
mrh13j@(f5san02)(cfg-sync In Sync)(Standby)(/Common)(tmos)# modify sys syslog iso-date enabled
```

5.	In Splunk, modify the inputs.conf file for Splunk Add-on to add the following source type;

F5 Source Type ->
```
System log (/var/log/ltm) - f5:bigip:syslog
APM log (/var/log/apm) - f5:bigip:apm:syslog
ASM log (/var/log/asm) - f5:bigip:asm:syslog
```

Inputs.conf ->
```
[udp://9514]
disabled = true
connection_host=ip
sourcetype = f5:bigip:syslog

[udp://9514]
disabled = true
connection_host=ip
sourcetype = f5:bigip:apm:syslog

[tcp://9515]
disabled = true
connection_host=ip
sourcetype = f5:bigip:asm:syslog
```

In here, syslog and APM is using port 9514/udp and ASM is using 9515/tcp.

6.	Go to Splunk and do the following searches to verify that syslog is showing up in Splunk;
a.	Do a search “host=f5san* mcpd” to see if it’s getting mcpd logs
b.	Do a search “host=f5san* tmm*” to see if it’s getting tmm logs

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
