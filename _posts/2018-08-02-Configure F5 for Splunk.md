---
title: "Configure F5 for Splunk"
comments: false
description: "Configure F5 for Splunk"
keywords: "F5, hsl, hish speed logging, request logging, management port logging, asm logging, apm logging"
published: true
---

Splunk IP = 10.10.10.1
Splunk TCP Port=9515 (for ASM)  
Splunk UDP Port=9514 (for syslog, HSL, and APM)  
F5SERV1 self IP for VLAN1 = 11.11.11.2  
F5SERV2 self IP for VLAN1 = 11.11.11.3  
F5SERV Floating IP for VLAN1 = 11.11.11.1  


#### Check network connectivity
1. Ping Splunk server from F5;  
```
[user@f5serv1:Active:In Sync] ~ # ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=63 time=0.838 ms
^C
```
If ping is down, it does not necessarily mean that no log will go to Splunk server because F5 will send logs to a predefined TCP/UDP port.

2. Check how F5 is reaching Splunk server;  
```
[user@f5serv1:Active:In Sync] ~ # ip route get 10.10.10.1
10.10.10.1 via 20.20.20.1 prd vlan1  src 11.11.11.2
cache
```
Here `11.11.11.2` is the TMM interface (also self non-floating IP for F5SERV1) which will be the source when logs are send to `10.10.10.1`.
If there is no route to Splunk server, add a static route and verify;  
```
user@f5serv1:Active:In Sync] ~ # ip route add 10.10.10.1/32 via vlan1
user@f5serv1:Active:In Sync] ~ # route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         20.20.20.1      0.0.0.0         UG    0      0        0 vlan1
0.0.0.0         10.1.1.1        0.0.0.0         UG    9      0        0 mgmt
10.1.1.0        0.0.0.0         255.255.254.0   U     0      0        0 mgmt
127.1.1.0       0.0.0.0         255.255.255.0   U     0      0        0 tmm
127.7.0.0       127.1.1.253     255.255.0.0     UG    0      0        0 tmm
127.20.0.0      0.0.0.0         255.255.0.0     U     0      0        0 tmm_bp
10.10.10.1      0.0.0.0         255.255.255.255 U     0      0        0 vlan1
20.20.20.0      0.0.0.0         255.255.255.0   U     0      0        0 vlan1
```
3. Run Netcat to check if you can send logs to a specific remote port of Splunk server.  
To test if you can reach a UDP remote port, run the following and then search in Splunk server with `HOST=f5serv1* f5serv1-UDP`. 
```
user@f5serv1:Active:In Sync] ~ # route echo '<0>f5serv1-UDP' | nc -w 1 -u 10.10.10.1 9514
```
To test if you can reach a TCP remote port, run the following and then search in Splunk server with `HOST=f5serv1* f5serv1-TCP`. 
```
user@f5serv1:Active:In Sync] ~ # route echo '<0>f5serv1-TCP' | nc -w 1 -t 10.10.10.1 9515
```
4. You can also do a tcpdump to check if you can send logs to a specific remote port of Splunk server.  
Run the following in one terminal to monitor the TCP port;
```
user@f5serv1:Active:In Sync] ~ # tcpdump -A -nni vlan1 host 10.10.10.1 and port 9515
```
While tcpdump is running, open another terminal and run the following and check if this log shows in tcpdump outout. Also check /var/log/ltm and search in Splunk server with `HOST=f5serv1* DUMPLING`.
```
user@f5serv1:Active:In Sync] ~ # logger -p local0.notice "DUMPLING”
```
If nc or tcpdump works, it means F5 can send logs to specific Splunk ports without any issue.

---

#### Add Splunk server to F5

