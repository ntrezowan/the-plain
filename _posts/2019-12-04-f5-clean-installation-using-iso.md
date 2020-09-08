---
title: "F5 Clean Installation Using ISO"
comments: false
description: "F5 Clean Installation Using ISO"
keywords: "f5, iso, tmos, installation, usb, disk, drive, bootable"
published: true

---



### Install using USB media

A. Create a bootable USB drive using your computer

1. Download the image from https://downloads.f5.com/ into you computer

2. Mount the image;

# mkdir /mnt/cd
# mount -o loop /shared/images/BIGIP-13.1.1.5-0.0.4.iso /mnt/cd
mount: /shared/images/BIGIP-13.1.1.5-0.0.4.iso is write-protected, mounting read-only

3. Insert a USB disk and run mkdisk script;

# cd /mnt/cd
# ./mkdisk

In the next step, F5 will ask you the following questions. Answer as appropriate to your environment;

7047 blocks
On which F5 platform will the installation media be used?
Index Platform
1 BIG-IP 1500
2 BIG-IP 1600
3 BIG-IP 3400
4 BIG-IP 3410
5 BIG-IP 3600
6 BIG-IP 3900
7 BIG-IP 4100
8 BIG-IP 6400
9 BIG-IP 6800
10 BIG-IP 6900
11 BIG-IP 8400
12 BIG-IP 8800
13 BIG-IP 8900
14 BIG-IP 8950
15 BIG-IP 11050
16 BIG-IP Viprion
17 BIG-IP_SAM 4300
18 Enterprise Manager EM500
19 Enterprise Manager EM3000
20 FirePass 1200
21 FirePass 4100
22 FirePass 4300

Please select a device by index (1 – 26) —>

Index Device Product Size Notes

1 /dev/sdb SanDisk Cruzer Blade 3819 MB May hold 2 products
Please select a device by index (1 – 1) —>1

Chosen device /dev/sdb is SanDisk Cruzer Blade

WARNING: The next step will destroy all data on this device!
Are you sure you want to continue? (y|n) [n] –> y

info: FAT Formatting device /dev/sdb
info: Installing bootloader on /dev/sdb
info: Mounting device /dev/sdb on /mnt/ozimLXTaYx
copying ./isolinux/vmlinuz => /mnt/ozimLXTaYx
copying ./isolinux/initrd.img => /mnt/ozimLXTaYx
The following products are available for transfer:
BIG-IP version 10.2.0

Would you like to transfer BIG-IP version 10.2.0? (y|n) [y] –> y
Queueing BIG-IP version 10.2.0
Ready to copy one product (this will take a while…).
Creating repository ISO in /mnt/tm_install/16082.mJRdtt (this will take a while)…
Using TS_MN000.RPM;1 for ././BIGIP1020/i686/TS-mng-fbagent-10.2.0-1707.0.i686.rpm (TS-mng-fbloader-10.2.0-1707.0.i686.rpm)
Using TMM_V000.RPM;1 for ././BIGIP1020/i686/tmm-vadc-debug-10.2.0-1707.0.i686.rpm (tmm-vadc-10.2.0-1707.0.i686.rpm)

1.25% done, estimate finish Sun Jun 26 21:58:00 2011
98.76% done, estimate finish Sun Jun 26 21:58:27 2011
Total translation table size: 0
Total rockridge attributes bytes: 72463
Total directory bytes: 135168
Path table size(bytes): 304
Max brk space used a3000
399956 extents written (781 MB)
Inserting md5sum into iso image…
md5 = cce0625d016f8571974647453cc6fd78
Inserting fragment md5sums into iso image…
fragmd5 = e3e88cee8adb0fcb92fe92e3a465ceb2b49ee5c4d1e47e46e153d687dcce
frags = 20
Setting supported flag to 0
done
Moving repository ISO to thumb drive (this will take a while) …done
Product setup complete.
Flushing disk buffer (this may take a while)… done

4. When the script completes, unmount the USB media;

# umount /mnt/cd

If you are having trouble unmounting the disk, run the following to see which user is using it and then kill the process;

# fuser /mnt/cd
# fuser -k /mnt/cd
# umount /mnt/cd

B. Boot F5 using the USB media

1. Boot F5 so that it can boot into Maintenance Operating System (MOS) from the USB

2. Choose Install the image

3. Select the default terminal emulation (vt100)

4. The following command will delete all data present in the F5 system;

# diskinit --style volumes

5. The following command will install the software;

# image2disk --format=volumes --nosaveconfig --nosavelicense

