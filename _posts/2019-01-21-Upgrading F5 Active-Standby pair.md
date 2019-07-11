---
title: "Upgrading F5 Active-Standby pair"
comments: false
description: "Configure F5 for Splunk"
keywords: "F5, hsl, hish speed logging, request logging, management port logging, asm logging, apm logging, configure"
published: true
---
### Environment
F5 Active Server = f51.example.com 
F5 Standby Server = f52.example.com


---
### A. Preparation before upgrade (applied to all HA units)
1. Check current software version of F5;
```
# tmsh
# show /sys software status

--------------------------------------------------
Sys::Software Status
Volume  Product   Version  Build  Active    Status
--------------------------------------------------
HD1.1    BIG-IP  12.1.3.4  0.0.2      no  complete
HD1.2    BIG-IP  13.1.0.7  0.0.1      no  complete
HD1.3    BIG-IP  13.1.1.3  0.0.1      yes complete
```
From the output, look for the partition which is Active and then you will find the Version. Here the version is `13.1.1.3`.
2. Verify license and renew if necessary;

BIG-IP license is stored at `/config/bigip.license` and it has two dates; `Licensed date` and `Service check date`.  
`Licensed date` will show the date when you used your `Registration Key` for the first time to license your BIG-IP system. To find `Licensed date`, run the following;
```
# grep "Licensed date" /config/bigip.license
Licensed date :                    20180601
```
`Service Check Date` is the date when you last reactivated your license and it gets updated every time you reactivate your license (assuming that there is an active service contract with F5 for this BIG-IP system). For example, if you have reactivate your license on June 30, 2018 then it will show as 20180630. To find `Service Check Date`, run the following;
```
# grep "Service check date" /config/bigip.license
Service check date :               20171013
```
There is another interesting date called `License Check Date` and this date is related with the software version of BIG-IP. For example, Version `12.1.0-12.1.3` has a `License Check Date` 2016-03-2018. The `License Check Date` enforcement is applied during system startup. The system compares the `License Check Date` with the `Service Check Date` exists in the license file. If the `Service Check Date` is earlier than the `License Check Date`, the system will initialize but will not load the configuration. To allow the configuration to load properly, you must update the `Service Check Date` in the `bigip.license` file by reactivating the system's license. To find the `License Check Date` for the version you planned to upgrade, visit https://support.f5.com/csp/article/K7727. For example, if you plan to upgrade to version `13.1.0-13.1.1`, then `License Check Date` is 20170912. 

Now by comparing `Service check date` with `License Check Date`, we see that 20171013 > 20170912 which means you do not need to reactive the license before upgrade. But it is always a good practice to reactivate your license everytime you upgrade the F5 because it extends your `Service check date` in the license file.

If `Service Check Date` < `License Check Date`, do the following to reactivate the license before upgrade;

  a) Log in to the Configuration utility
  b) Navigate to System > License > Reactivate
  c) Select either Automatic or Manual (if F5 cannot reach internet)
  d) Click Next and it will be reactivated

3. Check device certificate and renew if necessary;

To check the device certificate, do the following;
  a) Log in to the Configuration utility
  b) Navigate to System > Certificate Management > Device Certificate Management > Device Certificate

If you need to renew device certificate, do the following;
  a) Log in to the Configuration utility
  b) Navigate to System > Certificate Management > Device Certificate Management > Device Certificate
  c) Click Import and choose Certificate and Key as Import Type
  d) Choose both certificate and key
  e) Click Import

4. Do a ConfigSync to sync configuration on both units;
It is always a better to do a config sync before the upgrade and this way both units will have the latest configuration.

To do a config sync, do the following;
  a) Log in to the Configuration utility
  b) Navigate to Device Management > Overview
  c) For Device Groups, click the name of the device group (device-group-a-failover) you want to synchronize
  d) For Devices, click the name of the device from which you want to perform the synchronization action
  e) For Sync, click the appropriate synchronization action
  f) Click Sync

5. Generate a qkview and check for Upgrade Advisor in iHealth;
iHealth reports can be used to find if there is any issue if we upgrade F5 units from one version to another. To generate a qkview, do the following;

  a) Log in to the Configuration utility
  b) Navigate to System > Support
  c) Click New Support Snapshot
  d) For Health Utility, click Generate QKView
  e) Click Start
  f) To download the output file, click Download

After download the file from F5, upload it to https://ihealth.f5.com/ and then go to `Upgrade Advisor` and select the version to which you want to upgrade your units. Then check the recommended feedback.
For example, here is one advise that iHealth provided when we are upgrading from `12.1.3.4` to `13.1.1.3`;

