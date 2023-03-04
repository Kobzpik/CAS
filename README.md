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
 1. adding the firewall limitation with the Port 389 for the non-secure association and Port 636 unique to the secure port connection to allow remote clients to query openldap server.
 
```
  firewall-cmd --permanent --add-port=389/TCP
```
```
  firewall-cmd --permanent --add-port=636/TCP
```
```
  firewall-cmd –reload
```
 

 2. install required dependencies and build tools
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
 
 3. Install the Symas OpenLDAP Package
 ```
  wget -q https://repo.symas.com/configs/SOFL/rhel8/sofl.repo -O /etc/yum.repos.d/sofl.repo
```
```
 yum install symas-openldap-clients symas-openldap-servers –y
```
 4. Restart the slapd service
 ```
systemctl start slapd
```
```
systemctl enable slapd
```
 5.  New certificate needs to generated to valid for 365 days
```
openssl req -new -x509 -nodes -out /etc/openldap/certs/cert.pem -keyout /etc/openldap/certs/priv.pem -days 365
```
We need a properly formatted certificate to communicate as LDAPS. This certificate lets a OpenLDAP service listen for and automatically accept SSL connections. The server certificate is used for authenticating the OpenLDAP server to the client during the LDAPS setup and for enabling the SSL communication tunnel between the client and the server.
 
 6. Change the ownership and permission of certificate
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

 7. So Check the service if it’s running.
```
netstat -lt | grep ldap
```
Check the service using netstat to ensure services are properly running
