---
title: "Upgrading Automic Automation Engine to 12.3"
comments: false
description: "Upgrading Automic Automation Engine to 12.3"
keywords: "upgrade, automic, automation, engine, workload, 12.3. ca, broadcom"
published: true
---

### A. Preparation before upgrade 
1. Backup Automic Utility and extract new JAR;
```
# rm -rf /opt/ae/utility/bin_new/*
# rm -rf /opt/ae/utility/bin_orig/*
# cp -r /opt/ae/utility/bin/* /opt/ae/utility/bin_orig/
# cp /opt/iso/Automic.Automation_12.3.0_HF1/Automation.Platform/Utility/unix/linux/x64/utillx6.tar.gz /opt/ae/utility/bin_new
# tar -zxvf utillx6.tar.gz
```
In here, we are only interested of bin folder and db folder will be empty.

2. Backup Automic DB and extract new JAR;
```
# rm -rf /opt/ae/utility/db_new/*
# rm -rf /opt/ae/utility/db_orig/*
# cp -r /opt/ae/utility/db/* /opt/ae/utility/db_orig/
# cp /opt/iso/Automic.Automation_12.3.0_HF1/Automation.Platform/db/db.tar.gz /opt/ae/utility/db_new
# tar -zxvf db.tar.gz
```

3. Backup Automic Engine and extract new JAR;
```
# rm -rf /opt/ae/automationengine/bin_new/*
# rm -rf /opt/ae/automationengine/bin_orig/*
# cp -r /opt/ae/automationengine/bin/* /opt/ae/automationengine/bin_orig/
# cp /opt/iso/Automic.Automation_12.3.0_HF1/Automation.Platform/AutomationEngine/unix/linux/x64/ucslx6.tar.gz /opt/ae/automationengine/bin_new/
# tar -zxvf ucslx6.tar.gz
```

4. Backup Service Manager and extract new JAR;
```
# rm -rf /opt/ae/servicemanager/bin_new/*
# rm -rf /opt/ae/servicemanager/bin_orig/*
# cp -r /opt/ae/servicemanager/bin/* /opt/ae/servicemanager/bin_orig/
# cp /opt/iso/Automic.Automation_12.3.0_HF1/Automation.Platform/ServiceManager/unix/linux/x64/ucsmgrlx6.tar.gz /opt/ae/servicemanager/bin_new/
# tar -zxvf ucsmgrlx6.tar.gz
```

5. Create folder for CAPKI and copy the installer;

Download CAPKI installer from https://downloads.automic.com/downloads.
```
# mkdir /opt/ae/capki/
# cp /opt/iso/CA.PKI/unix/linux/x64/setup /opt/ae/capki/
```

6. Shutdown the system ;
```
# ps -ef | grep ucybsmgr
# kill pid
```

7. Check environment;
    a. Check Java version;
    Automation Engine supports OpenJDK Java 11 , Oracle Java 1.8 and Oracle Java 11. Check if you have proper version of Java;
    ```
    # java -version
    ```
    b. Check environment variables;
    ```
    # vi ~/.bashrc

    # AUTOMIC System Settings
    ORACLE_HOME=/opt/oracle/product/12.2.0.1/dbhome_1; export ORACLE_HOME
    AUTOMIC=/opt/ae/utility; export AUTOMIC
    PATH=.:$ORACLE_HOME/bin[:$PATH]; export PATH
    LD_LIBRARY_PATH=.:${AUTOMIC}/bin:$ORACLE_HOME/lib:/usr/lib:/lib[:$LD_LIBRARY_PATH]; export LD_LIBRARY_PATH
    TNS_ADMIN=/opt/oracle/product/12.2.0.1/dbhome_1/network/admin; export TNS_ADMIN
    CLASSPATH=${ORACLE_HOME}/jdbc/lib/:${ORACLE_HOME}/jlib/; export CLASSPATH
    ```




There is another interesting date called `License Check Date` and this date is related with the software version of BIG-IP. For example, Version `12.1.0-12.1.3` has a `License Check Date` 2016-03-2018. The `License Check Date` enforcement is applied during system startup. The system compares the `License Check Date` with the `Service Check Date` exists in the license file. If the `Service Check Date` is earlier than the `License Check Date`, the system will initialize but will not load the configuration. To allow the configuration to load properly, you must update the `Service Check Date` in the `bigip.license` file by reactivating the system's license. To find the `License Check Date` for the version you planned to upgrade, visit https://support.f5.com/csp/article/K7727. For example, if you plan to upgrade to version `13.1.0-13.1.1`, then `License Check Date` is 20170912.<br /><br />
Now by comparing `Service check date` with `License Check Date`, we see that 20171013 > 20170912 which means you do not need to reactive the license before upgrade. But it is always a good practice to reactivate your license everytime you upgrade the F5 because it extends your `Service check date` in the license file.<br /><br />
If `Service Check Date` < `License Check Date`, do the following to reactivate the license before upgrade;  
    a) Log in to the Configuration utility  
    b) Navigate to System > License > Reactivate  
    c) Select either Automatic (if F5 can reach internet) or Manual (if F5 cannot reach internet)  
    d) Click Next and it will be reactivated  

3. Check device certificate and renew if necessary;  
To check the device certificate, do the following;  
    a) Log in to the Configuration utility  
    b) Navigate to System > Certificate Management > Device Certificate Management > Device Certificate<br /><br />
If you need to renew device certificate, do the following;  
    a) Log in to the Configuration utility  
    b) Navigate to System > Certificate Management > Device Certificate Management > Device Certificate  
    c) Click Import and choose Certificate and Key as Import Type  
    d) Choose both certificate and key  
    e) Click Import  
    
4. Do a ConfigSync to sync configuration on both units;  
It is always a better to do a config sync before the upgrade and this way both units will have the latest configuration.<br /><br />
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
    f) To download the output file, click Download<br /><br />
After download the file from F5, upload it to https://ihealth.f5.com/ and then go to `Upgrade Advisor` and select the version to which you want to upgrade your units. Then che
