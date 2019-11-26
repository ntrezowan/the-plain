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

3. Backup Automation Engine and extract new JAR;
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
Download CAPKI installer from [https://downloads.automic.com/downloads](https://downloads.automic.com/downloads).
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
        openjdk version "11.0.4" 2019-07-16 LTS
        OpenJDK Runtime Environment 18.9 (build 11.0.4+11-LTS)
        OpenJDK 64-Bit Server VM 18.9 (build 11.0.4+11-LTS, mixed mode, sharing)

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

        # cp -r /opt/ae/utility/db_new/* /opt/ae/utility/db/

    Check if all the files are owned by autotask user and autotask group

    b. Start AE DB Load;

        # cd /opt/ae/utility/bin
        # ./ucybdbld -B -X/opt/ae/utility/db/general/12.3/UC_UPD.TXT

        ========================================================
        Starting C++ batch mode loader...
        User account (user/domain) <AUTOTASK/>
        Startup parameter <-B -X/opt/ae/utility/db/general/12.3/UC_UPD.TXT >
        Application name <./ucybdbld>
        Launch from </opt/ae/utility/bin/>
        LoadLibrary pointer = <0x1b882d0>
        20190905/134647.081 - U00033125 Operating System: <UC4_ON_UNIX>
        20190905/134647.081 - U00033125 Operating System: <UC4_ON_UNIX_LINUX>
        20190905/134647.081 - U00038002 DLL 'ucybdbld', version '12.3.0+build.1560517368775'. Start parameter: '-B -X/opt/ae/utility/db/general/12.3/UC_UPD.TXT ', (changelist '1560507727').
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

        # java -jar ./ucybdbld.jar -B -X/opt/iso/Automic.Automation_12.3.0_HF1/Rapid.Automation/RA.FTP/FtpAgent_solution.jar

        20191106/120923.533 - U00038162 Loading file: '/opt/iso/Automic.Automation_12.3.0_HF1/Rapid.Automation/RA.FTP/FtpAgent_solution.jar'.
        20191106/120923.661 - U00038163 File '/opt/iso/Automic.Automation_12.3.0_HF1/Rapid.Automation/RA.FTP/FtpAgent_solution.jar' successfully loaded.
        20191106/120923.701 - U00038165 RA Solution loaded successfully.
        20191106/120923.701 - U00038272 Loading RA Plugins.
        20191106/120923.702 - U00038269 Version check result:
        Version of the RA Plugins: '4.0.7'.
        Version of database objects: '12.3.0+hf.1.build.1565956991091'.




3. Upgrade Automation Engine

    a) Upgrade Automation Engine;
    
        # cp -r /apps/automic/automationengine/bin_new/bin/* /apps/automic/automationengine/bin/
        
    Check if all the files are owned by autotask user and autotask group

    b) Check CP libraries;
    
        # cd /apps/automic/automationengine/bin/
        # ldd -r ucsrvcp > wk.txt
        # cat wk.txt
        linux-vdso.so.1 =>  (0x00007ffd88724000)
        libzu00132.so => ./libzu00132.so (0x00007fe425490000)
        libucmsgq.so => ./libucmsgq.so (0x00007fe425259000)
        libucudb32.so => ./libucudb32.so (0x00007fe425008000)
        libuccache.so => ./libuccache.so (0x00007fe424dfb000)
        libsysapi.so => ./libsysapi.so (0x00007fe424bf6000)
        libzusynchk.so => ./libzusynchk.so (0x00007fe4249e9000)
        libuc001.so => ./libuc001.so (0x00007fe4247e4000)
        libzugss.so => ./libzugss.so (0x00007fe4245cc000)
        libzukms.so => ./libzukms.so (0x00007fe4243c6000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x0000003167e00000)
        libdl.so.2 => /lib64/libdl.so.2 (0x0000003167a00000)
        libstdc++.so.6 => ./libstdc++.so.6 (0x00000031cfa00000)
        libm.so.6 => /lib64/libm.so.6 (0x0000003168600000)
        libgcc_s.so.1 => ./libgcc_s.so.1 (0x00000031cf600000)
        libc.so.6 => /lib64/libc.so.6 (0x0000003167600000)
        librt.so.1 => /lib64/librt.so.1 (0x0000003168200000)
        /lib64/ld-linux-x86-64.so.2 (0x0000556b5c1be000)

    c) Check WP libraries;
    
        # cd /apps/automic/automationengine/bin/
        # ldd -r ucsrvwp > wk.txt
        # cat wk.txt
        linux-vdso.so.1 =>  (0x00007ffccdf97000)
        libucsj.so => ./libucsj.so (0x00007f4b61277000)
        libucdsfun.so => ./libucdsfun.so (0x00007f4b60ec7000)
        libucucc.so => ./libucucc.so (0x00007f4b60cb5000)
        libucuhg.so => ./libucuhg.so (0x00007f4b60a8f000)
        libucrtl.so => ./libucrtl.so (0x00007f4b60886000)
        libzuxml.so => ./libzuxml.so (0x00007f4b60664000)
        libucldap.so => ./libucldap.so (0x00007f4b60443000)
        libucexith.so => ./libucexith.so (0x00007f4b6023f000)
        libucuhsta.so => ./libucuhsta.so (0x00007f4b60032000)
        libucsmtp.so => ./libucsmtp.so (0x00007f4b5fe28000)
        libucumqa.so => ./libucumqa.so (0x00007f4b5fbb6000)
        libucqert.so => ./libucqert.so (0x00007f4b5f99e000)
        libucumulti.so => ./libucumulti.so (0x00007f4b5f798000)
        libzu00132.so => ./libzu00132.so (0x00007f4b5f566000)
        libucmsgq.so => ./libucmsgq.so (0x00007f4b5f330000)
        libucudb32.so => ./libucudb32.so (0x00007f4b5f0df000)
        libuccache.so => ./libuccache.so (0x00007f4b5eed1000)
        libsysapi.so => ./libsysapi.so (0x00007f4b5eccd000)
        libzusynchk.so => ./libzusynchk.so (0x00007f4b5eac0000)
        libuc001.so => ./libuc001.so (0x00007f4b5e8ba000)
        libzugss.so => ./libzugss.so (0x00007f4b5e6a3000)
        libzukms.so => ./libzukms.so (0x00007f4b5e49d000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x0000003167e00000)
        libdl.so.2 => /lib64/libdl.so.2 (0x0000003167a00000)
        libstdc++.so.6 => ./libstdc++.so.6 (0x00000031cfa00000)
        libm.so.6 => /lib64/libm.so.6 (0x0000003168600000)
        libgcc_s.so.1 => ./libgcc_s.so.1 (0x00000031cf600000)
        libc.so.6 => /lib64/libc.so.6 (0x0000003167600000)
        librt.so.1 => /lib64/librt.so.1 (0x0000003168200000)
        /lib64/ld-linux-x86-64.so.2 (0x0000562aa7289000)

    d) Check DB libraries;
    
        # cd /apps/automic/automationengine/bin/
        # ldd -r ucuoci.so > wk.txt
        # cat wk.txt
        linux-vdso.so.1 =>  (0x00007ffe7b5f1000)
        libsysapi.so => ./libsysapi.so (0x00007f4c178e1000)
        libzu00132.so => ./libzu00132.so (0x00007f4c176af000)
        libuc001.so => ./libuc001.so (0x00007f4c174aa000)
        libclntsh.so.12.1 => /opt/app/oracle/product/12.2.0.1/dbhome_1/lib/libclntsh.so.12.1 (0x00007f4c139f9000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007f4c137e8000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f4c135cb000)
        libstdc++.so.6 => ./libstdc++.so.6 (0x00007f4c133db000)
        libm.so.6 => /lib64/libm.so.6 (0x00007f4c13156000)
        libgcc_s.so.1 => ./libgcc_s.so.1 (0x00007f4c13049000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f4c12cb5000)
        librt.so.1 => /lib64/librt.so.1 (0x00007f4c12aac000)
        libmql1.so => /opt/app/oracle/product/12.2.0.1/dbhome_1/lib/libmql1.so (0x00007f4c12835000)
        libipc1.so => /opt/app/oracle/product/12.2.0.1/dbhome_1/lib/libipc1.so (0x00007f4c123f9000)
        libnnz12.so => /opt/app/oracle/product/12.2.0.1/dbhome_1/lib/libnnz12.so (0x00007f4c11cb0000)
        libons.so => /opt/app/oracle/product/12.2.0.1/dbhome_1/lib/libons.so (0x00007f4c11a62000)
        libnsl.so.1 => /lib64/libnsl.so.1 (0x00007f4c11848000)
        libaio.so.1 => /lib64/libaio.so.1 (0x00007f4c11647000)
        libresolv.so.2 => /lib64/libresolv.so.2 (0x00007f4c1142d000)
        /lib64/ld-linux-x86-64.so.2 (0x000055b48bc43000)
        libclntshcore.so.12.1 => /opt/app/oracle/product/12.2.0.1/dbhome_1/lib/libclntshcore.so.12.1 (0x00007f4c10e5e000)

    e) Start a CP;
    
        # cd /apps/automic/automationengine/bin/
        # ./ucsrvcp &

        UC4 CP-Server Version 12.3.0+hf.1.build.1565007573762 (PID=57961)

    Check if CP has started;
    
        # ps -ef | grep ucsrvcp

    Check log;
    
        # cat /apps/automic/automationengine/temp/CPsrv_log_001_00.txt

    f) Start a WP;
    
        # cd /apps/automic/automationengine/bin/
        # ./ucsrvwp &

        UC4 WP-Server Version 12.3.0+hf.1.build.1565007573762 (PID=58630)

        [autotask@autotaskdev01 bin]$ SERVER=ATASKDEV#WP001
        *** PRIMARY(ATASKDEV#WP001) ***

    Check if WP has started;
    
        # ps -ef | grep ucsrvwp

    Check log;
    
        # cat /apps/automic/automationengine/temp/WPsrv_log_001_00.txt

    g) Stop CP/WP processes;

        # ps -ef | grep ucsrvcp
        # kill pid

        # ps -ef | grep ucsrvwp
        # kill pid






