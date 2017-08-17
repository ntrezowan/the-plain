---
layout: post
title: "Setup SIP for Nokia E63"
comments: false
description: "Setup SIP for Nokia E63"
keywords: "sip, setup, configure, nokia, e63, voip"
---
This tutorial is to configure your Nokia E63 phone so that you can use it as a VOIP phone in your corporate office's IP telephony system.
> PBX: _Asterisk_  
> Hardware: _Nokia E63_    

___

1. From your phone, go to -  
    - `Menu > Tools > Settings > Connection > SIP Settings`.  
    - Choose `Options` and select `Add New`.  
    - Select `Use Default Profile`.  

```
[Sample configuration]
Change just the following settings in the profile while leaving all others as default;
Profile Name: VOIP_company_name
Public Username: xxx@4.5.6.7 (xxx - IP phone extension; 4.5.6.7 - Asterisk server IP)
Proxy Server: 8.9.10.11 (if any, then use the Proxy server IP; otherwise leave it blank)
Realm: password
Username: your_name
Password: your_password
Allow loose routing: Yes
Transport Type: UDP
Port: 5060
Register Server: 4.5.6.7 (Asterisk server IP)
Proxy server address: 8.9.10.11 (if any, then use the Proxy server IP; otherwise leave it blank)
```

2. Again from your phone, go to -  
`Menu > Tools > Settings > Connection > Internet Tel. Settings` and choose the newly created SIP configuration (e.g. *VOIP_company_name*).

3. After selecting the name, now choose the `SIP Profiles` and press `OK`.

4. To check your SIP status, go to -  
`Menu > Connect > Internet Tel. Registration Status`.  

You should see the name in the list and it will show as `Not Registered`.  
Press `OK` and soon your phone will registered with the Asterisk server.
