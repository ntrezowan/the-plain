---
title: "Installing Shibboleth Service Provider in RHEL"
comments: false
description: "Installing Shibboleth Service Provider in RHEL"
keywords: "F5, hsl, hish speed logging, request logging, management port logging, asm logging, apm logging, configure"
published: true
---
### Environment
F5SERV1 self IP for VLAN1 = 11.11.11.2  
F5SERV2 self IP for VLAN1 = 11.11.11.3  
F5SERV Floating IP for VLAN1 = 11.11.11.1  
F5SERV1 self IP for management VLAN = 10.1.1.2  
F5SERV2 self IP for management VLAN = 10.1.1.3  
F5SERV Floating IP for management VLAN = 10.1.1.1  
Splunk IP = 10.10.10.1  
Splunk TCP Port = 9515 (for ASM)  
Splunk UDP Port = 9514 (for SYSLOG, HSL and APM) 

---
### Logging Flow
> Virtual Server >> Logging Profile >> Log Publisher >> Log Destination >> Pool >> Splunk Log Server

---
### A. Installation
1.	Install Apache, NTP and Shibboleth
```
yum install httpd ntp shibboleth.x86_64
```
2.	Activate shibd at startup and start the service
```
sudo systemctl enable httpd.service
sudo systemctl enable shibd.service
sudo systemctl enable ntpd.service
```
3.	Start Apache and Shibd
```
sudo service httpd start
sudo service shibd start
sudo service ntpd start
```
4.	Check if the services are running;
```
ps aux | grep httpd
root     18366  0.0  0.4 328408 16532 ?        Ss   15:13   0:00 /usr/sbin/httpd

ps aux | grep shibd
shibd     8104  0.0  0.6 766416 25724 ?        Ssl  Dec07   0:14 /usr/sbin/shibd -p /var/run/shibboleth/shibd.pid -f -w 30
```
5.	Check if mod_shib.so module is loaded in Apache;
```
httpd -M | grep mod_shib
mod_shib (shared)
Syntax OK
```

If the module is missing, check if /etc/httpd/conf.d/shib.conf has the following line;
LoadModule mod_shib /usr/lib64/shibboleth/mod_shib_22.so

If not, you can add the line either in /etc/httpd/conf/httpd.conf or in /etc/httpd/conf.d/shib.conf file but not in both place (Apache does not allow multiple module entry definition)

6.	Check if there is any error;
```
grep -E 'CRIT|ERROR' /var/log/shibboleth/shibd.log
```

7.	Visit the following page;
```
https://example.com/Shibboleth.sso/Session
```
If it returns “A valid session was not found.”, it means shibd is running and working with Apache.

---

### B. Configuration
1. In F5, check `syslog-ng` global and local configuration;  
```
[user@f5serv1:Active:In Sync] ~ # cat /var/run/config/syslog-ng.conf
```
Check default log path of SYSLOG, APM and ASM;  
SYSLOG  -> # local0.*  
APM     -> # local1.*  
ASM     -> # local3.*  

2. Check if there is any pre-configured remote log server in F5; 
```
user@(f5serv1)(cfg-sync In Sync)(Active)(/Common)(tmos)# list /sys syslog remote-servers
sys syslog {
    remote-servers none
}
```
If there is any remote log server, we need to remove it because this type of configuration does not allow us to set severity level of outgoing logs. To remove `remote-servers`, run the following;
```
user@(f5serv1)(cfg-sync In Sync)(Active)(/Common)(tmos)# modify /sys syslog remote-servers none
user@(f5serv1)(cfg-sync In Sync)(Active)(/Common)(tmos)# save /sys config
```
3. Now add Splunk server as a `include` which will allow us to filter outgoing logs;
```
user@(f5serv1)(cfg-sync In Sync)(Active)(/Common)(tmos)# edit /sys syslog all-properties
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
        tcp(\"10.10.10.1\" port(9515));
        udp(\"10.10.10.1\" port(9514));
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
}
```
Here under `include` section, `10.10.10.1` is Splunk server IP and F5 will send logs to `9514/udp` and `9515/tcp` port of Splunk. In filter section, we set the severity level from informational to emergency for SYSLOG (/var/logs/ltm).