8. Upgrade Service Manager

    a) Upgrade Service Manager bin folder;

        # cp -r /apps/automic/servicemanager/bin_new/bin/* /apps/automic/servicemanager/bin/

    Check if all the files are owned by autotask user and autotask group

    b) Start ServiceManager;

        # cd /apps/automic/servicemanager/bin/
        # nohup ./ucybsmgr &

        # ps -ef | grep ucybsmgr

    Check logs

        # cat /apps/automic/servicemanager/temp/SMgr_log_00.txt



9.	Install JWP

    a) Remove the following content so that AE lib and plugins folder are empty;

        # rm /apps/automic/automationengine/bin/lib/*
        # rm /apps/automic/automationengine/bin/plugins/*

    b) Copy content of lib and plugins folders to AE;

        # cp /apps/automic/automationengine/bin_new/bin/lib/* /apps/automic/automationengine/bin/lib/

    It will copy org.eclipse.osgi.jar file only

        # cp /home/autotask/jwp/bin/plugins/* /apps/automic/automationengine/bin/plugins/

    c) Copy xdb6.jar and xmlparserv2.jar files to the lib folder to ensure that XML variables can be processed correctly;

        # cp /opt/app/oracle/product/12.2.0.1/dbhome_1/sqldeveloper/modules/oracle.xdk/xmlparserv2.jar /apps/automic/automationengine/bin/lib/

        # cp /opt/app/oracle/product/12.2.0.1/dbhome_1/sqldeveloper/rdbms/jlib/xdb6.jar /apps/automic/automationengine/bin/lib/

    d) Copy ojdbc8.jar from Oracle to lib folder;
    
        # cp /opt/app/oracle/product/12.2.0.1/dbhome_1/jdbc/lib/ojdbc8.jar /apps/automic/automationengine/bin/lib/

    e)	Install LDAP certificate;

    By default, JWP uses Java keystore cacerts. To import a LDAP cert, add both site cert and intermediate;

    •	Directly install the cert: 
    To directly install the certificate from LDAP F5 VIP;

        # telnet ldapqna.its.fsu.edu 636
        # cd /apps/java/jre/lib/security/
        # cp cacerts cacerts.backup
        # cd /apps/automic/automationengine/bin/
        # java -jar ucsrvjp.jar -installcert ldapqna.its.fsu.edu:636

        Loading KeyStore /apps/jdk1.8.0_181/jre/lib/security/cacerts...
        Opening connection to ldapqna.its.fsu.edu:636...
        Starting SSL handshake...

        No errors, certificate installed.

        # cd /apps/java/jre/lib/security/
        # keytool -list -v -keystore cacerts -alias "ldapqna" | more
        # keytool -list -v -keystore cacerts -alias "sectigo_intermediate" | more




    Create a new keystore if preferred: 
    You can also manually create a new file as following;
    
        # java -jar ucsrvjp.jar -installcert ldapqna.its.fsu.edu:636 /apps/automic/automationengine/bin/automic.jks

    f) Configure the Database;

    AUTOTASKSAN is working fine with OCI+JDBC driver, so no need to configure anything here, just verify that ucsrv.ini has OCI connection string;

        # vi /apps/automic/automationengine/bin/ucsrv.ini

        SQLDRIVERCONNECT=ODBCVAR=NNJNNORO,DSN=ATASKS;UID=AUTOTASK;PWD=--1066AB96C2CAB174EA07390CA11EB75A8D;SP=NLS_LANGUAGE=AMERICAN,NLS_TERRITORY=AMERICA,CODESET=WE8ISO8859P15

    In version 11.2.7, you do not need to use JDBC connection string, the OCI connection string will work;

    But for version 12.1 or higher, you have to use JDBC driver. Here is SAN JDBC connection string (not tested yet);
        [JDBC]
        SQLDRIVERCONNECT=jdbc:oracle:thin:@autotasksan01-utl.its.fsu.edu:1521/ATASKS

    Remember that UID must be all capital and this is a bug in Automic.

    g) Start JWP

        # cd /apps/automic/automationengine/bin
        # java -Xmx512M -jar ucsrvjp.jar

        UC4 ATASKDEV#WP-Server Version 12.3.0+hf.1.build.1565696063450 (PID=64826)

        # ps -ef | grep -i ucsrvjp

    Check logs;
        # cd /apps/automic/automationengine/temp/

    h) Add JWP to Service Manager;

    Add the following to uc4.smd;
        # vi /apps/automic/servicemanager/bin/uc4.smd

        ! JWP
        DEFINE UC4 JWP1;java -jar -Xrs -Xmx512M /apps/automic/automationengine/bin/ucsrvjp.jar -i/apps/automic/automationengine/bin/ucsrv.ini -svc%port%;/apps/automic/automationengine/bin


    Add the following to uc4.smc;
        # vi /apps/automic/servicemanager/uc4.smc

        WAIT 10
        CREATE UC4 WP5

    i) Restart Service Manager and check if JWP can be start/stop from ServiceManagerDialogue (THIS ONLY WORKS NOW AFTER DEFINING EVERYTHING)

        # cat CPsrv_log_002_00.txt | grep "R E A D Y   F O R   R U N



