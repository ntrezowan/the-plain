---
title: "Building F5 in AWS"
comments: false
description: "Building F5 in AWS"
keywords: "f5, ec2, aws, create, build, configure, deploy"
published: true

---


### Build F5 EC2 Instance

1. Go to [https://aws.amazon.com/marketplace](https://aws.amazon.com/marketplace) and search for `f5`. Here is an example AMI name;

		F5 BIG-IP VE - ALL (BYOL, 2 Boot Locations)

	License Type: Choose BYOL (Bring Your Own License) if you already purchased a license from F5. Choose PAYG (Pay As You Go) if you want to be charged based on usage
	
	Number of Boot Location: Choose 2 Boot Location if you have plan to upgrade F5 to a new version. Choose 1 Boot Location if you do not have any plan on upgrading in future
	
	ALL: This will allow you to provision all of the resources (LTM, ASM, AFM, APM etc.). You can also choose LTM/DNS only if you do not need to provision other modules

2. After you have decided on which version to install, click on it and then click on `Continue to Subscribe`

3. In the next page, Click on `Continue to Configuration`

4. The following page will allow you to choose the `Software Version` and `Region`. Select the appropriate version/region and click on `Continue to Launch`

5. In the `Launch this software` page, choose `Action` as `Launch through EC2` and click on `Launch`

6. In `Choose an Instance Type` page on the EC2 configuration, choose the instance type based on your network requirement (preferred m-type instance) and then click on `Configure Instance Details` and configure it as following;

	    Number of instances = 1 (if you are setting up a HA, choose more than 1 instance)
	    Network = vpc-04835675974ectt (this is the VPC where F5 will reside)
	    Subnet = subnet-00430005c1a2a133 (this is the subnet where F5 will reside)
	    Auto-assign Public IP = Disable (if you prefer all of your interfaces to be publicly accessible, then choose Enable)
	    Network Interface = (Add all the network interface that you need, you can also add them later when the instance is created. Here, we will have 4 interfaces with 1 management interface and 3 tmm interface and the IP will be automatically assigned by the VPC since it is configured to support DHCP. You can also put specific IP as Primary IP)
	    Device=eth01 | subnet=subnet-00430005c1a2a134 | Primary IP=auto-assigned
	    Device=eth02 | subnet=subnet-00430005c1a2a135 | Primary IP=auto-assigned
	    Device=eth03 | subnet=subnet-00430005c1a2a136 | Primary IP=auto-assigned
	    Device=eth04 | subnet=subnet-00430005c1a2a137 | Primary IP=auto-assigned

	Click on `Add Storage`

7. Go to https://support.f5.com/csp/article/K14946 and check how much disk space you need for the version you want to install. Click on `Encryption` and choose an existing KMS or create a new one. Click on `Add Tags`

8. Use appropriate Tags and click on `Configure Security Group`. Either create a new group or use an existing security group which serves your purpose and click on `Review and Launch`

9. Choose an existing Key Pair for SSH or create a new one (for example, f5-aws.pem) and click on `Launch Instance`

10. Click on `View Instances` and the EC2 will be deployed after 5-10 minutes. The instance will be ready to use when `Status Checks` shows as `2/2checks passed`

---

### Configure F5

1. SSH to the EC2 instance as following;

		# ssh -i f5-aws.pem admin@MANAGEMENT_IP

	It will not ask for password since this is a new installation and no password is set for the `admin` account yet

2. Run the following to set `admin` password;

		(tmos)# modify auth password admin
		changing password for admin
		new password:
		confirm password

3. Save the configuration;

		(tmos)# save sys config

4. Browse to the management IP (https://MANAGEMENT_IP) and active the license. You cannot install the license using CLI because admin account does not have permission to Advanced Shell yet
	a. Click `Next` on the `Welcome Page` after you logged in using the newly configured `admin` account
	b. Click on `Activate`. In the next page, fill the `Base Registration Key` and choose `Activation Method` as `Manual`
	c. Copy the `Dossier`, go to F5 licensing server (https://activate.f5.com/license/dossier.jsp), paste the Dossier you just copied in `Enter Your Dossier` field and click `Next`
	d. Copy the license, paste it in `License` section of F5 and click on `Next`

5. After licensing is completed, `Resource Provisioning` page will show up. In the `Platform` section, complete as following;

		Management Port Configuration: Automatic (DHCP) (If you specified the IP when you created the EC2 instance, then choose Manual and type the IP here)
		Host Name: f5aws.example.com
		Host IP Address: Use Management IP Address
		Time Zone: US/Eastern

6. Create VLANs;
When we created the EC2, we added three tmm interfaces, so we need to create three VLANs. 
To create the VLANs, go to `Network > VLANs` and click on `Create`. Configure tme as following;

		Name: vlanA
		Tag: 4090
		Interface: 1.1 
		Tagging: Untagged (Click Add so that it shows under Interfaces)

		Name: vlanB
		Tag: 4091
		Interface: 1.2 
		Tagging: Untagged (Click Add so that it shows under Interfaces)

		Name: vlanC
		Tag: 4090
		Interface: 1.3 
		Tagging: Untagged (Click Add so that it shows under Interfaces)

7. Create Self IP;

	Go to `Network > Self IPs` and click on `Create`. Configure them as following;

		Name: 10.1.1.1
		IP Address: 10.1.1.1
		Netmask: 255.255.255.0
		VLAN: vlanA
		Port Lockdown: Allow None
		Traffic Group: None

		Name: 10.1.2.1
		IP Address: 10.1.2.1
		Netmask: 255.255.255.0
		VLAN: vlanB
		Port Lockdown: Allow None
		Traffic Group: None

		Name: 10.1.3.1
		IP Address: 10.1.3.1
		Netmask: 255.255.255.0
		VLAN: vlanC
		Port Lockdown: Allow None
		Traffic Group: None

8. All the configuration has been completed at this point. Reboot F5 and start creating virtual servers.

---

NB: Do not use `EC2 > Instance > Actions > Instance State > Stop` to shutdown the F5 instance. If you do that, then F5 configuration will be corrupted. The proper way is the shutdown F5 first by running the following and then use stop the instance

	# shutdown -H now

If you accidently stopped the instance without shutting down F5 first, run the following so that `mcpd` can reload the configuration correctly;

	# bigstart restart mcpd
