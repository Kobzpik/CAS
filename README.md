# Central Authentication Server on Red Hat
## What is the central authentication server
Central authentication server is particular type of server that a user to allow access to multiple servers using the same authentication service. This server very helpful to manage multiple servers.

## Develop the central authentication server
Among the various options we can choose OpenLDAP as a best solution to develop a central authentication server on Redhat.
We Need to configure two servers as server machines and client machines to develop a central authentication server(CAS).

![image](https://github.com/Kobzpik/CAS/blob/main/image.jpg)

OS – RHEL 8.6                                                                 
Hostname – localhost
IP address – 192.168.56.102  

OS – RHEL 8.6
Hostname – ldap-client
IP address – 192.168.56.103

In addition, centos 8 and Rocky Linux can be used as OS to develop central authentication server following follow steps.

## Prerequisites 
* Both machine should have Rhel 8.6 install properly.
* Make sure both servers are accessible through ssh.
* Local yum repository should be configured on both machine.
 
 ## Server machine configuration
 **1. adding the firewall limitation with the Port 389 for the non-secure association and Port 636 unique to the secure port connection to allow remote clients to query openldap server.**
 
```
  firewall-cmd --permanent --add-port=389/TCP
```
```
  firewall-cmd --permanent --add-port=636/TCP
```
```
  firewall-cmd –reload
```
 

 **2. install required dependencies and build tools**
 We need to install some packages. For that we need to check whether this package is available in our machine or not.
 
 ```
  rpm -qa wget vim cyrus-sasl-devel libtool-ltdl-devel openssl-devel libdb-devel make libtool autoconf tar gcc perl perl-devel
```
* tar-1.30-5.el8.x86_64.
* libtool-2.4.6-25.el8.x86_64.
* libdb-devel-5.3.28-42.el8_4.x86_64.
* autoconf-2.69-29.el8.noarch.
* wget-1.19.5-10.el8.x86_64.
* cyrus-sasl-devel-2.1.27-6.el8_5.x86_64.
* gcc-8.5.0-10.el8.x86_64.
* openssl-devel-1.1.1k-6.el8_5.x86_64.
* make-4.2.1-11.el8.x86_64.
* libtool-ltdl-devel-2.4.6-25.el8.x86_64.
* perl-5.26.3-421.el8.x86_64.
* perl-devel-5.26.3-421.el8.x86_64.
 
 If the above output is not available, the package should be installed.
 ```
  dnf install wget vim cyrus-sasl-devel libtool-ltdl-devel openssl-devel libdb-devel make libtool autoconf tar gcc perl perl-devel –y
```
 
 **3. Install the Symas OpenLDAP Package**
 ```
  wget -q https://repo.symas.com/configs/SOFL/rhel8/sofl.repo -O /etc/yum.repos.d/sofl.repo
```
```
 yum install symas-openldap-clients symas-openldap-servers –y
```
 **4. Restart the slapd service**
 ```
systemctl start slapd
```
```
systemctl enable slapd
```
 **5.  New certificate needs to generated to valid for 365 days**
```
openssl req -new -x509 -nodes -out /etc/openldap/certs/cert.pem -keyout /etc/openldap/certs/priv.pem -days 365
```
We need a properly formatted certificate to communicate as LDAPS. This certificate lets a OpenLDAP service listen for and automatically accept SSL connections. The server certificate is used for authenticating the OpenLDAP server to the client during the LDAPS setup and for enabling the SSL communication tunnel between the client and the server.
 
 **6. Change the ownership and permission of certificate**
```
cd /etc/openldap/certs
```
```
chown ldap:ldap *
```
```
chmod 600 priv.pem
```
we need to change ownership and permission of the certificate file for increase certificate file confidentiality.

 **7. So Check the service if it’s running.**
```
netstat -lt | grep ldap
```
Check the service using netstat to ensure services are properly running

 **8. Setup root password**
 ```
Slappasswd
```
```
$ New password:
$ Re-enter new password:
$ {SSHA}xxxxxxxxxxxxxxxxxxxxxx
```

Here too you need to copy and save password key on the notepad for admin user configuration.
 
 **9. Replace olcSuffix and olcRootDN attribute**
 
 To complete this task we need a reference file for that we have to create .ldif file. For that let's create config,ldif file.

 $ vi config.ldif
```
dn: olcDatabase={2}mdb,cn=config 
changetype: modify 
replace: olcSuffix 
olcSuffix: dc=ldapmaster,dc=kobzserver,dc=com

dn: olcDatabase={2}mdb,cn=config 
changetype: modify 
replace: olcRootDN olcRootDN: cn=admin,dc=ldapmaster,dc=kobzserver,dc=com
```
Then run the following command
```
ldapmodify -Y EXTERNAL -H ldapi:/// -f config.ldif
```
The first line identifies the main entry in the LDAP that we are going to change. Just a moment ago, we saw the parameter olcSuffix inside the /etc/openldap/slapd.d/cn=config/olcDatabase={2}mdb.ldif file. In this file, the dn (distinguished name) attribute is dn: olcDatabase={2}mdb, and as the file is inside the config folder, the full dn attribute isdn: olcDatabase={2}mdb,cn=config.
Also Usually olcSuffix is choosen to reflect the domain name of the slapd running host. For example, if the server runs on a host thats domain name is ldapmaster.kobzserver.com, the olcSuffix attribute is set to dc=ldapmaster,dc= kobzserver,dc=com
To change the suffix of your database, you first have to modify the olcSuffix and olcRootDN attributes to match your domain name.

 **10.Add olcRootPW attribute**
 
 $vi config1.ldif
 
 ```
 dn: olcDatabase={2}mdb,cn=config 
 changeType: modify 
 add: olcRootPW 
 olcRootPW: {SSHA}K93VG/aHvWNSYAuBS7G+mj54nxlJfMj8
 ```
 ```
 ldapmodify -Y EXTERNAL -H ldapi:/// -f config1.ldif
 ```
 **11.We also have to allow access to the LDAP database to the admin user**

 $ vi config2.ldif
  ```
 dn: olcDatabase={1}monitor,cn=config
 changetype: modify 
 replace: olcAccess 
 olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external, cn=auth" read by dn.base="cn=admin,dc=ldapmaster,dc=kobzserver,dc=com" read by * none
 ```
 ```
 ldapmodify -Y EXTERNAL -H ldapi:/// -f config2.ldif
 ```
 We also have to allow access to the LDAP database to the admin user we just specified before (cn=admin,dc=ldapmaster,dc=kobzserver,dc=com).

 **12.Adding domain details**

 $ vi root.ldif
 ```
 dn: dc=ldapmaster,dc=kobzserver,dc=com 
 objectClass: dcObject 
 objectClass: organization 
 dc: ldapmaster 
 o: lapmaster
 ```
 ```
 ldapadd -f root.ldif -D cn=admin,dc=ldapmaster,dc=kobzserver,dc=com -w [pwd…]
 ```
 Note - You should replace your password instead [pwd…]

 In this, we have to manually create an entry for dc=ldapmaster,dc=kobzserver,dc=com in our LDAP server. We specify a series of attributes, such as distinguished name (dn), domain component (dc), and organization (o).

  **13.We can check whether the entry was created successfully by using the ldapsearch command.**
  ```
  ldapsearch -x -b dc=ldapmaster,dc=kobzserver,dc=com
  ```
  **14.Adding an Organizational Unit.**

  $ vi unit.ldif

  ```
  dn: ou=users,dc=ldapmaster,dc=kobzserver,dc=com 
  objectClass: organizationalUnit 
  ou: users
  ```
  ```
  ldapadd -f unit.ldif -D cn=admin,dc=ldapmaster,dc=kobzserver,dc=com -w [pwd…]
  ```
  Note - You should replace your password instead [pwd…]

  we have to an organizational unit (OU) called users in which to store all LDAP users.

  **15.For adding users into server we need to install cosine , inetorgperson and nis core schemes.**
  ```
  ldapadd -Y EXTERNAL -H ldapi:// -f /etc/openldap/schema/cosine.ldif 
  ldapadd -Y EXTERNAL -H ldapi:// -f /etc/openldap/schema/inetorgperson.ldif 
  ldapadd -Y EXTERNAL -H ldapi:// -f /etc/openldap/schema/nis.ldif
  ```
  then add the user

  $ vi user.ldif
  ```
  dn: uid=prabath,ou=users,dc=ldapmaster,dc=kobzserver,dc=com 
  objectClass: top 
  objectClass: account 
  objectClass: posixAccount 
  objectClass: shadowAccount 
  cn: prabath #Enter user name here
  uid: prabath #Enter user id here
  uidNumber: 1009 #Enter user id number here
  gidNumber: 1009 #Enter user group id number here
  homeDirectory: /home/prabath 
  loginShell: /bin/bash 
  gecos: prabath
  userPassword: 34195033 # Enter user password here
  shadowLastChange: 12 
  shadowMax: 12 
  shadowWarning: 2 
  ```
  ```
  ldapadd -f user1.ldif -x -D cn=admin,dc=ldapmaster,dc=kobzserver,dc=com -w [pwd…]
  ```
  Note - You should replace your instead [pwd…]

  In here we specify a series of attributes, such as cn (common name).

 **16.Adding group**
 
 $ vi group.ldif
 ```
 dn: cn=manager,ou=users,dc=ldapmaster,dc=kobzserver,dc=com 
 cn: manager 
 objectClass: groupOfNames 
 member: cn=manager,ou=users,dc=ldapmaster,dc=kobzserver,dc=com
 ```
 ```
 ldapadd -f group.ldif -D cn=admin,dc=ldapmaster,dc=kobzserver,dc=com -w [pwd…]
 ```
 **17.Adding password policy**
 
 * Install ppolocy module
 * Create password policy OU container
 $ vi pwpolicy-ou.ldif
 
 ```
 dn: ou=pwpolicy,dc=ldapmaster,dc=kobzserver,dc=com 
 objectClass: organizationalUnit 
 objectClass: top 
 ou: pwpolicy
 ```
 ```
 ldapadd -f pwpolicy-ou.ldif -D cn=admin,dc=ldapmaster,dc=kobzserver,dc=com -w [pwd…]
 ```
 * Create OpenLDAP Password Policy Overlay DN
 
 $ vi pwpolicyoverlay.ldif
 ```
 dn: olcOverlay=ppolicy,olcDatabase={2}mdb,cn=config 
 objectClass: olcOverlayConfig 
 objectClass: olcPPolicyConfig
 olcOverlay: ppolicy 
 olcPPolicyDefault: cn=default,ou=pwpolicy,dc=ldapmaster,dc=kobzserver,dc=com 
 olcPPolicyHashCleartext: TRUE
 ```
 ```
 ldapadd -f pwpolicyoverlay.ldif -D cn=admin,dc=ldapmaster,dc=kobzserver,dc=com -w [pwd…]
 ```
 * Create the password policy
 
 $ vi passwordpolicy.ldif
 
 ```
 dn: cn=default,ou=pwpolicy,dc=ldapmaster,dc=kobzserver,dc=com 
 objectClass: person 
 objectClass: pwdPolicyChecker 
 objectClass: pwdPolicy 
 cn: pwpolicy 
 sn: pwpolicy 
 pwdAttribute: userPassword 
 pwdMinAge: 1 
 pwdMaxAge: 2 
 pwdMinLength: 8 
 pwdExpireWarning: 1
 pwdGraceAuthNLimit: 5
 pwdLockoutDuration: 0
 pwdMaxFailure: 3
 pwdFailureCountInterval: 0
 pwdReset: TRUE
 pwdMustChange: TRUE
 pwdSafeModify: FALSE
 pwdAllowUserChange: FALSE
 dn: cn=default,ou=pwpolicy,dc=ldapmaster,dc=kobzserver,dc=com 
 objectClass: person 
 objectClass: pwdPolicyChecker
 objectClass: pwdPolicy 
 cn: pwpolicy 
 sn: pwpolicy 
 pwdAttribute: userPassword 
 pwdMinAge: 1 
 pwdMaxAge: 2 
 pwdMinLength: 8 
 pwdExpireWarning: 1 
 pwdGraceAuthNLimit: 5 
 pwdLockoutDuration: 0 
 pwdMaxFailure: 3 
 pwdFailureCountInterval: 0 
 pwdReset: TRUE
 pwdMustChange: TRUE 
 pwdSafeModify: FALSE 
 pwdAllowUserChange: FALSE
 ```
 ```
 ldapadd -f passwordpolicy.ldif -D cn=admin,dc=ldapmaster,dc=kobzserver,dc=com -w [pwd…]
 ```
 
