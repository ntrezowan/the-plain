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
    a) Check Java version;   
    Automation Engine supports OpenJDK Java 11, Oracle Java 1.8 and Oracle Java 11. Check if you have proper version of Java;

        # java -version

    b) Check environment variables;  

        # vi ~/.bashrc

        # AUTOMIC System Settings
        ORACLE_HOME=/opt/oracle/product/12.2.0.1/dbhome_1; export ORACLE_HOME
        AUTOMIC=/opt/ae/utility; export AUTOMIC
        PATH=.:$ORACLE_HOME/bin[:$PATH]; export PATH
        LD_LIBRARY_PATH=.:${AUTOMIC}/bin:$ORACLE_HOME/lib:/usr/lib:/lib[:$LD_LIBRARY_PATH]; export LD_LIBRARY_PATH
        TNS_ADMIN=/opt/oracle/product/12.2.0.1/dbhome_1/network/admin; export TNS_ADMIN
        CLASSPATH=${ORACLE_HOME}/jdbc/lib/:${ORACLE_HOME}/jlib/; export CLASSPATH


### B. Upgrade Automation Engine

1. Upgrade Automic Utility;  
    a) Upgrade Automic Utility bin folder;  

        # cp -r /opt/ae/utility/bin_new/bin/* /opt/ae/utility/bin/

    Check if all the files are owned by autotask user

    b) Check the config files;  
    All config file (*.INI) will not be replaced when we upgraded Automic Utility bin folder. These INI files have OCI DB connection string defined and you should verify the following files;

        AE.DB Archive: ucybdbar.ini
        AE.DB Client Copy: ucybdbcc.ini
        AE.DB Load: ucybdbld.ini
        AE.DB Reorg: ucybdbre.ini
        AE.DB Reporting Tool: ucybdbrt.ini
        AE.DB Revision Report: ucybdbrr.ini
        AE.DB Unload: ucybdbun.ini

    The UID should be in all capital as AUTOTASK.

    c) Check ucybdbar.sh execute permission;

        # ll ucybdbar.sh
        -rwxr-xr-x 1 autotask autotask 27 Sep  5 12:55 ucybdbar.sh

    If it does not have execute permission, set the permission;

        # chmod +x ucybdbar.sh