1. In F5, check `syslog-ng` global and local configuration;  
```
[mrh13j@f5san1:Active:In Sync] ~ # cat /var/run/config/syslog-ng.conf
```
```
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

#### Configure HSL using TMM

1.	Create	a pool and add the Splunk server as a backend server of the pool;
Go to `Local Traffic > Pools`. Click on Create, select Advanced from Configuration and complete the following;
```
Name=splunk_pool
Pool Health Monitor=gateway_icmp
Node Health Monitor=udp
Address=146.201.74.20
Service Port=9514
```

2.	Create an iRule and add it to a VIP. 

Here is a sample iRule which does Apache like HTTP Request/Response logging;
```
when CLIENT_ACCEPTED {
    set client_address [IP::client_addr]
    set vip [IP::local_addr]
    set hsl [HSL::open -proto TCP -pool /Common/splunk_pool]
}
when HTTP_REQUEST {
    set http_host [HTTP::host]:[TCP::local_port]
    set http_uri [HTTP::uri]
    set http_url $http_host$http_uri
    set http_method [HTTP::method]
    set http_version [HTTP::version]
    set http_user_agent [HTTP::header "User-Agent"]
    set http_content_type [HTTP::header "Content-Type"]
    set http_referrer [HTTP::header "Referer"]
    set tcp_start_time [clock clicks -milliseconds]
    set req_start_time [clock format [clock seconds] -format "%Y/%m/%d %H:%M:%S"]
    set cookie [HTTP::cookie names]
    set user [HTTP::username]
    set virtual_server [LB::server]
      
    if { [HTTP::header Content-Length] > 0 } then {
        set req_length [HTTP::header "Content-Length"]
    } else {
        set req_length 0
    }
}
when HTTP_RESPONSE {
#    set res_start_time [clock format [clock seconds] -format "%Y/%m/%d %H:%M:%S"]
    set node [IP::server_addr]
    set node_port [TCP::server_port]
    set http_status [HTTP::status]
    set req_elapsed_time [expr {[clock clicks -milliseconds] - $tcp_start_time}]
    if { [HTTP::header Content-Length] > 0 } then {
        set res_length [HTTP::header "Content-Length"]
    } else {
        set res_length 0
    }
    #log local0.info "SYSLOGDD CLIENT_IP=$client_address, VIP=$vip, VIP_NAME=\"$virtual_server\", SERVER_NODE=$node, SERVER_NODE_PORT=$node_port, HTTP_URL=$http_url, HTTP_VERSION=$http_version, HTTP_STATUS=$http_status, HTTP_METHOD=$http_method, HTTP_CONTENT_TYPE=$http_content_type, HTTP_USER_AGENT=\"$http_user_agent\", HTTP_REFERRER=\"$http_referrer\", COOKIE=\"$cookie\", REQUEST_START_TIME=$req_start_time, RESPONSE_START_TIME=$res_start_time, REQUEST_ELAPSED_TIME=$req_elapsed_time, BYTES_IN=$req_length, BYTES_OUT=$res_length\r\n"
    set hsl [HSL::open -proto UDP -pool /Common/splunk_pool]
    HSL::send $hsl "<190> CLIENT_IP=$client_address, VIP=$vip, VIP_NAME=\"$virtual_server\", SERVER_NODE=$node, SERVER_NODE_PORT=$node_port, HTTP_URL=$http_url, HTTP_VERSION=$http_version, HTTP_STATUS=$http_status, HTTP_METHOD=$http_method, HTTP_CONTENT_TYPE=$http_content_type, HTTP_USER_AGENT=\"$http_user_agent\", HTTP_REFERRER=\"$http_referrer\", COOKIE=\"$cookie\", REQUEST_START_TIME=$req_start_time,REQUEST_ELAPSED_TIME=$req_elapsed_time, BYTES_IN=$req_length, BYTES_OUT=$res_length\r\n"
}
when LB_FAILED {
    set hsl [HSL::open -proto UDP -pool /Common/splunk_pool]
    HSL::send $hsl "<190> CLIENT_IP=$client_address, VIP=$vip, VIP_NAME=\"$virtual_server\", SERVER_NODE=$node, SERVER_NODE_PORT=$node_port, HTTP_URL=$http_url, HTTP_VERSION=$http_version, HTTP_STATUS=$http_status, HTTP_METHOD=$http_method, HTTP_CONTENT_TYPE=$http_content_type, HTTP_USER_AGENT=\"$http_user_agent\", HTTP_REFERRER=\"$http_referrer\", COOKIE=\"$cookie\", REQUEST_START_TIME=$req_start_time,REQUEST_ELAPSED_TIME=$req_elapsed_time, BYTES_IN=$req_length, BYTES_OUT=$res_length\r\n"
}
```

3.	Visit the VIP where you have applied the iRule and then go to Splunk and search for HOST=f5san* appstst.

---

#### Configure HSL using Management Port

1.	Verify that F5 is using management port to reach Splunk;
```
mrh13j@(f5san1)(cfg-sync In Sync)(Active)(/Common)(tmos)# ip route get 146.201.74.20
146.201.74.20 via 10.1.114.1 dev mgmt  src 10.1.114.11
```

If not, configure F5 to so that it uses manament port to reach Splunk;
```
mrh13j@(f5san1)(cfg-sync In Sync)(Active)(/Common)(tmos)# create /sys log-config destination management-port mgmt ip-address 146.201.74.20 port 9514 protocol udp
mrh13j@(f5san1)(cfg-sync In Sync)(Active)(/Common)(tmos)# list sys management-route
sys management-route splunk {
    gateway 10.1.114.1
    network 146.201.74.22/32
}
mrh13j@(f5san1)(cfg-sync In Sync)(Active)(/Common)(tmos)# save sys config
```

2.	Create a Log destination
Go to `System > Logs > Configuration > Log Destinations`. Create New and do the following;
```
Name=splunk_hsl_via_mgmt_port
Type=Management Port
Address=146.201.74.20
Port 9515
Protocol=TCP
```

3.	Create a Log publisher
Go to `System > Logs > Configuration> Log Publisher`. Create New and do the following
```
Name=splunk_hsl_publisher
Destination=splunk_hsl_via_mgmt_port
```
4.	Create an iRule and add it to a VIP. 

Here is a sample iRule which does Apache like HTTP Request/Response logging;
```
when CLIENT_ACCEPTED {
    set client_address [IP::client_addr]
    set vip [IP::local_addr]
}
when HTTP_REQUEST {
    set http_host [HTTP::host]:[TCP::local_port]
    set http_uri [HTTP::uri]
    set http_url $http_host$http_uri
    set http_method [HTTP::method]
    set http_version [HTTP::version]
    set http_user_agent [HTTP::header "User-Agent"]
    set http_content_type [HTTP::header "Content-Type"]
    set http_referrer [HTTP::header "Referer"]
    set tcp_start_time [clock clicks -milliseconds]
    set req_start_time [clock format [clock seconds] -format "%Y/%m/%d %H:%M:%S"]
    set cookie [HTTP::cookie names]
    set user [HTTP::username]
    set virtual_server [LB::server]
      
    if { [HTTP::header Content-Length] > 0 } then {
        set req_length [HTTP::header "Content-Length"]
    } else {
        set req_length 0
    }
}
when HTTP_RESPONSE {
#    set res_start_time [clock format [clock seconds] -format "%Y/%m/%d %H:%M:%S"]
    set node [IP::server_addr]
    set node_port [TCP::server_port]
    set http_status [HTTP::status]
    set req_elapsed_time [expr {[clock clicks -milliseconds] - $tcp_start_time}]
    if { [HTTP::header Content-Length] > 0 } then {
        set res_length [HTTP::header "Content-Length"]
    } else {
        set res_length 0
    }
    #log local0.info "SYSLOGDD CLIENT_IP=$client_address, VIP=$vip, VIP_NAME=\"$virtual_server\", SERVER_NODE=$node, SERVER_NODE_PORT=$node_port, HTTP_URL=$http_url, HTTP_VERSION=$http_version, HTTP_STATUS=$http_status, HTTP_METHOD=$http_method, HTTP_CONTENT_TYPE=$http_content_type, HTTP_USER_AGENT=\"$http_user_agent\", HTTP_REFERRER=\"$http_referrer\", COOKIE=\"$cookie\", REQUEST_START_TIME=$req_start_time, RESPONSE_START_TIME=$res_start_time, REQUEST_ELAPSED_TIME=$req_elapsed_time, BYTES_IN=$req_length, BYTES_OUT=$res_length\r\n"
    set hsl [HSL::open -publisher /Common/splunk_pool]
    HSL::send $hsl "<190> CLIENT_IP=$client_address, VIP=$vip, VIP_NAME=\"$virtual_server\", SERVER_NODE=$node, SERVER_NODE_PORT=$node_port, HTTP_URL=$http_url, HTTP_VERSION=$http_version, HTTP_STATUS=$http_status, HTTP_METHOD=$http_method, HTTP_CONTENT_TYPE=$http_content_type, HTTP_USER_AGENT=\"$http_user_agent\", HTTP_REFERRER=\"$http_referrer\", COOKIE=\"$cookie\", REQUEST_START_TIME=$req_start_time,REQUEST_ELAPSED_TIME=$req_elapsed_time, BYTES_IN=$req_length, BYTES_OUT=$res_length\r\n"
}
when LB_FAILED {
    set hsl [HSL::open -publisher /Common/splunk_pool]
    HSL::send $hsl "<190> CLIENT_IP=$client_address, VIP=$vip, VIP_NAME=\"$virtual_server\", SERVER_NODE=$node, SERVER_NODE_PORT=$node_port, HTTP_URL=$http_url, HTTP_VERSION=$http_version, HTTP_STATUS=$http_status, HTTP_METHOD=$http_method, HTTP_CONTENT_TYPE=$http_content_type, HTTP_USER_AGENT=\"$http_user_agent\", HTTP_REFERRER=\"$http_referrer\", COOKIE=\"$cookie\", REQUEST_START_TIME=$req_start_time,REQUEST_ELAPSED_TIME=$req_elapsed_time, BYTES_IN=$req_length, BYTES_OUT=$res_length\r\n"
}
```
5.	Visit the VIP where you have applied the iRule and then go to Splunk and search for HOST=f5san* appstst.



---
#### Configure Request Logging

1.	Go to Local Traffic > Profiles > Other > Request Logging. Click on Create and complete the following;
```
Nmae=splunk_http_request_logging
Parent Profile=request-log
Request Logging=Enabled
Template=
$DATE_NCSA REQUEST -> CLIENT = $CLIENT_IP:$CLIENT_PORT, VIP = $VIRTUAL_IP:$VIRTUAL_PORT, HTTP_VERSION = $HTTP_VERSION, HTTP_METHOD = $HTTP_METHOD, HTTP_KEEPALIVE = $HTTP_KEEPALIVE, HTTP_PATH = $HTTP_PATH, HTTP_QUERY = $HTTP_QUERY, HTTP_REQUEST = $HTTP_REQUEST, HTTP_URI = $HTTP_URI

