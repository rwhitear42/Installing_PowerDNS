# Installing PowerDNS on CentOS 7

Perform a minimal installation of CentOS 7.

Login and sudo to root.

```sudo -i```

install epel.

```yum install epel-release.noarch ```

```
Running transaction
  Installing : epel-release-7-11.noarch                                                                                                    1/1
  Verifying  : epel-release-7-11.noarch                                                                                                    1/1

Installed:
  epel-release.noarch 0:7-11

Complete!
```

Install mariadb

```yum -y install mariadb-server mariadb```

**Note:** The URLs in the epel repo specify https connectivity. This may be a problem if your Internet access is via a secure proxy. In this case, either add the proxy to your yum.conf file or modify the /etc/yum.repos.d/epel.repo file and convert to http access.

```yum -y install mariadb-server mariadb```

```
Dependency Updated:
  mariadb-libs.x86_64 1:5.5.60-1.el7_5

Complete!
```

Start and persist the mariadb services

```systemctl enable mariadb.service```
```systemctl start mariadb.service```

Check the status of the services

```systemctl status mariadb.service```

```
mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-10-25 15:09:34 BST; 34s ago
```

Secure and set a password for the mariadb service

```mysql_secure_installation```

```
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): **ENTER**

OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] **Y**

New password: **PWORD**
Re-enter new password: **PWORD**
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] **Y**
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] **n**
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] **Y**
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] **Y**
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```


Install PowerDNS

```yum -y install pdns pdns-backend-mysql```

```
Installed:
  pdns.x86_64 0:3.4.11-4.el7                                      pdns-backend-mysql.x86_64 0:3.4.11-4.el7

Dependency Installed:
  boost-program-options.x86_64 0:1.53.0-27.el7        boost-serialization.x86_64 0:1.53.0-27.el7        cryptopp.x86_64 0:5.6.2-10.el7
  lmdb-libs.x86_64 0:0.9.22-2.el7

Complete!
```

Create a database for PowerDNS

```mysql -u root -p```  (Password created when securing mariadb).
```CREATE DATABASE powerdns;```

Create a user called powerdns

```GRANT ALL ON powerdns.* TO 'powerdns'@'localhost' IDENTIFIED BY 'C15co123';```
```GRANT ALL ON powerdns.* TO 'powerdns'@'centos7.localdomain' IDENTIFIED BY 'C15co123';```
```FLUSH PRIVILEGES;```


Create DB tables used by powerdns

```USE powerdns;```

```
CREATE TABLE domains (
id INT auto_increment,
name VARCHAR(255) NOT NULL,
master VARCHAR(128) DEFAULT NULL,
last_check INT DEFAULT NULL,
type VARCHAR(6) NOT NULL,
notified_serial INT DEFAULT NULL,
account VARCHAR(40) DEFAULT NULL,
primary key (id)
);
```

```CREATE UNIQUE INDEX name_index ON domains(name);```

```
CREATE TABLE records (
id INT auto_increment,
domain_id INT DEFAULT NULL,
name VARCHAR(255) DEFAULT NULL,
type VARCHAR(6) DEFAULT NULL,
content VARCHAR(255) DEFAULT NULL,
ttl INT DEFAULT NULL,
prio INT DEFAULT NULL,
change_date INT DEFAULT NULL,
primary key(id)
);
```

```CREATE INDEX rec_name_index ON records(name);```
```CREATE INDEX nametype_index ON records(name,type);```
```CREATE INDEX domain_id ON records(domain_id);```

```
CREATE TABLE supermasters (
ip VARCHAR(25) NOT NULL,
nameserver VARCHAR(255) NOT NULL,
account VARCHAR(40) DEFAULT NULL
);
```

```
MariaDB [powerdns]> quit;
Bye
```

```vi /etc/pdns/pdns.conf```

Find the section that starts with:

```
#################################
# launch        Which backends to launch and order to query them in
#
# launch=
```

...and add:

```
launch=gmysql
gmysql-host=localhost
gmysql-user=powerdns
gmysql-password=C15co123
gmysql-dbname=powerdns
```

Section should look like this:

```
#################################
# launch        Which backends to launch and order to query them in
#
# launch=
launch=gmysql
gmysql-host=localhost
gmysql-user=powerdns
gmysql-password=C15co123
gmysql-dbname=powerdns
```

