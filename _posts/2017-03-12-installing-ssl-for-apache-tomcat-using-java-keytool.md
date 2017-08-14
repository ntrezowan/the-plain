---
title: "Installing SSL for Apache Tomcat using Java keytool"
---

### A. Creating new keystore to install a new certificate
In this example, we are assuming that Java and Apache Tomcat are installed to their default location.  
Java Keytool location: `C:\Program Files\Java\jre\bin\`  
Tomcat config location: `C:\Program Files\Apache Tomcat\conf\`  
Tomcat keystore location: `C:\Program Files\Apache Tomcat\conf\SSL\`

*__Steps:__*
1. Open CMD as admin and create new keystore (SHA-2) in the Tomcat keystore folder;  
```
C:\Program Files\Apache Tomcat\conf\SSL\>C:\Program Files\Java\jre\bin\keytool -genkey -alias example.com -keyalg RSA -keystore keystore-example.com.jks -keysize 2048  
Enter keystore password:
Re-enter new password:
What is your first and last name?
[Unknown]:  example.com
What is the name of your organizational unit?
[Unknown]:  Information Technology
What is the name of your organization?
[Unknown]:  Internet Assigned Numbers Authority
What is the name of your City or Locality?
[Unknown]:  Los Angeles
What is the name of your State or Province?
[Unknown]:  California
What is the two-letter country code for this unit?
[Unknown]:  US
Is CN=example.com, OU=Information Technology, O=Internet Assigned Numbers Authority, L=Los Angeles, ST=California, C=US correct?
  [no]:  yes
Enter key password for <example.com>
        (RETURN if same as keystore password):
Re-enter new password:
```
The password you have used here will be used every time you try to open/modify this newly created keystore file.

2. Generate a new CSR for `example.com` domain;
```
C:\Program Files\Apache Tomcat\conf\SSL\>C:\Program Files\Java\jre\bin\keytool -certreq -alias example.com -keystore keystore-example.com.jks -file C:\Users\Administrator\Desktop\example.com.csr
Enter keystore password:
```

3. Use `example.com.csr` to get a new certificate From a CA. Save intermediate certificate as `intermediate.cer` and site certificate as `example.com.p7b`. Copy both to `C:\Users\Administrator\Desktop\` folder on the server.

4. Import intermediate certificate to the keystore;
```
C:\Program Files\Apache Tomcat\conf\SSL\>C:\Program Files\Java\jre\bin\keytool -import -trustcacerts -alias intermediate -file C:\Users\Administrator\Desktop\intermediate.cer -keystore keystore-example.com.jks
Enter keystore password:
Certificate was added to keystore
```

5. Verify that intermediate certificate is imported correctly;
```
C:\Program Files\Apache Tomcat\conf\SSL\>C:\Program Files\Java\jre\bin\keytool -list -v -alias intermediate -keystore keystore-example.com.jks
```


6. Import site certificate (example.com) to the keystore;
```
C:\Program Files\Apache Tomcat\conf\SSL\>C:\Program Files\Java\jre\bin\keytool -import -alias example.com -trustcacerts -file C:\Users\Administrator\Desktop\example.com.p7b -keystore keystore-example.com.jks
Enter keystore password:
Certificate reply was installed in keystore
```


7. Verify that site certificate is imported correctly;
```
C:\Program Files\Apache Tomcat\conf\SSL\>C:\Program Files\Java\jre\bin\keytool -list -v -alias example.com -keystore keystore-example.com.jks
```


8. Open `server.xml` file using Notepad located in `C:\Program Files\Apache Tomcat\conf\` and look for `keystoreFile` string. Modify it to the following;
```
keystoreFile="conf/SSL/keystore-example.com.jks"
```

9. Restart Apache Tomcat service from Windows Services.

10. Verify the changes by visiting hosted site's certificate.

___

### B. Renewing license for an existing keystore


1. Take a backup of the current site certificate;
```
C:\Program Files\Apache Tomcat\conf\SSL\>C:\Program Files\Java\jre\bin\keytool -export -alias example.com -file C:\Users\Administrator\Desktop\example.com-old.crt -keystore keystore-example.com.jks
```
Also take a backup of the `keystore-example.com.jks` keystore file.

2. Renew the certificate from the same CA and save it as `example.com.p7b` format. Copy the file to `C:\Users\Administrator\Desktop\` folder in the server;
```
C:\Program Files\Apache Tomcat\conf\SSL\>C:\Program Files\Java\jre\bin\keytool -import -alias example.com -trustcacerts -file C:\Users\Administrator\Desktop\example.com.p7b -keystore keystore-example.com.jks
Enter keystore password:
Certificate reply was installed in keystore
```

3. Verify that site certificate is imported correctly;
```
C:\Program Files\Apache Tomcat\conf\SSL\>C:\Program Files\Java\jre\bin\keytool -list -v -alias example.com -keystore keystore-example.com.jks
```

4. Restart Apache Tomcat service from Windows Services.
5. Verify the changes by visiting hosted site's certificate.

___


### C. Installing root certificate


Obtain the root certificate from the CA and then we need to import it to the `cacerts` keystore file located in `C:\Program Files\Java\jre\lib\security\` folder.

1. To install a root CA certificate, obtain the root certificate and save it as root.cer. Copy the file to `C:\Users\Administrator\Desktop\` folder in the server. Run the following command to import the certificate;
```
C:\Program Files\Java\jre\lib\security\>C:\Program Files\Java\jre\bin\keytool -import -trustcacerts -alias rootca -file C:\Users\Administrator\Desktop\root.cer -keystore cacerts
Enter keystore password:
```
Here the `cacerts` keystore password is `changeit` (Default).

2. To view all the root certificate, run the following command;
```
C:\Program Files\Java\jre\lib\security\>C:\Program Files\Java\jre\bin\keytool -list -v -keystore cacerts
Enter keystore password:
```