10.	Install JCP

    a) Add the following to ucsrv.ini;
        # vi /apps/automic/automationengine/bin/ucsrv.ini

        [REST]
        port=8088
        sslEnabled=0
        keystore=/apps/automic/automationengine/bin/ssl/keystore
        keystorePassword=changeit
        keyPassword=changeit
        parallelDbConnections=5

    or HTTPS

        [REST]
        port=8443
        sslEnabled=1
        keystore=/apps/ssl_certs/tomcat-jcp.jks
        keystorePassword=changeit
        keyPassword=changeit
        parallelDbConnections=5

    Copy the file tomcat-jcp.jks from autotaskdev01/autotasksan01 to autotaskprd01.

    b) Add JCP to Service Manager;

    Add the following to uc4.smd;
        # vi /apps/automic/servicemanager/bin/uc4.smd

        ! JCP
        DEFINE UC4 JCP1;java -jar -Xrs -Xmx512M /apps/automic/automationengine/bin/ucsrvjp.jar -i/apps/automic/automationengine/bin/ucsrv.ini -svc%port% -rest;/apps/automic/automationengine/bin

    Add the following to uc4.smc
        # vi /apps/automic/servicemanager/bin/uc4.smc

        WAIT 10
        CREATE UC4 JCP1

    c) If you want to use HTTPS, create a keystore with *.its.fsu.edu cert+key and rename the alias to jetty;

        # cd /apps/java/jre/lib/security/

        # openssl pkcs12 -inkey its_fsu_edu_wildcard_2021.key -in its_fsu_edu_wildcard_2021.crt -export -out jetty.pkcs12

        # keytool -importkeystore -srckeystore jetty.pkcs12 -srcstoretype PKCS12 -destkeystore keystore

        # keytool -changealias -alias 1 -destalias jetty -keystore keystore

        # keytool -v -list -keystore keystore

        # cp keystore /apps/automic/automationengine/bin/ssl/

    d) Restart Service Manager and check if JWP can be start/stop from ServiceManagerDialogue

    e) Check logs;
        # cd /apps/automic/automationengine/temp/

    JCP normally will be the last of the CP’s.