2. Upgrade AE DB scheme and load initial data

    a. Upgrade db folder;

        # cp -r /apps/automic/utility/db_new/* /apps/automic/utility/db/

    Check if all the files are owned by autotask user and fsubatch group

    b. Start AE DB Load;

        # cd /apps/automic/utility/bin
        # ./ucybdbld -B -X/apps/automic/utility/db/general/12.3/UC_UPD.TXT

        ========================================================
        Starting C++ batch mode loader...
        User account (user/domain) <AUTOTASK/>
        Startup parameter <-B -X/apps/automic/utility/db/general/12.3/UC_UPD.TXT >
        Application name <./ucybdbld>
        Launch from </apps/automic/utility/bin/>
        LoadLibrary pointer = <0x1b882d0>
        20190905/134647.081 - U00033125 Operating System: <UC4_ON_UNIX>
        20190905/134647.081 - U00033125 Operating System: <UC4_ON_UNIX_LINUX>
        20190905/134647.081 - U00038002 DLL 'ucybdbld', version '12.3.0+build.1560517368775'. Start parameter: '-B -X/apps/automic/utility/db/general/12.3/UC_UPD.TXT ', (changelist '1560507727').
        20190905/134647.081 - U00038259 Build Date: '2019-06-14', '15:07:38'
        20190905/134647.092 - ----------------------------------------------------------------------------------------------------
        20190905/134647.092 - [GLOBAL]
        20190905/134647.092 - language  = (E,D)
        20190905/134647.092 - logging   = ../temp/ucybdbld_log_##.txt
        20190905/134647.092 - logcount  = 10
        20190905/134647.092 - helplib   = uc.msl
        20190905/134647.092 - helpcache = ALL
        20190905/134647.092 - docu_path = ../../Docu/uc4/webhelp
        20190905/134647.092 - input     = ../db/
        …
        …
        20190911/131253.733 - U00003532 UCUDB: Checking data source ...
        20190911/131257.706 - U00003533 UCUDB: Check of data source finished: No errors. Performance CPU/DB: '1842013'/'369 (1000/2.706869 s)'
        20190911/131257.706 - U00003544 UCUDB: Reference values tested with Windows 2003 on XEON 1500 MHz: CPU 813865, DB 470
        20190911/131257.706 - U00003524 UCUDB: ===> Time critical DB call!       OPC: 'OPEN' time: '4:047.702.000'
        20190911/131257.706 - U00003525 UCUDB: ===> ''
        20190911/131257.707 - U00038091 Change application directory to '../db/'.
        20190911/131257.707 - U00038042 DB-Version = 12.3/R
        20190911/131257.731 - U00038084 Processing completed. Caution: If you use two or more users, you have to renew the roles of the AE user (privileges) and the synonyms.
        20190911/131257.736 - U00003523 UCUDB: Maximum time required for a DB call: '4:047.702.000'.
        20190911/131257.736 - U00003522 UCUDB: Database closed. Total time for DB calls: '4:075.260.999' seconds.
        20190911/131257.736 - U00003549 UCUDB: '            2026' 'OTHERS    ' calls took '0:002.766.000' sec.
        20190911/131257.736 - U00003549 UCUDB: '               3' 'SELECT    ' calls took '0:008.211.000' sec.
        20190911/131257.736 - U00003549 UCUDB: '               5' 'EXECUTE   ' calls took '4:059.009.999' sec.
        20190911/131257.736 - U00003549 UCUDB: '               0' 'UPDATE    ' calls took '0:000.000.000' sec.
        20190911/131257.736 - U00003549 UCUDB: '               0' 'DELETE    ' calls took '0:000.000.000' sec.
        20190911/131257.736 - U00003549 UCUDB: '               0' 'INSERT    ' calls took '0:000.000.000' sec.
        20190911/131257.736 - U00003549 UCUDB: '               3' 'READ      ' calls took '0:000.047.999' sec.
        20190911/131257.736 - U00003549 UCUDB: '            3025' 'CLOSESTMT ' calls took '0:000.052.000' sec.
        20190911/131257.736 - U00003549 UCUDB: '            1014' 'TRANSACT  ' calls took '0:005.173.999' sec.
        Application return code = 0

    c. Load RA Jar;

        # java -jar ./ucybdbld.jar -B -X/apps/iso/Autotask/Rapid.Automation/RA.FTP/FtpAgent_solution.jar

        20191106/120923.533 - U00038162 Loading file: '/apps/iso/Autotask/Rapid.Automation/RA.FTP/FtpAgent/FtpAgent_deploy_file.jar'.
        20191106/120923.661 - U00038163 File '/apps/iso/Autotask/Rapid.Automation/RA.FTP/FtpAgent/FtpAgent_deploy_file.jar' successfully loaded.
        20191106/120923.701 - U00038165 RA Solution loaded successfully.
        20191106/120923.701 - U00038272 Loading RA Plugins.
        20191106/120923.702 - U00038269 Version check result:
        Version of the RA Plugins: '4.0.7'.
        Version of database objects: '12.3.0+hf.1.build.1565956991091'.

        20191106/120923.711 - U00038162 Loading file: '/apps/iso/Autotask/Rapid.Automation/RA.FTP/FtpAgent/ecc-ae-sheet-ra-ftp.jar'.
        20191106/120923.724 - U00038163 File '/apps/iso/Autotask/Rapid.Automation/RA.FTP/FtpAgent/ecc-ae-sheet-ra-ftp.jar' successfully loaded.
        20191106/120923.737 - U00038273 RA Plugins loaded successfully.
        20191106/120923.754 - U00003523 UCUDB: Maximum time required for a DB call: '19:700.547.000'.
        20191106/120923.754 - U00003522 UCUDB: Database closed. Total time for DB calls: '20:794.676.000' seconds.
        20191106/120923.754 - U00003549 UCUDB: '            2306' 'OTHERS    ' calls took '0:007.253.000' sec.
        20191106/120923.754 - U00003549 UCUDB: '              31' 'SELECT    ' calls took '0:110.639.000' sec.
        20191106/120923.754 - U00003549 UCUDB: '             290' 'EXECUTE   ' calls took '20:239.570.000' sec.
        20191106/120923.754 - U00003549 UCUDB: '               5' 'UPDATE    ' calls took '0:010.429.999' sec.
        20191106/120923.754 - U00003549 UCUDB: '               0' 'DELETE    ' calls took '0:000.000.000' sec.
        20191106/120923.754 - U00003549 UCUDB: '             270' 'INSERT    ' calls took '0:377.413.000' sec.
        20191106/120923.754 - U00003549 UCUDB: '              32' 'READ      ' calls took '0:001.380.000' sec.
        20191106/120923.754 - U00003549 UCUDB: '            3867' 'CLOSESTMT ' calls took '0:001.500.999' sec.
        20191106/120923.754 - U00003549 UCUDB: '            1018' 'TRANSACT  ' calls took '0:046.489.999' sec.




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
