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
1. Install Apache, NTP and Shibboleth;
```
yum install httpd ntp shibboleth.x86_64
```
2. Activate `shibd` at startup and start the service;
```
sudo systemctl enable httpd.service
sudo systemctl enable shibd.service
sudo systemctl enable ntpd.service
```
3. Start Apache and Shibboleth SP;
```
sudo service httpd start
sudo service shibd start
```
4. Check if the services are running;
```
ps aux | grep httpd
root     18366  0.0  0.4 328408 16532 ?        Ss   15:13   0:00 /usr/sbin/httpd
```
```
ps aux | grep shibd
shibd     8104  0.0  0.6 766416 25724 ?        Ssl  Dec07   0:14 /usr/sbin/shibd -p /var/run/shibboleth/shibd.pid -f -w 30
```
5. Check if `mod_shib.so` module is loaded in Apache;
```
httpd -M | grep mod_shib
mod_shib (shared)
Syntax OK
```
If the module is missing, check if `/etc/httpd/conf.d/shib.conf` has the following line;
```
LoadModule mod_shib /usr/lib64/shibboleth/mod_shib_22.so
```
If not, you can add the line either in `/etc/httpd/conf/httpd.conf` or in `/etc/httpd/conf.d/shib.conf` file but not in both place (Apache does not allow multiple module entry).

6. Check if there is any error;
```
grep -E 'CRIT|ERROR' /var/log/shibboleth/shibd.log
```

7. Check if SELinux is enabled;
```
sestatus
SELinux status:                 enabled
SELinuxfs mount:                /selinux
Current mode:                   permissive
Mode from config file:          enforcing
Policy version:                 24
Policy from config file:        targeted
```
If it is enabled then it is suggested to run in permissive mode. Follow the instruction at https://wiki.shibboleth.net/confluence/display/SP3/CommonErrors (Can't connect to listener process) where it explains how to create a policy so that SELinux allows httpd to access shibd pid.

8. Visit the following page;
```
https://example.com/Shibboleth.sso/Session
```
If it returns `A valid session was not found.`, it means `shibd` is running and working with Apache.

---