Start and persist the pdns services

```systemctl enable pdns.service```
```systemctl start pdns.service```

...and check the status

```systemctl status pdns.service```



```
[root@powerdns ~]# systemctl status pdns.service
● pdns.service - PowerDNS Authoritative Server
   Loaded: loaded (/usr/lib/systemd/system/pdns.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-10-25 15:31:38 BST; 8s ago
  Process: 11238 ExecStart=/usr/sbin/pdns_server --daemon (code=exited, status=0/SUCCESS)
 Main PID: 11239 (pdns_server)
   CGroup: /system.slice/pdns.service
           └─11239 /usr/sbin/pdns_server --daemon

Oct 25 15:31:38 powerdns.uktme.cisco.com systemd[1]: Started PowerDNS Authoritative Server.
Oct 25 15:31:38 powerdns.uktme.cisco.com pdns[11239]: Listening on controlsocket in '/var/run/pdns.controlsocket'
Oct 25 15:31:38 powerdns.uktme.cisco.com pdns[11239]: UDP server bound to 0.0.0.0:53
Oct 25 15:31:38 powerdns.uktme.cisco.com pdns[11239]: TCP server bound to 0.0.0.0:53
Oct 25 15:31:38 powerdns.uktme.cisco.com pdns[11239]: PowerDNS Authoritative Server 3.4.11 (jenkins@autotest.powerdns.com) (C) 2001-20...COM BV
Oct 25 15:31:38 powerdns.uktme.cisco.com pdns[11239]: Using 64-bits mode. Built on 20180201163352 by mockbuild@buildhw-08.phx2.fedorap...5-16).
Oct 25 15:31:38 powerdns.uktme.cisco.com pdns[11239]: PowerDNS comes with ABSOLUTELY NO WARRANTY. This is free software, and you are w...ion 2.
Oct 25 15:31:38 powerdns.uktme.cisco.com pdns[11239]: Creating backend connection for TCP
Oct 25 15:31:38 powerdns.uktme.cisco.com pdns[11239]: About to create 3 backend threads for UDP
Oct 25 15:31:38 powerdns.uktme.cisco.com pdns[11239]: Done launching threads, ready to distribute questions
Hint: Some lines were ellipsized, use -l to show in full.
```


Install PowerAdmin


```yum install httpd php php-devel php-gd php-imap php-ldap php-mysql php-odbc php-pear php-xml php-xmlrpc php-mbstring php-mcrypt php-mhash gettext```

```yum -y install php-pear-DB php-pear-MDB2-Driver-mysql ```

Start and persist Apache

```systemctl enable httpd.service```
```systemctl start httpd.service```

...and check the status

```systemctl status httpd.service```


Install wget

```yum install wget```


Download PowerAdmin

```cd /var/www/html/```
```wget http://downloads.sourceforge.net/project/poweradmin/poweradmin-2.1.7.tgz```
```tar xfv poweradmin-2.1.7.tgz```

Open the ports on the FW

```
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload
```

http://<IP of this server>/poweradmin-2.1.7/intall/


Some GUI stuff...


Command line:

```mysql -u root -p```

```
GRANT SELECT, INSERT, UPDATE, DELETE
ON powerdns.*
TO 'poweruser'@'localhost'
IDENTIFIED BY 'C15co123';
```

```
MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]>
MariaDB [(none)]> quit;
Bye
```

GUI proceed to step 6

Terminal:

```cd poweradmin-2.1.7/inc```


```vi config.inc.php```

```
<?php

$db_host        = 'localhost';
$db_user        = 'poweruser';
$db_pass        = 'C15co123';
$db_name        = 'powerdns';
$db_type        = 'mysql';
$db_layer        = 'PDO';

$session_key        = 'VJgPt7iXAqdb8rTCRUfzRz2gfDWtfDi3CZRr-dZa$^&Cqv';

$iface_lang        = 'en_EN';

$dns_hostmaster        = 'centos7.localhost';
$dns_ns1        = 'ns1.uktme.cisco.com';
$dns_ns2        = 'ns2.uktme.cisco.com';
```


```cd ..```
```[root@powerdns poweradmin-2.1.7]# cp install/htaccess.dist .htaccess```


```[root@powerdns poweradmin-2.1.7]# rm -rf ./install/```

```http://<This server>/poweradmin-2.1.7/```
