---
title: "Installing OpenNMS on Ubuntu Server"
---

#### Adding OpenNMS repository

1. Add the repository from `/etc/apt/sources.list.d`;
```
# nano /etc/apt/sources.list
```
Add the following lines in `sources.list`;
```
deb http://debian.opennms.org stable main
deb-src http://debian.opennms.org stable main
```
Add OpenNMS PGP key;
```
# wget -O - http://debian.opennms.org/OPENNMS-GPG-KEY | sudo apt-key add -
```

4. Update repository cache;
```
# sudo apt-get update
```

___


#### Installing and configuring OpenNMS

1. Install OpenNMS using OpenMNS repo;
```
# sudo apt-get install opennms
```
It will install OpenNMS along with PostgreSQL database.

2. Now edit `pg_hba.conf` to grant permission to postgres user to access the database without password;
```
# cd /etc/postgresql/8.4/main/
# nano pg_hba.conf
```
Add the following lines;
```
local all all trust
host all all 127.0.0.1/32 trust
```
Now edit `postgresql.conf` to change the `listen_addresses`;
```
# nano postgresql.conf
```
Add the following lines;
```
listen_addresses = 'localhost'
max_connections = 100
```
Start postgresql database;
```
# /etc/init.d/postgresql restart
```

3. Prepare OpenNMS database;
```
# sudo -u postgres createdb -U postgres -E UNICODE opennms
```
Check if the database is working properly;
```
# psql -U postgres --host=localhost opennms
```

___

#### Installing IPLIKE

1. Install IPLIKE as postgres user, to do so we need to create a postgres system account;
```
# useradd postgres
# sudo -u postgres
# cd /usr/sbin/
# install_iplike.sh
```

2. Setting up Java environment;
```
# sudo /usr/share/opennms/bin/runjava -s
# sudo /usr/share/opennms/bin/runjava -S /usr/java/jdk1.5.0_12/bin/java
```

3. Run OpenNMS installer;
```
# sudo /usr/share/opennms/bin/install -dis
```

4. Start OpenNMS;
```
root@opennms:~# sudo service opennms start
```

___

Browse to `http://localhost:8980/opennms/` and log in as `admin` using password `admin`.
