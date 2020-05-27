---
title: "Encrypt EC2 EBS Root Volume in AWS"
comments: false
description: "Encrypt EC2 EBS Root Volume in AWS"
keywords: "ec2, ebs, root, volume, aws, encryption, decryption, encrypt, decrypt"
published: true

---

 
1. Create IAM KMS encryption key

	a. Go to KMS > Customer managed keys and click on Create Key

	b. Choose Key Type as Symmetric and click Next

	c. Set an Alias/Tag and click Next

	d. Choose the Key administrators and click Next

	e. In the next page, verify the policy and click Finish

		
2. Check the exisitng partition table

	a. SSH to the EC2 instance and check the curent partition;

		# lsblk
		NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
		xvda    202:0    0   8G  0 disk
		└─xvda1 202:1    0   8G  0 part /

	b. Create a file and check later to see if all files are available after restore;

		# touch abc.txt
		
3. Create snapshot of the root volume (/dev/xvda) 

	a. Select the unencrypted volume
	
	b. Select Action > Create Snapshot.
	
	c. Open the newly created Snapshot and name it as ec2name-unencrypted-snapshot.
		
4. Encrypt the newly created snapshot 

	a. On the newly created snapshot, select Action > Copy.
	
	b. Check the Destination Regoion and name it as ec2name-encrypted-snapshot.
	
	c. Enable Encryption, choose the Master Key and click Finish.
	
	d. Verify that this copied snapshot is encrypted
		
5. Create a new encrypted volume from the encrypted snapshot

	a. Choose the encrypted snapshot, select Action > Create Volume.
	
	b. Name it ec2name-encrypted-volume, check the AZ (both EC2 and Volume needs to be in the same AZ), and Master Key. 
	
	c. Click Create Volume and open the volume
		
6. Detach the existing unencryoted volume and replace with the encrypted volume

	a. Go to the EC2, select Action > Instance State > Stop.
	
	b. Go to Volumes, select the unencrypted volume (in-use state), select Action > Detach Volume.
	
	c. Go to Volumes, select the encrypted volume (ec2name-encrypted-volume), select Action > Attach Volume.
	
	d. Choose the original instance where you want to attach this volume, modify the Device to /dev/xvda (previous name) and click Attach.
		
7. Start the instance
	
8. Check the partition table

	a. SSH to the EC2 instance and check the curent partition;

		# lsblk 
		NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT 
		xvda    202:0    0   8G  0 disk
		└─xvda1 202:1    0   8G  0 part /
		
	b. Check if the previouslu created file exists;
	
		# ls -la

9. Delete the old volume and the two snapshots (both unencrypted and encrypted)
