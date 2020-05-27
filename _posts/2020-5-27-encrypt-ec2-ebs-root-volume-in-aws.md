---
title: "Encrypt EC2 EBS Root Volume in AWS"
comments: false
description: "Encrypt EC2 EBS Root Volume in AWS"
keywords: "ec2, ebs, root, volume, aws, encryption, decryption, encrypt, decrypt"
published: true

---

### How EBS encrypts/decrypts a volume

1. An encrypted EBS volume requests for `Data Encryption Key` to KMS upon creation

2. KMS sends a `Data Encryption Key` encrypted by the `Customer Managed Key` which is stored in the EBS volume metadata

3. When an EC2 instance is launched, the instance (not the EBS volume) sends a request to KMS to decrypt the  `Data Encryption Key`

4. KMS decrypts the `Data Encryption Key` using the `Customer Managed Key` and sends it as plaintext to EC2 instance

5. EC2 instance decrypts the volume using the Data Encryption Key

 
---


### Encrypting the Root Volume 

1. Create a `Customer Managed Key`

	a. Go to `KMS > Customer managed keys` and click `Create Key`

	b. Choose `Key Type` as `Symmetric` and click Next

	c. Set an `Alias` and click Next

	d. Choose the `Key administrators` and click Next

	e. Verify the policy and click Finish

		
2. Check existing volume

	a. SSH to the EC2 instance and check current volumes

		# lsblk
		NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
		xvda    202:0    0   8G  0 disk
		└─xvda1 202:1    0   8G  0 part /

	b. Create a file to check later if this file is available after restore

		# touch abc.txt
		
		
3. Create snapshot of the root volume

	a. Go to `EC2 > Volumes`, and choose the unencrypted root volume
	
	b. Select `Action > Create Snapshot`
	
	c. Open the newly created snapshot and name it something as `ec2name-unencrypted-snapshot`
		
		
4. Encrypt the new snapshot 

	a. Go to `EC2 > Snapshots`, choose the new snapshot, select `Action > Copy`
	
	b. Check the `Destination Region` and name it as `ec2name-encrypted-snapshot`
	
	c. Select `Encryption`, choose the `Master Key` that you have just created and click Finish
	
	d. Verify that the snapshot is encrypted with the KMS key
		
		
5. Create a new encrypted volume from the encrypted snapshot and attach to the instance

	a. Go to `EC2 > Snapshots`, choose the encrypted snapshot, and select `Action > Create Volume`
	
	b. Name it `ec2name-encrypted-volume`, check the AZ (both EC2 and Volume needs to be in the same AZ), and `Master Key`
	
	c. Click `Create Volume`
		
		
6. Detach the existing unencrypted volume from the EC2 and attach the encrypted volume

	a. Go to `EC2 > Instances`, choose the instance and select `Action > Instance State > Stop`
	
	b. Go to `EC2 > Volumes`, choose the unencrypted volume (in-use state), select `Action > Detach Volume`
	
	c. Go to `EC2 > Volumes`, choose the encrypted volume, select `Action > Attach Volume`
	
	d. Choose the `Instance ID` to select the `Instance`, modify `Device` to `/dev/xvda` and click `Attach`
		
		
7. Start the instance

	a. Go to `EC2 > Instances`, choose the instance

	b. Select `Action > Instance State > Start`
	
	
8. Check the partition table

	a. SSH to the EC2 instance and check current volume

		# lsblk 
		NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT 
		xvda    202:0    0   8G  0 disk
		└─xvda1 202:1    0   8G  0 part /
		
	b. Check if the previously created file exists
	
		# ls -la


9. Delete the old volume and the two snapshots (both unencrypted and encrypted)