4. Change date format to `iso-date`;
```
user@(f5serv1)(cfg-sync In Sync)(Standby)(/Common)(tmos)# modify sys syslog iso-date enabled
user@(f5serv1)(cfg-sync In Sync)(Active)(/Common)(tmos)# save /sys config
```
5. In Splunk, modify `inputs.conf` so that F5 source-type matches with `inputs.conf`;  
F5 Source Type ->
```
SYSLOG  (/var/log/ltm) -> f5:bigip:syslog
APM     (/var/log/apm) -> f5:bigip:apm:syslog
ASM     (/var/log/asm) -> f5:bigip:asm:syslog
```
inputs.conf ->
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
In here, SYSLOG and APM is using `9514/udp` and ASM is using `9515/tcp`.

6. Go to Splunk and search with the following to verify that SYSLOG (/var/log/ltm) shows up in Splunk;
- Search `host=f5serv1* mcpd` to see if it’s getting `mcpd` logs
- Search `host=f5serv1* tmm*` to see if it’s getting `tmm` logs

---

### C. Test
1. Create a pool and add Splunk as a backend server of the pool.
Go to `Local Traffic > Pools`. Click Create, select Advanced from Configuration and configure as following;
```
Name = splunk_pool
Pool Health Monitor = gateway_icmp
Node Health Monitor = udp
Address = 10.10.10.1
Service Port = 9514
```

2. Create an iRule and add it to a VIP. 
Go to `Local Traffic > iRules > iRules List`. Click Create and here is a sample iRule which does Apache like HTTP Request/Response logging;
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
    #log local0.info "SYSLOGOD CLIENT_IP=$client_address, VIP=$vip, VIP_NAME=\"$virtual_server\", SERVER_NODE=$node, SERVER_NODE_PORT=$node_port, HTTP_URL=$http_url, HTTP_VERSION=$http_version, HTTP_STATUS=$http_status, HTTP_METHOD=$http_method, HTTP_CONTENT_TYPE=$http_content_type, HTTP_USER_AGENT=\"$http_user_agent\", HTTP_REFERRER=\"$http_referrer\", COOKIE=\"$cookie\", REQUEST_START_TIME=$req_start_time, RESPONSE_START_TIME=$res_start_time, REQUEST_ELAPSED_TIME=$req_elapsed_time, BYTES_IN=$req_length, BYTES_OUT=$res_length\r\n"
    set hsl [HSL::open -proto UDP -pool /Common/splunk_pool]
    HSL::send $hsl "<190> HSL, CLIENT_IP=$client_address, VIP=$vip, VIP_NAME=\"$virtual_server\", SERVER_NODE=$node, SERVER_NODE_PORT=$node_port, HTTP_URL=$http_url, HTTP_VERSION=$http_version, HTTP_STATUS=$http_status, HTTP_METHOD=$http_method, HTTP_CONTENT_TYPE=$http_content_type, HTTP_USER_AGENT=\"$http_user_agent\", HTTP_REFERRER=\"$http_referrer\", COOKIE=\"$cookie\", REQUEST_START_TIME=$req_start_time,REQUEST_ELAPSED_TIME=$req_elapsed_time, BYTES_IN=$req_length, BYTES_OUT=$res_length\r\n"
}
when LB_FAILED {
    set hsl [HSL::open -proto UDP -pool /Common/splunk_pool]
    HSL::send $hsl "<190> HSL, CLIENT_IP=$client_address, VIP=$vip, VIP_NAME=\"$virtual_server\", SERVER_NODE=$node, SERVER_NODE_PORT=$node_port, HTTP_URL=$http_url, HTTP_VERSION=$http_version, HTTP_STATUS=$http_status, HTTP_METHOD=$http_method, HTTP_CONTENT_TYPE=$http_content_type, HTTP_USER_AGENT=\"$http_user_agent\", HTTP_REFERRER=\"$http_referrer\", COOKIE=\"$cookie\", REQUEST_START_TIME=$req_start_time,REQUEST_ELAPSED_TIME=$req_elapsed_time, BYTES_IN=$req_length, BYTES_OUT=$res_length\r\n"
}
```

3. Browse the VIP where you have applied the iRule and then go to Splunk and search for `HOST=f5serv1* HSL`. If nothing shows up in Splunk, uncomment `#log local0.info` from the iRule to start writing logs in local SYSLOG (/var/logs/ltm). If logs are writing in local file but not showing up in Splunk, it means there is some network issue. If logs are not writing in local SYSLOG, try using a simpler iRule.