TMOS vulnerability: Password changes for local users may not be preserved unless the configuration is explicitly saved (K37250780)

6. Create a backup of the config file;
It is always good to have a backup of the config file before upgrade. This way, we can quickly restore F5 to previous stable state if there is any issues during upgrade.

To create UCS file, do the following;
  a) Log in to the Configuration utility
  b) Navigate to System > Archives
  c) To initiate the process of creating a new UCS archive, click Create
  d) In the File Name box, type a name for the file
  e) To create the UCS archive file, click Finished
  f) When the system completes the backup process, examine the status page for any reported errors before proceeding to the next step.
  g) To return to the Archive List page, click OK.
  h) Copy the .ucs file to a secure file system (i.e. shared NFS).

7. Verify volume formatting scheme;

Run the following to check if Big-IP system is using volume formatting system or partition formatting system;
```
# lvscan
  ACTIVE            '/dev/vg-db-sda/dat.maint.1' [300.00 MiB] inherit
  ACTIVE            '/dev/vg-db-sda/dat.share.1' [20.00 GiB] inherit
  ACTIVE            '/dev/vg-db-sda/dat.log.1' [500.00 MiB] inherit
  ACTIVE            '/dev/vg-db-sda/dat.swapvol.1' [1000.00 MiB] inherit
  ACTIVE            '/dev/vg-db-sda/set.1.root' [440.00 MiB] inherit
  ACTIVE            '/dev/vg-db-sda/set.1._usr' [3.29 GiB] inherit
  ACTIVE            '/dev/vg-db-sda/set.1._config' [3.17 GiB] inherit
  ACTIVE            '/dev/vg-db-sda/set.1._var' [3.00 GiB] inherit
  ACTIVE            '/dev/vg-db-sda/set.2.root' [440.00 MiB] inherit
  ACTIVE            '/dev/vg-db-sda/set.2._usr' [4.01 GiB] inherit
  ACTIVE            '/dev/vg-db-sda/set.2._config' [3.17 GiB] inherit
  ACTIVE            '/dev/vg-db-sda/set.2._var' [3.00 GiB] inherit
  ACTIVE            '/dev/vg-db-sda/app.ASWADB.set.2.mysqldb' [12.00 GiB] inherit
  ACTIVE            '/dev/vg-db-sda/app.avr.dat.avrdata' [3.81 GiB] inherit
  ACTIVE            '/dev/vg-db-sda/app.afm.dat.afmdata' [3.81 GiB] inherit
  ACTIVE            '/dev/vg-db-sda/app.asm.dat.asmdata1' [4.05 GiB] inherit
  ACTIVE            '/dev/vg-db-sda/set.3.root' [440.00 MiB] inherit
  ACTIVE            '/dev/vg-db-sda/set.3._usr' [4.01 GiB] inherit
  ACTIVE            '/dev/vg-db-sda/set.3._config' [3.17 GiB] inherit
  ACTIVE            '/dev/vg-db-sda/set.3._var' [3.00 GiB] inherit
```
If it returns no volume scheme, then it means Big-IP is using partition formatting scheme.

8. Import the software/hotfix image:

  a) Log in to the Configuration utility with administrative privileges
  b) To upload the necessary ISO files, navigate to System > Software Management
  c) Click Import
  d) Click Browse to select the SIG file (BIGIP-13.xxx.iso.384.sig). This is a SHA384 signed digest
  e) Click Import again
  f) Click Browse to select the ISO file (BIGIP-13.x.x.x.x.xxxx.iso)
  g) Click Import
  h) Click Import again
  i) Click Browse to select the pem file (archive.pubkey.xxxxxxxxx.pem). Download 3072 bit one since the SIG is using 3072 bit RSA public key
  j) Click Import
  k) After uploading the image, it will be listed under software image list
  l) Verify that all the files are under /shared/images;
```
ls -ltr /shared/images
total 7048744
-rw-r--r--. 1 tomcat tomcat 1938057216 2018-06-15 10:30 BIGIP-13.1.0.7-0.0.1.iso
-rw-r--r--. 1 tomcat tomcat 2009556992 2018-06-15 13:01 BIGIP-12.1.3.4-0.0.2.iso
-rw-r--r--. 1 tomcat tomcat 2057873408 2019-01-15 11:18 BIGIP-13.1.1.3-0.0.1.iso
-rw-r--r--. 1 tomcat tomcat        384 2019-01-15 11:23 BIGIP-13.1.1.3-0.0.1.iso.384.sig
-rw-r--r--. 1 tomcat tomcat        625 2019-01-15 11:28 archive.pubkey.20160210.pem
```
m) Verify the SIG;
```
# openssl dgst -sha384 -verify /shared/images/archive.pubkey.20160210.pem -signature /shared/images/BIGIP-13.1.1.3-0.0.1.iso.384.sig /shared/images/BIGIP-13.1.1.3-0.0.1.iso

Verified OK
```

