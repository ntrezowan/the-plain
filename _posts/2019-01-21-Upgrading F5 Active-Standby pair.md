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
2. Verify license and renew if necessary;<br /><br />
BIG-IP license is stored at `/config/bigip.license` and it has two dates; `Licensed date` and `Service check date`.<br /><br />
`Licensed date` will show the date when you used your Registration Key for the first time to license your BIG-IP system. To find `Licensed date`, run the following;
```
# grep "Licensed date" /config/bigip.license
Licensed date :                    20180601
```
`Service Check Date` is the date when you last reactivated your license and it gets updated every time you reactivate your license (assuming that there is an active service contract with F5 for this BIG-IP system). For example, if you have reactivate your license on June 30, 2018 then it will show as 20180630. To find `Service Check Date`, run the following;
```
# grep "Service check date" /config/bigip.license
Service check date :               20171013
```
There is another interesting date called `License Check Date` and this date is related with the software version of BIG-IP. For example, Version `12.1.0-12.1.3` has a `License Check Date` 2016-03-2018. The `License Check Date` enforcement is applied during system startup. The system compares the `License Check Date` with the `Service Check Date` exists in the license file. If the `Service Check Date` is earlier than the `License Check Date`, the system will initialize but will not load the configuration. To allow the configuration to load properly, you must update the `Service Check Date` in the `bigip.license` file by reactivating the system's license. To find the `License Check Date` for the version you planned to upgrade, visit https://support.f5.com/csp/article/K7727. For example, if you plan to upgrade to version `13.1.0-13.1.1`, then `License Check Date` is 20170912.<br /><br />
Now by comparing `Service check date` with `License Check Date`, we see that 20171013 > 20170912 which means you do not need to reactive the license before upgrade. But it is always a good practice to reactivate your license everytime you upgrade the F5 because it extends your `Service check date` in the license file.<br /><br />
If `Service Check Date` < `License Check Date`, do the following to reactivate the license before upgrade;  
  a) Log in to the Configuration utility  
  b) Navigate to System > License > Reactivate  
  c) Select either Automatic or Manual (if F5 cannot reach internet)  
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
  
5. Generate a qkview and check for Upgrade Advisor in iHealth;<br /><br />
iHealth reports can be used to find if there is any issue if we upgrade F5 units from one version to another. To generate a qkview, do the following;  
  a) Log in to the Configuration utility  
  b) Navigate to System > Support  
  c) Click New Support Snapshot  
  d) For Health Utility, click Generate QKView  
  e) Click Start  
  f) To download the output file, click Download<br /><br />
After download the file from F5, upload it to https://ihealth.f5.com/ and then go to `Upgrade Advisor` and select the version to which you want to upgrade your units. Then check the recommended feedback.<br /><br />
For example, here is one advise that iHealth provided when we are upgrading from `12.1.3.4` to `13.1.1.3`;<br /><br />
TMOS vulnerability: Password changes for local users may not be preserved unless the configuration is explicitly saved (K37250780)  

6. Create a backup of the config file;<br /><br />
It is always good to have a backup of the config file before upgrade. This way, we can quickly restore F5 to previous stable state if there is any issues during upgrade.<br /><br />
To create UCS file, do the following;
  a) Log in to the Configuration utility  
  b) Navigate to System > Archives  
  c) To initiate the process of creating a new UCS archive, click Create  
  d) In the File Name box, type a name for the file  
  e) To create the UCS archive file, click Finished  
  f) When the system completes the backup process, examine the status page for any reported errors before proceeding to the next step  
  g) To return to the Archive List page, click OK  
  h) Copy the .ucs file to a secure file system (i.e. shared NFS)

7. Verify volume formatting scheme;<br /><br />
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

8. Import the software/hotfix image:<br /><br />
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
9. Check that root login to shell is possible;<br /><br />
In situation during the upgrade, F5 might not be able to access ADFS/LDAP and so your user/pass might not work. Check if you have a root account and if you can SSH to it by running the following;
```
ssh root@f5prd01.its.fsu.edu
ssh root@f5prd02.its.fsu.edu 
```

