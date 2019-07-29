---
title: "Installing Jenkins on Tomcat"
comments: false
description: "Installing Jenkins on Tomcat"
keywords: "jenkins, tomcat, publish over ssh"
published: true
---

### A. Install Java

1. Install OpenJDK;
```
# yum install java-x.x.x-openjdk.x86_64
```

2. Check Java version;
```
# java -version
openjdk version "x.x.x_xxx"
OpenJDK Runtime Environment (build x.x.x_xxx-xxx)
OpenJDK 64-Bit Server VM (build xx.xxx-xxx, mixed mode)
```

3. Setup `JAVA_HOME` and `JRE_HOME` path for all user;
```
# sudo cp /etc/profile /etc/profile_backup
# echo 'export JAVA_HOME=/usr/lib/jvm/jre-x.x.x-openjdk' | sudo tee -a /etc/profile
# echo 'export JRE_HOME=/usr/lib/jvm/jre' | sudo tee -a /etc/profile
source /etc/profile
```

4. Verify the paths;
```
# echo $JAVA_HOME
/usr/lib/jvm/jre-x.x.x-openjdk
```
```
# echo $JRE_HOME
/usr/lib/jvm/jre
```

---

### B. Install Tomcat

1. Download Tomcat from `http://tomcat.apache.org/` and extract;
```
# cd /opt
# wget http://us.mirrors.quenda.co/apache/tomcat/tomcat-x/vx.x.xx/bin/apache-tomcat-x.x.xx.tar.gz
# tar xzf apache-tomcat-x.x.xx
# mv apache-tomcat-x.x.xx tomcat
```

2. Configure a Tomcat user for Tomcat Web Application Manager;
```
# vi /opt/tomcat/conf/tomcat-users.xml
```    
```    
    <?xml version='1.0' encoding='utf-8'?>
    <tomcat-users>
        <role rolename="manager-gui"/>
        <role rolename="manager-script"/>
        <role rolename="manager-jmx"/>
        <role rolename="manager-jmx"/>
        <role rolename="admin-gui"/>
        <role rolename="admin-script"/>
        <user username="tomcat" password="tomcat" roles="manager-gui,manager-script,manager-jmx,manager-status,admin-gui,admin-script"/>
    </tomcat-users>
```

3. Open port `8080` from firewalld and reload the configuration;
```
# firewall-cmd --zone=public --permanent --add-port=8080/tcp
# firewall-cmd –reload
```

4. Start tomcat;
```
# cd /opt/tomcat/bin
# ./startup.sh
```

5. Check if the server is listening on port 8080;
```
# netstat -an | grep 8080
tcp6       0      0 :::8080       :::*          LISTEN
```

6. Browse to `http://localhost:8080` and Tomcat index page should load.

---

### C. Install Jenkins
1. Download the WAR from `http://mirrors.jenkins.io/war-stable/latest/jenkins.war`;
```
# cd /opt
# wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
```

2. Jenkins saves configuration/jobs file under `/home/USER/.jenkins` directory by default. To choose a different location, define `JENKINS_HOME` in path;
```
# mkdir /opt/jenkins
# echo 'export JENKINS_HOME=/opt/jenkins/' | sudo tee -a /etc/profile
source /etc/profile
```

3. Login to Tomcat Web Application Manager (http://localhost:8080/manager/html) with `tomcat` user that we have configured in `tomcat-users.xml` file.

4. Go to the Deploy section and complete as following;
```
Context Path: /jenkins
WAR or Directory URL: /opt/jenkins.war
```
Click on Deploy.

5. Check if Jenkins has started running by looking at the `Running` column of the Application lists.

6. Go to `http://localhost:8080/jenkins` and it will ask to `Unlock Jenkins`. Copy the content and run it in terminal;
```
# cat /USER/.jenkins/secrets/initialAdminPassword
  98073dfbfabe4cc8961b1225a171f8db
```
Paste the `secret` to the browser and press Continue.

7. The next page will ask you to `Install plugins`. You can either choose `Install Suggested Plugins` or you can choose `Select plugins to Install`. You can remove/add new plugins later from GUI by visiting `Manage Jenkins > Manage Plugins`.
8. After plugin installation is complete, it will ask you to create a new admin account. Create the account and click Continue.
9. In the `Instance Configuration` page, it will ask for the Jenkins URL. If you prefer to have FQDN name (https://example.com/jenkins), set it here.

---

**Configuring Publish Over SSH plugin**

This plugin can be used to push code from Jenkins server to a remote application server using SSH protocol. For example, if you have your code hosted in a Jenkins server and you want to push the code automatically to an application servers, (appserver1) this plugin can do it for you. To install the plugin and configure the plugin for remote application server, do the following;

1. Install the plugin
Go to Manage Jenkins > Manage Plugins and search for `publish orver ssh` and then install the plugin

2. Create ssh key in the Jenkins server;
```
# ssh-keygen
```

3. Copy the public key to an application server where you are hosting your application. If `appserver1` is the application server, then we will send the public key of Jenkins user in Jenkins server to sysA Jenkins user’s `./ssh` folder.
```
# ssh-copy-id -i id_rsa.pub jenkins@appserver1 
```

4. Login to `appserver1` using Jenkins user and check if it has the `authorized_keys` file;
```
# ls -la /usr/Jenkins/.ssh/
total 4
-rw------- 1 jenkins jenkins 397 Oct  1 15:53 authorized_keys
```

5. Login to Jenkins server with jenkins user and check if `appserver1` info is listed in `known_hosts` file;
```
cat /usr/Jenkins/.ssh/known_hosts | grep system
```

6. Login to Jenkins GUI, and go to `Manage Jenkins > Configure System`. In the Publish over SSH section, add the following;
```
Path to key=/usr/Jenkins/.ssh/id_rsa
SSH Servers
Name=system
Hostname=system.example.com
Username=Jenkins
```
Click on Test Configuration and it should yield `Success`. If not, verify that `jenkins` user of Jenkins server can ssh to system without password.

---
