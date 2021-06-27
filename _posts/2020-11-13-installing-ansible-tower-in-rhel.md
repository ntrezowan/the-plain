---
title: "Installing Ansible Tower in RHEL"
comments: false
description: "Installing Ansible Tower in RHEL"
keywords: "ansible, tower, rhel, 8"
published: true

---


1. Install EPEL Repository;
        
        $ dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

2. Install Ansible;

        $ yum install ansible 

3. Download Ansible Tower;

        $ mkdir /tmp/ansible-tower-setup
        $ cd /tmp/ansible-tower-setup
        $ curl -k -O https://releases.ansible.com/ansible-tower/setup/ansible-tower-setup-latest.tar.gz

4. Download Ansible Tower;

        $ tar xvf ansible-tower-setup-latest.tar.gz -C /opt/
        $ cd /ansible-tower-setup-x.x.x-x

5. Modify inventory file and add the following;

        $ vi inventory

        admin_password='0'
        pg_password='0'

6. Install Ansible Tower (this will run install.yml playbook)

        $ ./setup.sh


7. Go to https://IP_ADDRESS and login with admin user and then use your Red Hat or Red Hat Satellite credentials to install the license.