10. Check that admin login to GUI is possible;<br /><br />
In situation during the upgrade, F5 might not be able to access ADFS/LDAP and so your user/pass might not work. Check if you have a admin account and if you can login to GUI using the admin account;
```
https://f51.example.com
https://f52.example.com
```
---
### B. Check running configuration integrity (applied to all HA units) and disable Auto-Failover
If there is an issue in the running configuration, then it will give error after the upgrade. Check if there is any error in the configuration file by running the following command;
```
# tmsh load /sys config verify
Validating system configuration...
  /defaults/asm_base.conf
  /defaults/config_base.conf
  /defaults/ipfix_ie_base.conf
  /defaults/ipfix_ie_f5base.conf
  /defaults/low_profile_base.conf
  /defaults/low_security_base.conf
  /defaults/policy_base.conf
  /defaults/wam_base.conf
  /defaults/analytics_base.conf
  /defaults/apm_base.conf
  /defaults/apm_saml_base.conf
  /defaults/app_template_base.conf
  /defaults/classification_base.conf
  /var/libdata/dpi/conf/classification_update.conf
  /defaults/urlcat_base.conf
  /defaults/daemon.conf
  /defaults/pem_base.conf
  /defaults/profile_base.conf
  /defaults/sandbox_base.conf
  /defaults/security_base.conf
  /defaults/urldb_base.conf
  /usr/share/monitors/base_monitors.conf
Validating configuration...
  /config/bigip_base.conf
  /config/bigip_user.conf
  /config/bigip.conf
  /config/bigip_script.conf
  /config/partitions/Sandbox/bigip.conf
There were warnings:
SSLv2 is no longer supported and has been removed. The 'sslv2' keyword in the cipher string of the ssl profile (/Common/clientssl-test) has been ignored.
SSLv2 is no longer supported and has been removed. The 'sslv2' keyword in the cipher string of the ssl profile (/Common/serverssl-test) has been ignored.
```
Fix the issues before you go forward with upgrading F5. `WARNING` can be ignored but not suggested.<br /><br />
To turn off Auto-Failback on both F5 before upgrade to prevent active-active condition, do the following in v13 or above;
  a) On the Main menu, Click Device Management > Traffic Group > traffic-group-1  
  b) Uncheck "Always Failback to First Device if it is Available"  
  c) Click Save  
  d) Do a config sync<br /><br />
Do the following to disable Auto failover in v12 or below;
  a) On the Main menu, Click Device Management > Traffic Groups  
  b) Select traffic-group-1  
  c) Select Advanced from Configuration  
  d) Uncheck Auto Failover  
  e) Click Update  

### C. Open a proactive service request with F5 Technical Support
Proactive service requests provide F5 Technical Support advance notice of your maintenance window to save time in case a problem arises that requires F5 Technical Support assistance. Here is more on this;
https://support.f5.com/csp/article/K16022

### D. Upgrading the units

---DO-NOT-SYNC---

### Upgrade F52.example.com (standby unit first)

2.	Force F5PRD02 to offline state

a)	On the Main menu, click Device Management > Devices .
b)	Click the name of F5PRD02.
c)	The device properties screen opens.
d)	Click Force Offline.
e)	F5PRD02 changes to offline state.

Once F5PRD02 changes to offline state, ensure that traffic passes normally for all active traffic groups on the other devices.

3.	Reactivate software license if required

a)	On the Main menu, click System > License .
b)	Click Re-activate.
c)	For the Activation Method setting, select the Automatic (requires outbound connectivity) option.
d)	Click Next.
e)	The BIG-IP software license renews automatically.
f)	Click Continue.


4.	Reboot the machine before upgrade
a)	Reboot F5PRD02 to clean up memory
During reboot, check logs
# tail -f /var/log/ltm

b)	After reboot, check the running configuration integrity
# tmsh load sys config verify 

