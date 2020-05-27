---
title: "Encrypt EC2 EBS Root Volume in AWS"
comments: false
description: "Encrypt EC2 EBS Root Volume in AWS"
keywords: "ec2, ebs, root, volume, aws, encryption, decryption, encrypt, decrypt"
published: true

---

 
### 1. Create IAM KMS encryption key

a. Go to KMS > Customer managed keys and click on Create Key

b. Choose Key Type as Symmetric and click Next
	
c. Set an Alias/Tag and click Next
	
d. Choose the Key administrators and click Next.
	
e. In the next page, verify the policy and click Finish.
		
### 2. Check the exisitng partition table

	a. SSH to the EC2 instance and check the curent partition;

	# lsblk
	NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
	xvda    202:0    0   8G  0 disk
	└─xvda1 202:1    0   8G  0 part /


b. Create a file and check later to see if all files are available after restore;

	touch abc.txt
		
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
SSH to the EC2 instance and check the curent partition;

	# lsblk 
	NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT 
	xvda    202:0    0   8G  0 disk
	└─xvda1 202:1    0   8G  0 part /
		
	# ls -la

9. Delete the old volume and the two snapshots (both unencrypted and encrypted)
	

---

### Asymmetric Key encryption 
1. To use asymmetric encryption, we first need to create a public-private key pair. Run the following to create a pair;

        # gpg --gen-key
        gpg (GnuPG) 2.0.22; Copyright (C) 2013 Free Software Foundation, Inc.
        This is free software: you are free to change and redistribute it.
        There is NO WARRANTY, to the extent permitted by law.

        Please select what kind of key you want:
           (1) RSA and RSA (default)
           (2) DSA and Elgamal
           (3) DSA (sign only)
           (4) RSA (sign only)
        Your selection? 1
        RSA keys may be between 1024 and 4096 bits long.
        What keysize do you want? (2048) 
        Requested keysize is 2048 bits
        Please specify how long the key should be valid.
                 0 = key does not expire
              <n>  = key expires in n days
              <n>w = key expires in n weeks
              <n>m = key expires in n months
              <n>y = key expires in n years
        Key is valid for? (0) 0
        Key does not expire at all
        Is this correct? (y/N) y

        GnuPG needs to construct a user ID to identify your key.

        Real name: user1
        Email address: user1@example.com
        Comment: 
        You selected this USER-ID:
            "user1 <user1@example.com>"

        Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
        You need a Passphrase to protect your secret key.

    At this point it will ask for a passphrase. This is the passphrase which you will use to decrypt encrypted files send to you along with your private key. You should not share this passphrase and private key with anyone.

        We need to generate a lot of random bytes. It is a good idea to perform
        some other action (type on the keyboard, move the mouse, utilize the
        disks) during the prime generation; this gives the random number
        generator a better chance to gain enough entropy.
        We need to generate a lot of random bytes. It is a good idea to perform
        some other action (type on the keyboard, move the mouse, utilize the
        disks) during the prime generation; this gives the random number
        generator a better chance to gain enough entropy.
        gpg: /home/user1/.gnupg/trustdb.gpg: trustdb created
        gpg: key C56171B8 marked as ultimately trusted
        public and secret key created and signed.

        gpg: checking the trustdb
        gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
        gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
        pub   2048R/C56171B8 2019-12-18   
              Key fingerprint = 148D B70C 7D30 5DED 6EDC  FB54 F90F 870A C561 71B8
        uid                  user1 <user1@example.com>
        sub   2048R/78C8F6D8 2019-12-18   

    The newly created pair is located here;
    
        # ls ~/.gnupg/
        gpg.conf  private-keys-v1.d  pubring.gpg  pubring.gpg~  random_seed  secring.gpg  S.gpg-agent  trustdb.gpg

2. To see all the GPG keys;

        # gpg --list-keys
        /home/user1/.gnupg/pubring.gpg
        -----------------------------------
        pub   2048R/C56171B8 2019-12-18   
        uid                  user1 <user1@example.com>
        sub   2048R/78C8F6D8 2019-12-18   

3. To encrypt `input.txt` with this new asymmetric key, do the following;

        gpg --encrypt --recipient user1@example.com input.txt

4. To decrypt a file using your private key and passphrase, do the following;

        # gpg --output output.txt --decrypt input.txt.gpg 

        You need a passphrase to unlock the secret key for
        user: "user1 <user1@example.com>"
        2048-bit RSA key, ID 78C8F6D8, created 2019-12-18   (main key ID C56171B8)

        gpg: encrypted with 2048-bit RSA key, ID 78C8F6D8, created 2019-12-18   
              "user1 <user1@example.com>"

5. If you want to export your public key, do the following; 

        gpg --armor --output public_key.cer --export 'user1@example.com'

6. If you want to export your private key for backup purposes or anything else, do the following;  

        # gpg --armor --output private_key.asc --export-secret-keys 'user1@example.com'

7. To check when the key will expire;  

        # gpg --edit-key user1@example.com
        gpg (GnuPG) 2.0.22; Copyright (C) 2013 Free Software Foundation, Inc.
        This is free software: you are free to change and redistribute it.
        There is NO WARRANTY, to the extent permitted by law.

        Secret key is available.

        pub  2048R/C56171B8  created: 2019-12-18  expires: never       usage: SC  
                             trust: ultimate      validity: ultimate
        sub  2048R/78C8F6D8  created: 2019-12-18  expires: never       usage: E   
        [ultimate] (1). user1 user1@example.com

8. To check all contents of the public key ring;

        # gpg --check-sigs
        /home/user1/.gnupg/pubring.gpg
        -----------------------------------
        pub   2048R/C56171B8 2019-12-18   
        uid                  user1 <user1@example.com>
        sig!3        C56171B8 2019-12-18  user1 <user1@example.com>
        sub   2048R/78C8F6D8 2019-12-18  
        sig!         C56171B8 2019-12-18  user1 user1@example.com

9. To check all contents of the private key ring;

        # gpg --list-secret-keys
        /home/user1/.gnupg/secring.gpg
        -----------------------------------
        sec   2048R/C56171B8 2019-12-18  
        uid                  user1 <user1@example.com>
        ssb   2048R/78C8F6D8 2019-12-18  