### B. Configuration
1. Configure `/etc/shibboleth/shibboleth2.xml` as following;
```
    <SPConfig xmlns="urn:mace:shibboleth:3.0:native:sp:config"
        xmlns:conf="urn:mace:shibboleth:3.0:native:sp:config"
        clockSkew="180">
        <OutOfProcess tranLogFormat="%u|%s|%IDP|%i|%ac|%t|%attr|%n|%b|%E|%S|%SS|%L|%UA|%a" />
        <ApplicationDefaults entityID="https://example.com/sp"
            homeURL="https://example.com/Shibboleth.sso/Session"
            REMOTE_USER="eppn subject-id pairwise-id persistent-id"
            cipherSuites="DEFAULT:!EXP:!LOW:!aNULL:!eNULL:!DES:!IDEA:!SEED:!RC4:!3DES:!kRSA:!SSLv2:!SSLv3:!TLSv1:!TLSv1.1">
            <Sessions lifetime="7200" timeout="3600" relayState="ss:mem"
                      checkAddress="false" handlerSSL="true" cookieProps="https">
                <SSO entityID="https://example.com/idp"
                     discoveryProtocol="SAMLDS" discoveryURL="https://ds.example.org/DS/WAYF">
                  SAML2
                </SSO>
                <Logout>SAML2 Local</Logout>
                <LogoutInitiator type="Admin" Location="/Logout/Admin" acl="127.0.0.1 ::1 10.10.10.111 10.10.10.222" />
                <Handler type="MetadataGenerator" Location="/Metadata" signing="false"/>
                <Handler type="Status" Location="/Status" acl="127.0.0.1 ::1 10.10.10.111 10.10.10.222"/>
                <Handler type="Session" Location="/Session" showAttributeValues="false"/>
                <Handler type="DiscoveryFeed" Location="/DiscoFeed"/>
            </Sessions>
            <Errors supportContact="root@localhost"
                helpLocation="/about.html"
                styleSheet="/shibboleth-sp/main.css"/>
            <MetadataProvider type="XML" path="/etc/shibboleth/idp-metadata/idp-metadata.xml"/>
            <AttributeExtractor type="XML" validate="true" reloadChanges="false" path="attribute-map.xml"/>
            <AttributeFilter type="XML" validate="true" path="attribute-policy.xml"/>
            <CredentialResolver type="File" use="signing"
                key="/etc/shibboleth/certs/sp-key.pem" certificate="/etc/shibboleth/certs/sp-cert.pem"/>
        </ApplicationDefaults>
        <SecurityPolicyProvider type="XML" validate="true" path="security-policy.xml"/>
        <ProtocolProvider type="XML" validate="true" reloadChanges="false" path="protocols.xml"/>
    </SPConfig>
```
2. Create a self-signed cert and save it in `/etc/shibboleth/certs` folder;
```
openssl req -x509 -sha256 -nodes -days 3650 -newkey rsa:2048 -subj "/CN=example.com" -keyout /etc/shibboleth/certs/sp-key.pem -out /etc/shibboleth/certs/sp-cert.pem
```
Verify the content of the cert;
```
openssl rsa -in /etc/shibboleth/certs/sp-key.pem -text
openssl x509 -noout -in /etc/shibboleth/certs/sp-cert.pem -text
```
Verify cert fingerprint;
```
openssl x509 -noout -in /etc/shibboleth/sp-cert.pem -fingerprint -sha1
```
3. Generate metadata for SP;
```
/etc/shibboleth/metagen.sh -c certs/sp-cert.pem -h example.com -e https://example.com/sp > sp-metadata.xml
```
4. Obtain IdP metadata and copy it to `/etc/shibboleth/idp-metadata/` folder.

5. Modify `/etc/httpd/conf.d/shibd.conf` to the following so that if a user visit https://example.com/resources, they will be send to IdP to login before accessing the location.
```
    <Location /resources>
      AuthType shibboleth
      ShibRequestSetting requireSession 1
      ShibCompatWith24 On
      require shib-session
    </Location>
```
6. Restart shibd to load IdP metadata and certificate;
```
sudo service shibd restart
```
7. Check if metadata loaded properly;
```
grep idp-metadata.xml /var/log/shibboleth/shibd.log
loaded XML resource (/idp-metadata/idp-metadata.xml)
```

---

### C. Test
1. Go to the following page and it will display `Status` as `OK`;
```
https://shibdev.its.fsu.edu/Shibboleth.sso/Status
```
2. Go to the following page;
```
https://example.com/resources
```
Your browser will be connected to Apache on port 443 and since the AuthType for /resources in shib.conf is “shibboleth”, you will be redirected to https://example.com/idp. After logging in, you will be redirected to /resources location and Apache will return 404 if there is no sample index.html file in this directory.

3. To test if SP is getting the attributes from IdP, go to;
```
https://example.com/Shibboleth.sso/Login
```
After successful login, you will get the following;
```
Miscellaneous
Session Expiration (barring inactivity): 120 minute(s)
Client Address: 146.201.20.247
SSO Protocol: urn:oasis:names:tc:SAML:2.0:protocol
Identity Provider: https://idp.fsu.edu
Authentication Time: 2018-12-10T21:36:41.536Z
Authentication Context Class: urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
Authentication Context Decl: (none)
Attributes	
affiliation: 1 value(s)
displayName: 1 value(s)
eppn: 1 value(s)
givenName: 1 value(s)
mail: 1 value(s)
sn: 1 value(s)
unscoped-affiliation: 2 value(s)
```


It is strongly suggested to create separate VHOST and httpd-ssl.conf file and also tell httpd.conf to load mod_shib module because shib.conf will will be overwritten everytime Shibboleth is updated.


