# Central Authentication Server on Red Hat
## What is the central authentication server
Central authentication server is particular type of server that a user to allow access to multiple servers using the same authentication service. This server very helpful to manage multiple servers.

We Need to configure two servers as server machines and client machines to develop a central authentication server(CAS).

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
 