c)	Restart mcpd and then reboot
# touch /service/mcpd/forceload
# reboot

Check logs
# tail -f /var/log/ltm
# tail -f /var/log/ltm | egrep “err|warn”
# tail -f/var/log/liveinstall.log
# egrep 'err|warn' /var/log/ltm



5.	Install the new version software

a)	Log in to the Configuration utility with administrative privileges.
b)	Navigate to System > Software Management > Image List.
c)	Select the Software Image and click Install. A new window will pop up called Install Software Image.
d)	Select an available disk from the Select Disk menu. Here HD1 is an LVM disk.
e)	Select an empty volume set from the Volume Set Name menu, or type a new volume set name. Volumes are named as HD1.1, HD1.2, HD1.3 etc, so to create a new volume, type “3” and it will create HD1.3 and install the image there. 
f)	Click Install.
g)	To see the installation progress, view the Install Status column of the Installed Images section of the page.


6.	Reboot to the newly upgraded software volume

a)	Log in to the Configuration utility with administrative privileges.
b)	Navigate to System > Software Management > Boot Locations.
c)	If you select Install Configuration to Yes, it will ask from where you want to copy the configuration from. Choose the latest one.

If there have been no changes since you performed the upgrade and /or make any changes in the configuration and syncs, you do not need to set the Install Configuration option to Yes when activating the new volume. But if you make any changes, it’s better to select Yes when activating the new volume.

d)	Click the boot location containing the newly upgraded software volume.
e)	To restart the system to the specified boot location, click Activate.
f)	To close the confirmation message, click OK
g)	Check which boot location is loaded after reboot
# watch tmsh show sys software

h)	Also check if installation fails;
# tail -f/var/log/liveinstall.log

i)	Check LTM logs
# tail -f /var/log/ltm

7.	Check F5PRD02 version after reboot
a)	Go to System > Configuration > Device > General
b)	Check the version


8.	Bring F5PRD02 to Standby state

a)	Release Device A from offline state.
b)	On the Main menu, click Device Management > Devices .
c)	Click the name of Device A.
d)	The device properties screen opens.
e)	Click Release Offline.
f)	F5PRD02 changes to standby state.

The new version of BIG-IP software is installed on F5PRD02, with all traffic groups in standby state.


# Make F5PRD02 the active load balancer #
 	
1.	Force F5PRD01 to standby state

a)	Login to F5PRD01
b)	On the Main menu, click Device Management > Devices .
c)	Click the name of F5PRD01.
d)	The device properties screen opens.
e)	Click Force to Standby.
f)	F5PRD01 changes to standby state.

Once F5PRD01 changes to offline state, ensure that traffic passes normally for all active traffic groups on the other devices.

2.	Verify that F5PRD02 is the active load balancer
3.	Verify expected objects appear in the shared and non-shared portions of the configuration.
a)	To verify that the expected objects appear in the shared and non-shared portions of the configuration, navigate to Local Traffic > Pools.
Confirm that the expected objects are present and compare with F5PRD01.
b)	Navigate to Network > VLANs.
Confirm that the expected objects are present and compare with F5PRD01
c)	Go to iApps, open a VIP and go to Reconfigure to see if everything is loading properly.

4.	Check the most recent logs (/var/log/ltm for example) for obvious signs of issues like repeating messages. Comparing logs to the active unit or to the logs prior to the upgrade can be helpful. 
5.	Check SSL with SSLLabs
6.	Generate a qkview and review it in the iHealth Diagnostics section for currently known issues.
7.	Contact teams to begin application testing. If testing successes, processed with upgrading F5PRD01. Otherwise check “Backing out software upgrade” 

# Upgrade F5PRD01 # 

1.	Force F5PRD01 to offline state

a)	On the Main menu, click Device Management > Devices .
b)	Click the name of F5PRD01.
c)	The device properties screen opens.
d)	Click Force Offline.
e)	F5PRD01 changes to offline state.

Once F5PRD01 changes to offline state, ensure that traffic passes normally for all active traffic groups on the other devices.