6. After the installation, remove the USB and reboot




### A. Get the latest AMI list of F5 images
1. Go to [https://github.com/F5Networks/f5-aws-cloudformation](https://github.com/F5Networks/f5-aws-cloudformation)

2. Browse to `AMI Maps` folder, choose the version you want to use and download the JSON file of either BYOL (Bring Your Own License) or PAYG (Pay As You Go). This is the AMI list that will be added in the CloudFormation template

3. If you want to use an older version, use the AWS Matrix ([https://github.com/F5Networks/f5-aws-cloudformation/blob/master/aws-bigip-version-matrix.md](https://github.com/F5Networks/f5-aws-cloudformation/blob/master/aws-bigip-version-matrix.md)) to find out the `Release Tags` of the version you want. Then go to [https://github.com/F5Networks/f5-aws-cloudformation](https://github.com/F5Networks/f5-aws-cloudformation) and use the `Tags` under `Master` branch to find the `Release Tag` you have chosen. Now go to the `AMI Maps` folder and you will see the JSON file of the older versions

---

### B. Create CouldFormation Template
1. Go to [https://github.com/F5Networks/f5-aws-cloudformation](https://github.com/F5Networks/f5-aws-cloudformation)

2. Browse to `supported` folder, choose either Standalone, Failover or Autoscale and then choose the number of network interfaces you needed for the F5. Now choose BYOL, or PAYG and save the template file (for example, f5-existing-stack-byol-n-nic-bigip.template which has been used for this configuration) as a local text file (f5-cf.txt)

3. Open `f5-cf.txt` and search for `BigipRegionMap`. Replace all the region declaration with the AMI list obtained previously

---

### C. Create EC2 instance
1. Go to `AWS Management Console > CloudFormation`. Click on `Create Stack`. Choose `Upload a template file` under `Specify Template`. Upload `f5-cf.txt`, click `Next` and configure as following;

        Stack Name: f5-cf

        VPC: vpc-0223567fe5974c63 (Choose the VPC where F5 will reside)
        Management Subnet AZ1: subnet-0d3a58dfc40277fa (Choose the subnet ID which will be used for management interface, AWS will dynamically assign the IP)
        Subnet1 in AZ1: subnet-0d3a58dfc40277fb (Choose the subnet ID for additional tmm interface, AWS will dynamically assign the IP)
        Subnet1 in AZ2: subnet-0d3a58dfc40277fc (Choose the subnet ID for additional tmm interface, AWS will dynamically assign the IP)
        Number Of Additional NICs: 3 (Adding three more interfaces)
        Additional NIC Location: subnet-0d3a58dfc40277fd, subnet-0d3a58dfc40277fe, subnet-0d3a58dfc40277ff (adding 3 additional subnet ID to have three more tmm interfaces)
        Provision Public IP addresses for the BIG-IP interfaces: No (Choose Yes if you want these interfaces to have public IP)

        BIG-IP Image Name: AllTwoBootLocation (If you do not need to upgrade F5 in future, you can choose AllOneBootLocation. If you prefer to provision LTM only, then choose image name that starts with LTM*)
        AWS Instance Size: m5.4xlarge (You need m5.4xlarge or larger since this instance type supports up to 8 interfaces. Check instance type in https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html to know what type of instance you need to support your required network interfaces)

        License Key 1: XXXXX-XXXXX-XXXXX-XXXXX-XXXXXXX (Input your license key)
        SSH Key: f5-aws.pem (Choose the SSH that will be used to SSH to F5)
        Source Address(es) for Management Access: 10.0.1.0/28 (Only these IPs are allowed to SSH to F5)
        Source Address(es) for Web Application Access (80/443): 10.0.1.0/28 (Only these IPs are allowed to reach the Configuration Utility)
        NTP Server: ntp1.example.com,ntp2.example.com,ntp3.example.com
        Timezone (Olson): US/Eastern
        BIG-IP Modules: ltm:medium, asm:nominal,afm:nominal,apm:nominal (specify resource provisioning here)

2. Click `Next` (unless you need to specify the IAM role to run this stack in CloudFormation) and finally click `Create stack`

4. It will create an EC2 instance called `Big-IP:f5-cf` and after 10-15 minutes, you will be able to access the Configuration Utility using the IP of management interface

---

Note: You cannot encrypt the disk volume of the EC2 instance if you use CloudFormation. If you need to encrypt the volume, then use [https://github.com/f5devcentral/f5-bigip-image-generator/](https://github.com/f5devcentral/f5-bigip-image-generator/). 