9. Check that root login to shell is possible;
In situation during the upgrade, F5 might not be able to access ADFS/LDAP and so your user/pass might not work. Check if you have a root account and if you can SSH to it by running the following;
```
ssh root@f5prd01.its.fsu.edu
ssh root@f5prd02.its.fsu.edu 
```
10. Check that admin login to GUI is possible;
In situation during the upgrade, F5 might not be able to access ADFS/LDAP and so your user/pass might not work. Check if you have a admin account and if you can login to GUI using the admin account;
https://f51.example.com
https://f52.example.com



---

### B. Add Splunk on F5
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

### C. Configure HSL to use TMM ports
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

---

### D. Configure HSL to use Management port
1. Verify that F5 is using management port to reach Splunk;
```
user@(f5serv1)(cfg-sync In Sync)(Active)(/Common)(tmos)# ip route get 10.10.10.1
10.10.10.1 via 10.1.1.1 prd mgmt  src 10.1.1.2
```
If not, configure F5 so that it uses management port to reach Splunk;
```
user@(f5serv1)(cfg-sync In Sync)(Active)(/Common)(tmos)# create /sys log-config destination management-port splunk ip-address 10.10.10.1 port 9514 protocol udp
user@(f5serv1)(cfg-sync In Sync)(Active)(/Common)(tmos)# list sys management-route
sys management-route splunk {
    gateway 10.1.1.1
    network 10.10.10.1/32
}
user@(f5serv1)(cfg-sync In Sync)(Active)(/Common)(tmos)# save sys config
```

2. Create a Log destination.  
Go to `System > Logs > Configuration > Log Destinations`. Create New and configure as following;
```
Name = splunk_hsl_via_mgmt_port
Type = Management Port
Address = 10.10.10.1
Port = 9515
Protocol = TCP
```

3. Create a Log publisher.  
Go to `System > Logs > Configuration> Log Publisher`. Create New and configure as following;
```
Name = splunk_hsl_publisher
Destination = splunk_hsl_via_mgmt_port
```
4. Create an iRule and add it to a VIP. 
Go to `Local Traffic > iRules > iRules List`. Click Create and here is a sample iRule which does Apache like HTTP Request/Response logging;
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
    set hsl [HSL::open -publisher /Common/splunk_hsl_publisher]
    HSL::send $hsl "<190> HSL, CLIENT_IP=$client_address, VIP=$vip, VIP_NAME=\"$virtual_server\", SERVER_NODE=$node, SERVER_NODE_PORT=$node_port, HTTP_URL=$http_url, HTTP_VERSION=$http_version, HTTP_STATUS=$http_status, HTTP_METHOD=$http_method, HTTP_CONTENT_TYPE=$http_content_type, HTTP_USER_AGENT=\"$http_user_agent\", HTTP_REFERRER=\"$http_referrer\", COOKIE=\"$cookie\", REQUEST_START_TIME=$req_start_time,REQUEST_ELAPSED_TIME=$req_elapsed_time, BYTES_IN=$req_length, BYTES_OUT=$res_length\r\n"
}
when LB_FAILED {
    set hsl [HSL::open -publisher /Common/splunk_hsl_publisher]
    HSL::send $hsl "<190> HSL, CLIENT_IP=$client_address, VIP=$vip, VIP_NAME=\"$virtual_server\", SERVER_NODE=$node, SERVER_NODE_PORT=$node_port, HTTP_URL=$http_url, HTTP_VERSION=$http_version, HTTP_STATUS=$http_status, HTTP_METHOD=$http_method, HTTP_CONTENT_TYPE=$http_content_type, HTTP_USER_AGENT=\"$http_user_agent\", HTTP_REFERRER=\"$http_referrer\", COOKIE=\"$cookie\", REQUEST_START_TIME=$req_start_time,REQUEST_ELAPSED_TIME=$req_elapsed_time, BYTES_IN=$req_length, BYTES_OUT=$res_length\r\n"
}
```
5. Browse the VIP where you have applied the iRule and then go to Splunk and search for `HOST=f5serv1* HSL`.

---

### E. Configure HSL for AFM
1. Create an unformatted HSL log destination. 
Go to `System > Logs > Configuration > Log Destinations`. Click on Create and configure as following;
```
Name = splunk_afm_unformatted
Type = Remote HSL
Pool = splunk_pool
Protocol = UDP
```

2. Create a formatted HSL log destination for Splunk. Go to `System > Logs > Configuration > Log Destinations`. Click on Create and configure as following;
```
Name = splunk_afm_formatted
Type = Remote Syslog
Syslog Format = BSD Syslog
Forward to = splunk_afm_unformatted
```

3. Create a HSL log publisher. Go to `System > Logs > Configuration > Log Publishers`. Click on Create and configure as following;
```
Name = splunk_afm_publisher
Destination = splunk_afm_formatted
```

4. Create a firewall logging profile. Go to `Security > Event Logs > Logging Profiles`. Click on Create and configure as following;
```
Name = splunk_afm_logging
Network Firewall = Tick
---
Publisher = splunk_afm_publisher
Aggregate Rate Limit = Indefinite
Log Rule Messages 
		  -> Accept = Indefinite
		  -> Drop = Indefinite
		  -> Reject = Indefinite