2.	Reactivate software license if required

a)	On the Main menu, click System > License .
b)	Click Re-activate.
c)	For the Activation Method setting, select the Automatic (requires outbound connectivity) option.
d)	Click Next.
e)	The BIG-IP software license renews automatically.
f)	Click Continue.


3.	Reboot the machine before upgrade
d)	Reboot F5PRD01 to clean up memory
During reboot, check logs
# tail -f /var/log/ltm

e)	After reboot, check the running configuration integrity
# tmsh load sys config verify 

f)	Restart mcpd and then reboot
# touch /service/mcpd/forceload
# reboot

Check logs
# tail -f /var/log/ltm
# tail -f /var/log/ltm | grep err
# tail -f/var/log/liveinstall.log


4.	Install the new version software

a)	Log in to the Configuration utility with administrative privileges.
b)	Navigate to System > Software Management > Image List.
c)	Select the Software Image and click Install. A new window will pop up called Install Software Image.
d)	Select an available disk from the Select Disk menu. Here HD1 is an LVM disk.
e)	Select an empty volume set from the Volume Set Name menu, or type a new volume set name. Volumes are named as HD1.1, HD1.2, HD1.3 etc, so to create a new volume, type “3” and it will create HD1.3 and install the image there. 
f)	Click Install.
g)	To see the installation progress, view the Install Status column of the Installed Images section of the page. It will take some time.


5.	Reboot to the newly upgraded software volume

a)	Log in to the Configuration utility with administrative privileges.
b)	Navigate to System > Software Management > Boot Locations.
c)	If you select Install Configuration to Yes, it will ask from where you want to copy the configuration from. Choose the latest one.

If there have been no changes since you performed the upgrade and /or make any changes in the configuration and syncs, you do not need to set the Install Configuration option to Yes when activating the new volume. But if you make any changes, it’s better to select Yes when activating the new volume.

d)	Click the boot location containing the newly upgraded software volume.
e)	To restart the system to the specified boot location, click Activate.
f)	To close the confirmation message, click OK
g)	Check which boot location is loaded after reboot
# watch tmsh show sys software

h)	Also check if installation fails;
# tail -f /var/log/liveinstall.log

i)	Check LTM logs
# tail -f /var/log/ltm | grep err

6.	Check F5PRD01 version after reboot
c)	Go to System > Configuration > Device > General
d)	Check the version


7.	Force F5PRD01 to standby state

a)	Release F5PRD01 from offline state.
b)	On the Main menu, click Device Management > Devices .
c)	Click the name of F5PRD01.
d)	The device properties screen opens.
e)	Click Release Offline.
f)	F5PRD01 changes to standby state.

The new version of BIG-IP software is installed on F5PRD01, with all traffic groups in standby state.

# Do SYNC

1.	From F5PRD01, sync configuration with F5PRD02
a)	Log in to the Configuration utility.
b)	Navigate to Device Management > Overview.
c)	For Device Groups, click the name of the device group (device-group-a-failover or datasync-global-dg) you want to synchronize.
d)	For Devices, click the name of the device from which you want to perform the synchronization action.
e)	For Sync, click the appropriate synchronization action.
f)	Click Sync.

2.	Force F5PRD02 to be standby mode
a)	Login to F5PRD02
b)	On the Main menu, click Device Management > Devices .
c)	Click the name of F5PRD02.
d)	The device properties screen opens.
e)	Click Force to Standby.
f)	F5PRD02 changes to standby state.

3.	Enable Auto failover and sync
Do the following to disable Network failover;
a)	On the Main menu, Click Device Management > Device Groups.
b)	Select device-group-a-failover
c)	Select Advanced from Configuration
d)	Uncheck Network Failover
e)	Click Update
f)	Go to each F5 device and verify/create the backup cron job in /etc/cron.daily/ bigip_ltm_11_backup.sh and /etc/cron.monthly/remote_qkview_script is with permission 700.
g)	