12.	Install CAPKI

    a) Install CAPKI

    Go to https://downloads.automic.com/downloads/component_downloads and search for capki. Download it and move it to /apps/automic/capki.

    Request UNIX team to install it as root;
        # cd /apps/automic/capki/
        # ./setup install caller=AE123 veryverbose env=all instdir=/apps/automic/capki/

    If return code is 0, it means CAPKI has installed successfully. 

    b) Check the environmental variable;

        # echo $CALIB
        /apps/automic/capki/lib

        # echo $CABIN
        /apps/automic/capki/bin

        # echo $CASHCOMP
        /apps/automic/capki

    If these returns NULL, logout and then login and then restart Service Manager in the server.

    c) Check certificate;  
    When CAPKI setup script finishes, it will create a self-signed certificate. The cert and key will be stored in the following location;

    Cert: /apps/automic/automationengine/bin/ucsrv_certificate.pem
    Key: /apps/automic/automationengine/bin/ucsrv_key.pem

    Check ucsrv.ini file to see if CAPKI is configured to point to these locations;

        # vi /apps/automic/automationengine/bin/ucsrv.ini

        certificate=/apps/automic/servicemanager/bin/../../automationengine/bin/ucsrv_certificate.pem
        key=/apps/automic/servicemanager/bin/../../automationengine/bin/ucsrv_key.pem
        chain=

    d) Restart ServiceManager and then use ServiceManager Dialogue. Now it will show as Secure (TLS 1.2)

    e) To install this into your machine, get CA.PKI from autotaskprd01 and install it with the following parameter;
    
        PS > .\setup.exe install caller=AE123