HSL Protocol=UDP
Pool Name=splunk_pool
Response Logging=Enabled
Template=
$DATE_NCSA, RESPONSE -> CLIENT = $CLIENT_IP:$CLIENT_PORT, VIP = $VIRTUAL_IP:$VIRTUAL_PORT, SERVER = $SERVER_IP:$SERVER_PORT, HTTP_VERSION = $HTTP_VERSION, HTTP_METHOD = $HTTP_METHOD, HTTP_KEEPALIVE = $HTTP_KEEPALIVE, HTTP_PATH = $HTTP_PATH, HTTP_QUERY = $HTTP_QUERY, HTTP_REQUEST = $HTTP_REQUEST, HTTP_STATUS = $HTTP_STATUS, HTTP_URI = $HTTP_URI, SNAT_IP = $SNAT_IP:$SNAT_PORT, F5_HOSTNAME = $BIGIP_HOSTNAME, RESPONSE_TIME = $RESPONSE_MSECS, RESPONSE_SIZE = $RESPONSE_SIZE

HSL Protocol=UDP
Pool Name=splunk_pool
```

2.	Go to `Local Traffic > Virtual Servers`. Click on the VIP which you want to use Request Logging. Select Advanced of Configuration and then choose the Request Logging Profile as splunk_http_request_logging

3.	Visit the VIP and then search in Splunk with `HOST=F5san* REQUEST`