Log IP Errors = Enabled
Log TCP Errors = Enabled
Log TCP Event = Enabled
Log Transation Fields = Enabled
Always Log Region = Enabled
Storage Format = User-Defined
ACTION=${action}, SOURCE_IP=${src_ip}:${src_port}, REMOTE_IP=${dest_ip}:${dest_port}, REMOTE_LOCATION=${dest_geo}, PROTOCOL=${protocol}, TRANS_SOURCE_IP=${translated_src_ip}:${translated_src_port}, TRANS_REMOTE_IP=${translated_dest_ip}:${translated_dest_port}, DROP_REASON=${drop_reason}, VLAN=${vlan}
---
IP Intelligence 
Publisher = splunk_afm_publisher
Log Translation Fields = Enabled
---
Traffic Statistics
Publisher = splunk afm_publisher
Aggregate Rate Limit = Indefinite
Log Timer Events = Active Flows
---
Port Misuse
Publisher = splunk_afm_publisher
Aggregate Rate Limit = Indefinite
```

5. Add the logging profile to a VIP with a APM policy. Go to `Local Traffic > Virtual Servers`. Click on the Virtual Server where you want to apply logging and go to `Security > Policies` tab. Now configure as following;
```
Network Firewall 
		  -> Enforcement = Enabled
		  -> Staging = Disabled
Policy = {a_prefedined policy}
Log Profile = splunk_afm_logging
```

6. Go to Splunk and search for `host=f5serv1* "ACTION=Drop"`.

---

### F. Configure Request Logging
1. Go to `Local Traffic > Profiles > Other > Request Logging`. Click Create and configure as following;
```
Name = splunk_http_request_logging
Parent Profile = request-log
Request Logging = Enabled
Template = $DATE_NCSA REQUEST -> CLIENT = $CLIENT_IP:$CLIENT_PORT, VIP = $VIRTUAL_IP:$VIRTUAL_PORT, HTTP_VERSION = $HTTP_VERSION, HTTP_METHOD = $HTTP_METHOD, HTTP_KEEPALIVE = $HTTP_KEEPALIVE, HTTP_PATH = $HTTP_PATH, HTTP_QUERY = $HTTP_QUERY, HTTP_REQUEST = $HTTP_REQUEST, HTTP_URI = $HTTP_URI
HSL Protocol = UDP
Pool Name = splunk_pool
Response Logging = Enabled
Template = $DATE_NCSA, RESPONSE -> CLIENT = $CLIENT_IP:$CLIENT_PORT, VIP = $VIRTUAL_IP:$VIRTUAL_PORT, SERVER = $SERVER_IP:$SERVER_PORT, HTTP_VERSION = $HTTP_VERSION, HTTP_METHOD = $HTTP_METHOD, HTTP_KEEPALIVE = $HTTP_KEEPALIVE, HTTP_PATH = $HTTP_PATH, HTTP_QUERY = $HTTP_QUERY, HTTP_REQUEST = $HTTP_REQUEST, HTTP_STATUS = $HTTP_STATUS, HTTP_URI = $HTTP_URI, SNAT_IP = $SNAT_IP:$SNAT_PORT, F5_HOSTNAME = $BIGIP_HOSTNAME, RESPONSE_TIME = $RESPONSE_MSECS, RESPONSE_SIZE = $RESPONSE_SIZE
HSL Protocol = UDP
Pool Name = splunk_pool
```

2. Go to `Local Traffic > Virtual Servers`. Click on the VIP which you want to use Request Logging. Select Advanced of Configuration and then choose `Request Logging Profile` as `splunk_http_request_logging`

3. Browse the VIP where you have applied the iRule and then go to Splunk and search for `HOST=f5serv1* REQUEST`

