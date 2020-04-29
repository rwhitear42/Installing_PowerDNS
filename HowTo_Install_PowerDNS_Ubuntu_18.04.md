# Installing PowerDNS on Ubuntu 18.04


Using instructions from:
* https://computingforgeeks.com/install-powerdns-and-powerdns-admin-on-ubuntu-18-04-debian-9-mariadb-backend/
* https://computingforgeeks.com/install-mariadb-10-on-ubuntu-18-04-and-centos-7/
* https://docs.powerdns.com/recursor/getting-started.html

Websites used in understanding how all this works:
* https://github.com/ngoduykhanh/PowerDNS-Admin
* https://stackoverflow.com/questions/32568225/powerdns-works-only-on-local
* https://bbs.archlinux.org/viewtopic.php?id=153781
* https://doc.powerdns.com/recursor/getting-started.html#installation
* https://doc.powerdns.com/authoritative/guides/recursion.html
* https://doc.powerdns.com/md/authoritative/howtos/#basic-setup-configuring-database-connectivity
* https://doc.powerdns.com/authoritative/settings.html
* https://blog.jonaharagon.com/setting-up-powerdns-with-mysql-replication-on-ubuntu-18-04/
* https://github.com/ngoduykhanh/PowerDNS-Admin/wiki/Running-PowerDNS-Admin-with-Systemd,-Gunicorn--and--Nginx
* https://github.com/ngoduykhanh/PowerDNS-Admin/wiki/Running-PowerDNS-Admin-on-Ubuntu-or-Debian
* https://computingforgeeks.com/install-powerdns-and-powerdns-admin-on-ubuntu-18-04-debian-9-mariadb-backend/



#### Remove existing version of mariadb (If it exists)

**rwhitear@dns-01:~$** sudo apt-get remove mariadb-server

```sh
[sudo] password for rwhitear:
Reading package lists... Done
Building dependency tree
Reading state information... Done
Package 'mariadb-server' is not installed, so not removed
The following packages were automatically installed and are no longer required:
  libllvm7 linux-headers-4.18.0-15 linux-headers-4.18.0-15-generic linux-image-4.18.0-15-generic linux-modules-4.18.0-15-generic linux-modules-extra-4.18.0-15-generic
Use 'sudo apt autoremove' to remove them.
0 to upgrade, 0 to newly install, 0 to remove and 0 not to upgrade.
```

**rwhitear@dns-01:~$** sudo apt-get install software-properties-common

```sh
Reading package lists... Done
Building dependency tree
Reading state information... Done
software-properties-common is already the newest version (0.96.24.32.12).
The following packages were automatically installed and are no longer required:
  libllvm7 linux-headers-4.18.0-15 linux-headers-4.18.0-15-generic linux-image-4.18.0-15-generic linux-modules-4.18.0-15-generic linux-modules-extra-4.18.0-15-generic
Use 'sudo apt autoremove' to remove them.
0 to upgrade, 0 to newly install, 0 to remove and 0 not to upgrade.
```

**rwhitear@dns-01:~$** sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8

```sh
Executing: /tmp/apt-key-gpghome.nFb2P0J0QR/gpg.1.sh --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
gpg: key F1656F24C74CD1D8: 6 signatures not checked due to missing keys
gpg: key F1656F24C74CD1D8: public key "MariaDB Signing Key <signing-key@mariadb.org>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

**rwhitear@dns-01:~$** sudo add-apt-repository "deb [arch=amd64,arm64,ppc64el] http://mariadb.mirror.liquidtelecom.com/repo/10.4/ubuntu $(lsb_release -cs) main"

```sh
Ign:1 http://dl.google.com/linux/chrome/deb stable InRelease
Hit:2 http://gb.archive.ubuntu.com/ubuntu bionic InRelease
Get:3 http://dl.google.com/linux/chrome/deb stable Release [943 B]
Get:4 http://gb.archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]
Hit:5 https://download.docker.com/linux/ubuntu bionic InRelease
Get:6 http://dl.google.com/linux/chrome/deb stable Release.gpg [819 B]
Get:7 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]
Get:8 http://gb.archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]
...
```

**rwhitear@dns-01:~$** sudo apt update

```sh
Hit:1 http://gb.archive.ubuntu.com/ubuntu bionic InRelease
Get:2 http://gb.archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]
Hit:3 https://download.docker.com/linux/ubuntu bionic InRelease
Ign:4 http://dl.google.com/linux/chrome/deb stable InRelease
Get:5 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]
Get:6 http://dl.google.com/linux/chrome/deb stable Release [943 B]
...
```

**rwhitear@dns-01:~$** sudo apt -y install mariadb-server mariadb-client

```sh
Preparing to unpack .../1-libmysqlclient20_5.7.29-0ubuntu0.18.04.1_amd64.deb ...
Unpacking libmysqlclient20:amd64 (5.7.29-0ubuntu0.18.04.1) ...
Selecting previously unselected package libdbd-mysql-perl.
Preparing to unpack .../2-libdbd-mysql-perl_4.046-1_amd64.deb ...
Unpacking libdbd-mysql-perl (4.046-1) ...
...
Setting up mariadb-client (1:10.4.12+maria~bionic) ...
Setting up mariadb-server (1:10.4.12+maria~bionic) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Processing triggers for libc-bin (2.27-3ubuntu1) ...
```

**rwhitear@dns-01:~$** sudo mysql_secure_installation

```sh
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] n
 ... skipping.

You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] y
New password: <NEW MARIADB ROOT PASSWORD>
Re-enter new password: <NEW MARIADB ROOT PASSWORD>
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n]
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n]
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n]
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n]
 ... Success!

Cleaning up...

All done!  If you`ve completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

Ensure that mariadb has installed correctly.

**rwhitear@dns-01:~$** mysql -u root -p

```sh
Enter password: <ENTER MARIADB ROOT PASSWORD SET PREVIOUSLY>
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 55
Server version: 10.4.12-MariaDB-1:10.4.12+maria~bionic-log mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SELECT VERSION();
+--------------------------------------------+
| VERSION()                                  |
+--------------------------------------------+
| 10.4.12-MariaDB-1:10.4.12+maria~bionic-log |
+--------------------------------------------+
1 row in set (0.000 sec)

MariaDB [(none)]> exit;
```

**rwhitear@dns-01:~$** mysql -u root -p

```sh
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 56
Server version: 10.4.12-MariaDB-1:10.4.12+maria~bionic-log mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE powerdns;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> GRANT ALL ON powerdns.* TO 'powerdns'@'localhost' \
    -> IDENTIFIED BY 'C!5co123';
Query OK, 0 rows affected (0.004 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> USE powerdns;
Database changed
MariaDB [powerdns]>
```

Paste the following SQL statements into mariadb:

```sh
CREATE TABLE domains (
  id                    INT AUTO_INCREMENT,
  name                  VARCHAR(255) NOT NULL,
  master                VARCHAR(128) DEFAULT NULL,
  last_check            INT DEFAULT NULL,
  type                  VARCHAR(6) NOT NULL,
  notified_serial       INT UNSIGNED DEFAULT NULL,
  account               VARCHAR(40) CHARACTER SET 'utf8' DEFAULT NULL,
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'latin1';

CREATE UNIQUE INDEX name_index ON domains(name);


CREATE TABLE records (
  id                    BIGINT AUTO_INCREMENT,
  domain_id             INT DEFAULT NULL,
  name                  VARCHAR(255) DEFAULT NULL,
  type                  VARCHAR(10) DEFAULT NULL,
  content               VARCHAR(64000) DEFAULT NULL,
  ttl                   INT DEFAULT NULL,
  prio                  INT DEFAULT NULL,
  change_date           INT DEFAULT NULL,
  disabled              TINYINT(1) DEFAULT 0,
  ordername             VARCHAR(255) BINARY DEFAULT NULL,
  auth                  TINYINT(1) DEFAULT 1,
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'latin1';

CREATE INDEX nametype_index ON records(name,type);
CREATE INDEX domain_id ON records(domain_id);
CREATE INDEX ordername ON records (ordername);


CREATE TABLE supermasters (
  ip                    VARCHAR(64) NOT NULL,
  nameserver            VARCHAR(255) NOT NULL,
  account               VARCHAR(40) CHARACTER SET 'utf8' NOT NULL,
  PRIMARY KEY (ip, nameserver)
) Engine=InnoDB CHARACTER SET 'latin1';


CREATE TABLE comments (
  id                    INT AUTO_INCREMENT,
  domain_id             INT NOT NULL,
  name                  VARCHAR(255) NOT NULL,
  type                  VARCHAR(10) NOT NULL,
  modified_at           INT NOT NULL,
  account               VARCHAR(40) CHARACTER SET 'utf8' DEFAULT NULL,
  comment               TEXT CHARACTER SET 'utf8' NOT NULL,
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'latin1';

CREATE INDEX comments_name_type_idx ON comments (name, type);
CREATE INDEX comments_order_idx ON comments (domain_id, modified_at);


CREATE TABLE domainmetadata (
  id                    INT AUTO_INCREMENT,
  domain_id             INT NOT NULL,
  kind                  VARCHAR(32),
  content               TEXT,
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'latin1';

CREATE INDEX domainmetadata_idx ON domainmetadata (domain_id, kind);


CREATE TABLE cryptokeys (
  id                    INT AUTO_INCREMENT,
  domain_id             INT NOT NULL,
  flags                 INT NOT NULL,
  active                BOOL,
  content               TEXT,
  PRIMARY KEY(id)
) Engine=InnoDB CHARACTER SET 'latin1';

CREATE INDEX domainidindex ON cryptokeys(domain_id);


CREATE TABLE tsigkeys (
  id                    INT AUTO_INCREMENT,
  name                  VARCHAR(255),
  algorithm             VARCHAR(50),
  secret                VARCHAR(255),
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'latin1';

CREATE UNIQUE INDEX namealgoindex ON tsigkeys(name, algorithm);
```

```sh
MariaDB [powerdns]> CREATE TABLE records ( \
       id                    BIGINT AUTO_INCREMENT, \
       domain_id             INT DEFAULT NULL, \
       name                  VARCHAR(255) DEFAULT NULL, \
       type                  VARCHAR(10) DEFAULT NULL, \
       content               VARCHAR(64000) DEFAULT NULL, \
       ttl                   INT DEFAULT NULL, \
       prio                  INT DEFAULT NULL, \
       change_date           INT DEFAULT NULL, \
       disabled              TINYINT(1) DEFAULT 0, \
       ordername             VARCHAR(255) BINARY DEFAULT NULL, \
       auth                  TINYINT(1) DEFAULT 1, \
       PRIMARY KEY (id) \
     ) Engine=InnoDB CHARACTER SET 'latin1';
Query OK, 0 rows affected (0.018 sec)

MariaDB [powerdns]>
MariaDB [powerdns]> CREATE INDEX nametype_index ON records(name,type);
Query OK, 0 rows affected (0.019 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [powerdns]> CREATE INDEX domain_id ON records(domain_id);
Query OK, 0 rows affected (0.020 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [powerdns]> CREATE INDEX ordername ON records (ordername);
Query OK, 0 rows affected (0.020 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [powerdns]>
MariaDB [powerdns]>
MariaDB [powerdns]> CREATE TABLE supermasters (
    ->   ip                    VARCHAR(64) NOT NULL,
    ->   nameserver            VARCHAR(255) NOT NULL,
    ->   account               VARCHAR(40) CHARACTER SET 'utf8' NOT NULL,
    ->   PRIMARY KEY (ip, nameserver)
    -> ) Engine=InnoDB CHARACTER SET 'latin1';
Query OK, 0 rows affected (0.019 sec)

MariaDB [powerdns]>
MariaDB [powerdns]>
MariaDB [powerdns]> CREATE TABLE comments (
    ->   id                    INT AUTO_INCREMENT,
    ->   domain_id             INT NOT NULL,
    ->   name                  VARCHAR(255) NOT NULL,
    ->   type                  VARCHAR(10) NOT NULL,
    ->   modified_at           INT NOT NULL,
    ->   account               VARCHAR(40) CHARACTER SET 'utf8' DEFAULT NULL,
    ->   comment               TEXT CHARACTER SET 'utf8' NOT NULL,
    ->   PRIMARY KEY (id)
    -> ) Engine=InnoDB CHARACTER SET 'latin1';
Query OK, 0 rows affected (0.018 sec)

MariaDB [powerdns]>
MariaDB [powerdns]> CREATE INDEX comments_name_type_idx ON comments (name, type);
Query OK, 0 rows affected (0.018 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [powerdns]> CREATE INDEX comments_order_idx ON comments (domain_id, modified_at);
Query OK, 0 rows affected (0.016 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [powerdns]>
MariaDB [powerdns]>
MariaDB [powerdns]> CREATE TABLE domainmetadata (
    ->   id                    INT AUTO_INCREMENT,
    ->   domain_id             INT NOT NULL,
    ->   kind                  VARCHAR(32),
    ->   content               TEXT,
    ->   PRIMARY KEY (id)
    -> ) Engine=InnoDB CHARACTER SET 'latin1';
Query OK, 0 rows affected (0.017 sec)

MariaDB [powerdns]>
MariaDB [powerdns]> CREATE INDEX domainmetadata_idx ON domainmetadata (domain_id, kind);
Query OK, 0 rows affected (0.015 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [powerdns]>
MariaDB [powerdns]>
MariaDB [powerdns]> CREATE TABLE cryptokeys (
    ->   id                    INT AUTO_INCREMENT,
    ->   domain_id             INT NOT NULL,
    ->   flags                 INT NOT NULL,
    ->   active                BOOL,
    ->   content               TEXT,
    ->   PRIMARY KEY(id)
    -> ) Engine=InnoDB CHARACTER SET 'latin1';
Query OK, 0 rows affected (0.019 sec)

MariaDB [powerdns]>
MariaDB [powerdns]> CREATE INDEX domainidindex ON cryptokeys(domain_id);
Query OK, 0 rows affected (0.019 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [powerdns]>
MariaDB [powerdns]>
MariaDB [powerdns]> CREATE TABLE tsigkeys (
    ->   id                    INT AUTO_INCREMENT,
    ->   name                  VARCHAR(255),
    ->   algorithm             VARCHAR(50),
    ->   secret                VARCHAR(255),
    ->   PRIMARY KEY (id)
    -> ) Engine=InnoDB CHARACTER SET 'latin1';
Query OK, 0 rows affected (0.021 sec)

MariaDB [powerdns]>
MariaDB [powerdns]> CREATE UNIQUE INDEX namealgoindex ON tsigkeys(name, algorithm);
Query OK, 0 rows affected (0.020 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [powerdns]>
MariaDB [powerdns]>
MariaDB [powerdns]> show tables;
+--------------------+
| Tables_in_powerdns |
+--------------------+
| comments           |
| cryptokeys         |
| domainmetadata     |
| domains            |
| records            |
| supermasters       |
| tsigkeys           |
+--------------------+
7 rows in set (0.000 sec)

MariaDB [powerdns]> exit;
Bye
```

Ubuntu 18.04 comes with systemd-resolve which you need to disable since it binds to port 53 which will conflict with PowerDNS ports. Run the following commands to disable the resolved service:

**rwhitear@dns-01:~$** sudo systemctl disable systemd-resolved

```sh
Removed /etc/systemd/system/multi-user.target.wants/systemd-resolved.service.
Removed /etc/systemd/system/dbus-org.freedesktop.resolve1.service.
```

**rwhitear@dns-01:~$** sudo systemctl stop systemd-resolved

**rwhitear@dns-01:~$** ls -lh /etc/resolv.conf

```sh
lrwxrwxrwx 1 root root 39 Jul 23  2019 /etc/resolv.conf -> ../run/systemd/resolve/stub-resolv.conf
```

**rwhitear@dns-01:~$** sudo rm /etc/resolv.conf

**rwhitear@dns-01:~$** sudo -i

**root@dns-01:~#** echo "nameserver 8.8.8.8" > /etc/resolv.conf

**root@dns-01:~#**  cat /etc/resolv.conf

```sh
nameserver 8.8.8.8
```

**root@dns-01:~#** exit

```sh
rwhitear@dns-01:~$
```

**rwhitear@dns-01:~$** sudo apt-get update

```sh
Ign:1 http://dl.google.com/linux/chrome/deb stable InRelease
Hit:2 http://gb.archive.ubuntu.com/ubuntu bionic InRelease
Get:3 http://dl.google.com/linux/chrome/deb stable Release [943 B]
Get:4 http://gb.archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]
...
```

**rwhitear@dns-01:~$** sudo apt-get install pdns-server pdns-backend-mysql

```sh
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  libllvm7 linux-headers-4.18.0-15 linux-headers-4.18.0-15-generic linux-image-4.18.0-15-generic linux-modules-4.18.0-15-generic linux-modules-extra-4.18.0-15-generic
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  dbconfig-common dbconfig-mysql libboost-program-options1.65.1 pdns-backend-bind
The following NEW packages will be installed
  dbconfig-common dbconfig-mysql libboost-program-options1.65.1 pdns-backend-bind pdns-backend-mysql pdns-server
...
```
**NOTE: If asked whether to install dbconfig-common, select the \<No> option as we have already installed and configured mariadb for PowerDNS.**

Modify /etc/powerdns/pdns.d/pdns.local.gmysql.conf file with the following:

**rwhitear@dns-01:~$** sudo cat /etc/powerdns/pdns.d/pdns.local.gmysql.conf

```sh
# MySQL Configuration
#
# Launch gmysql backend
launch+=gmysql

# gmysql parameters
gmysql-host=localhost
gmysql-port=3306
gmysql-dbname=powerdns
gmysql-user=powerdns
gmysql-password=<THE PASSWORD YOU CONFIGURED EARLIER>
gmysql-dnssec=yes
# gmysql-socket=
```

**rwhitear@dns-01:~$** sudo systemctl restart pdns

**rwhitear@dns-01:~$** sudo systemctl status pdns

```sh
● pdns.service - PowerDNS Authoritative Server
   Loaded: loaded (/lib/systemd/system/pdns.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2020-03-12 11:13:50 GMT; 10s ago
     Docs: man:pdns_server(1)
           man:pdns_control(1)
           https://doc.powerdns.com
 Main PID: 25790 (pdns_server)
    Tasks: 8 (limit: 4666)
   CGroup: /system.slice/pdns.service
           └─25790 /usr/sbin/pdns_server --guardian=no --daemon=no --disable-syslog --log-timestamp=no --write-pid=no

Mar 12 11:13:50 dns-01 pdns_server[25790]: TCPv6 server bound to [::]:53
Mar 12 11:13:50 dns-01 pdns_server[25790]: PowerDNS Authoritative Server 4.1.1 (C) 2001-2017 PowerDNS.COM BV
Mar 12 11:13:50 dns-01 pdns_server[25790]: Using 64-bits mode. Built using gcc 7.3.0.
Mar 12 11:13:50 dns-01 pdns_server[25790]: PowerDNS comes with ABSOLUTELY NO WARRANTY. This is free software, and you are welcome to redistribute it according to the terms
Mar 12 11:13:50 dns-01 pdns_server[25790]: Creating backend connection for TCP
Mar 12 11:13:50 dns-01 pdns_server[25790]: [bindbackend] Parsing 0 domain(s), will report when done
Mar 12 11:13:50 dns-01 pdns_server[25790]: [bindbackend] Done parsing domains, 0 rejected, 0 new, 0 removed
Mar 12 11:13:50 dns-01 systemd[1]: Started PowerDNS Authoritative Server.
Mar 12 11:13:50 dns-01 pdns_server[25790]: About to create 3 backend threads for UDP
Mar 12 11:13:51 dns-01 pdns_server[25790]: Done launching threads, ready to distribute questions
```

**rwhitear@dns-01:~$** sudo netstat -tap | grep pdns

```sh
tcp        0      0 0.0.0.0:domain          0.0.0.0:*               LISTEN      25790/pdns_server
tcp6       0      0 [::]:domain             [::]:*                  LISTEN      25790/pdns_server
```

``` sh
rwhitear@ns2:~$ sudo vi /etc/powerdns/pdns.conf
```

``` sh
#################################
# local-port    The port on which we listen
#
local-port=5300
```

``` sh
rwhitear@ns1:~$ sudo systemctl restart pdns
rwhitear@ns1:~$ sudo systemctl status pdns
```

``` sh
● pdns.service - PowerDNS Authoritative Server
   Loaded: loaded (/lib/systemd/system/pdns.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2020-04-29 06:46:58 UTC; 8s ago
     Docs: man:pdns_server(1)
           man:pdns_control(1)
           https://doc.powerdns.com
 Main PID: 5832 (pdns_server)
    Tasks: 8 (limit: 4660)
   CGroup: /system.slice/pdns.service
           └─5832 /usr/sbin/pdns_server --guardian=no --daemon=no --disable-syslog --log-timestamp=no --write-pid=no

Apr 29 06:46:58 ns1 pdns_server[5832]: TCPv6 server bound to [::]:5300
Apr 29 06:46:58 ns1 pdns_server[5832]: PowerDNS Authoritative Server 4.1.1 (C) 2001-2017 PowerDNS.COM BV
Apr 29 06:46:58 ns1 pdns_server[5832]: Using 64-bits mode. Built using gcc 7.3.0.
Apr 29 06:46:58 ns1 pdns_server[5832]: PowerDNS comes with ABSOLUTELY NO WARRANTY. This is free software, and you are welcome to redistribute it according to
Apr 29 06:46:58 ns1 pdns_server[5832]: Creating backend connection for TCP
Apr 29 06:46:58 ns1 pdns_server[5832]: [bindbackend] Parsing 0 domain(s), will report when done
Apr 29 06:46:58 ns1 pdns_server[5832]: [bindbackend] Done parsing domains, 0 rejected, 0 new, 0 removed
Apr 29 06:46:58 ns1 systemd[1]: Started PowerDNS Authoritative Server.
Apr 29 06:46:58 ns1 pdns_server[5832]: About to create 3 backend threads for UDP
Apr 29 06:46:58 ns1 pdns_server[5832]: Done launching threads, ready to distribute questions
```

# Install PowerDNS recursor

``` sh
rwhitear@ns2:~$ sudo apt-get update
```

``` sh
rwhitear@ns2:~$ sudo apt-get install pdns-recursor
```

``` sh
rwhitear@ns1:~$ pdns_recursor --no-config --config | grep config-dir
# api-config-dir	Directory where REST API stores config and zones
# api-config-dir=
# config-dir	Location of configuration directory (recursor.conf)
# config-dir=/etc/powerdns
```

``` sh
rwhitear@ns1:~$ sudo vi /etc/powerdns/recursor.conf
```

``` #!/bin/sh
#################################
# allow-from    If set, only allow these comma separated netmasks to recurse
#
allow-from=127.0.0.0/8, 10.0.0.0/8, 100.64.0.0/10, 169.254.0.0/16, 192.168.0.0/16, 172.16.0.0/12, ::1/128, fc00::/7, fe80::/10

#################################
# local-address IP addresses to listen on, separated by spaces or commas. Also accepts ports.
#
local-address=127.0.0.1,10.237.97.134
```

``` sh
rwhitear@ns1:~$ sudo systemctl restart pdns-recursor
rwhitear@ns1:~$ sudo systemctl status pdns-recursor
```

``` sh
● pdns-recursor.service - PowerDNS Recursor
   Loaded: loaded (/lib/systemd/system/pdns-recursor.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2020-04-29 06:48:36 UTC; 6s ago
     Docs: man:pdns_recursor(1)
           man:rec_control(1)
           https://doc.powerdns.com
 Main PID: 5881 (pdns_recursor)
    Tasks: 4 (limit: 4660)
   CGroup: /system.slice/pdns-recursor.service
           └─5881 /usr/sbin/pdns_recursor --daemon=no --write-pid=no --disable-syslog --log-timestamp=no

Apr 29 06:48:36 ns1 pdns_recursor[5881]: Listening for TCP queries on 127.0.0.1:53
Apr 29 06:48:36 ns1 pdns_recursor[5881]: Listening for TCP queries on 10.237.97.134:53
Apr 29 06:48:36 ns1 pdns_recursor[5881]: Set effective group id to 114
Apr 29 06:48:36 ns1 pdns_recursor[5881]: Set effective user id to 112
Apr 29 06:48:36 ns1 pdns_recursor[5881]: Launching 3 threads
Apr 29 06:48:36 ns1 systemd[1]: Started PowerDNS Recursor.
Apr 29 06:48:36 ns1 pdns_recursor[5881]: Done priming cache with root hints
Apr 29 06:48:36 ns1 pdns_recursor[5881]: Done priming cache with root hints
Apr 29 06:48:36 ns1 pdns_recursor[5881]: Enabled 'epoll' multiplexer
Apr 29 06:48:36 ns1 pdns_recursor[5881]: Done priming cache with root hints
```
























Install PowerDNS admin

**rwhitear@dns-01:~$** sudo apt-get install python3-dev

```sh
Reading package lists... Done
Building dependency tree
Reading state information... Done
python3-dev is already the newest version (3.6.7-1~18.04).
python3-dev set to manually installed.
The following packages were automatically installed and are no longer required:
  libllvm7 linux-headers-4.18.0-15 linux-headers-4.18.0-15-generic linux-image-4.18.0-15-generic linux-modules-4.18.0-15-generic linux-modules-extra-4.18.0-15-generic
Use 'sudo apt autoremove' to remove them.
0 to upgrade, 0 to newly install, 0 to remove and 0 not to upgrade.
```


rwhitear@ns1:~$ sudo apt-get install -y libmysqlclient-dev python-mysqldb libsasl2-dev libffi-dev \
libldap2-dev libssl-dev libxml2-dev libxslt1-dev libxmlsec1-dev pkg-config


rwhitear@ns1:~$ sudo curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -

rwhitear@ns1:~$ sudo -i
root@ns1:~# echo "deb https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list
root@ns1:~# exit
logout
rwhitear@ns1:~$


rwhitear@ns1:~$ sudo apt-get update


rwhitear@ns1:~$ sudo apt-get install  yarn


rwhitear@ns1:~$ sudo git clone https://github.com/ngoduykhanh/PowerDNS-Admin.git /opt/web/powerdns-admin


rwhitear@ns1:~$ cd /opt/web/powerdns-admin


rwhitear@ns1:/opt/web/powerdns-admin$ sudo apt-get install python3-venv



rwhitear@ns1:/opt/web/powerdns-admin$ sudo python3 -m venv flask

rwhitear@ns1:/opt/web/powerdns-admin$ sudo -i


root@ns1:~# cd /opt/web/powerdns-admin/


root@ns1:/opt/web/powerdns-admin# . flask/bin/activate
(flask) root@ns1:/opt/web/powerdns-admin#


(flask) root@ns1:/opt/web/powerdns-admin# pip install --upgrade pip


(flask) root@ns1:/opt/web/powerdns-admin# pip install -r requirements.txt


(flask) root@ns1:/opt/web/powerdns-admin# exit
logout
rwhitear@ns1:/opt/web/powerdns-admin$ mysql -u root -p


MariaDB [(none)]> CREATE DATABASE powerdnsadmin;

MariaDB [(none)]> GRANT ALL PRIVILEGES ON powerdnsadmin.* TO 'pdnsadminuser'@'%' IDENTIFIED BY 'C!5co123';

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> quit



rwhitear@ns1:/opt/web/powerdns-admin$ sudo vi ./powerdnsadmin/default_config.py


``` sh
### DATABASE CONFIG
SQLA_DB_USER = 'powerdns'
SQLA_DB_PASSWORD = 'C!5co123'
SQLA_DB_HOST = '127.0.0.1'
SQLA_DB_NAME = 'powerdns'
SQLALCHEMY_TRACK_MODIFICATIONS = True

### DATBASE - MySQL
SQLALCHEMY_DATABASE_URI = 'mysql://'+SQLA_DB_USER+':'+SQLA_DB_PASSWORD+'@'+SQLA_DB_HOST+'/'+SQLA_DB_NAME
```

rwhitear@ns1:/opt/web/powerdns-admin$ sudo -i
root@ns1:~# cd /opt/web/powerdns-admin/
root@ns1:/opt/web/powerdns-admin# . ./flask/bin/activate
(flask) root@ns1:/opt/web/powerdns-admin#

(flask) root@ns1:/opt/web/powerdns-admin# export FLASK_APP=powerdnsadmin/__init__.py

(flask) root@ns1:/opt/web/powerdns-admin# flask db upgrade


``` sh
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 787bdba9e147, Init DB
INFO  [alembic.runtime.migration] Running upgrade 787bdba9e147 -> 59729e468045, Add view column to setting table
INFO  [alembic.runtime.migration] Running upgrade 59729e468045 -> 1274ed462010, Change setting.value data type
INFO  [alembic.runtime.migration] Running upgrade 1274ed462010 -> 4a666113c7bb, Adding Operator Role
INFO  [alembic.runtime.migration] Running upgrade 4a666113c7bb -> 31a4ed468b18, Remove all setting in the DB
INFO  [alembic.runtime.migration] Running upgrade 31a4ed468b18 -> 654298797277, Upgrade DB Schema
INFO  [alembic.runtime.migration] Running upgrade 654298797277 -> 0fb6d23a4863, Remove user avatar
INFO  [alembic.runtime.migration] Running upgrade 0fb6d23a4863 -> 856bb94b7040, Add comment column in domain template record table
INFO  [alembic.runtime.migration] Running upgrade 856bb94b7040 -> b0fea72a3f20, Update domain serial columns type
INFO  [alembic.runtime.migration] Running upgrade b0fea72a3f20 -> 3f76448bb6de, Add user.confirmed column
```

(flask) root@ns1:/opt/web/powerdns-admin# yarn install --pure-lockfile

(flask) root@ns1:/opt/web/powerdns-admin# flask assets build

(flask) root@ns1:/opt/web/powerdns-admin# ./run.py

``` sh
 * Serving Flask app "powerdnsadmin" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
[2020-04-29 08:00:02,873] [_internal.py:113] INFO -  * Running on http://0.0.0.0:9191/ (Press CTRL+C to quit)
[2020-04-29 08:00:02,874] [_internal.py:113] INFO -  * Restarting with stat
[2020-04-29 08:00:03,503] [_internal.py:113] WARNING -  * Debugger is active!
[2020-04-29 08:00:03,521] [_internal.py:113] INFO -  * Debugger PIN: 276-254-404
```





























**rwhitear@dns-01:~$** sudo apt-get install -y libmysqlclient-dev python-mysqldb libsasl2-dev libffi-dev libldap2-dev libssl-dev libxml2-dev libxslt1-dev libxmlsec1-dev pkg-config


```sh
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  gyp javascript-common libjs-async libjs-inherits libjs-jquery libjs-node-uuid libjs-underscore libllvm7 libuv1-dev linux-headers-4.18.0-15
  linux-headers-4.18.0-15-generic linux-image-4.18.0-15-generic linux-modules-4.18.0-15-generic linux-modules-extra-4.18.0-15-generic node-abbrev node-ansi
...
```

**rwhitear@dns-01:~$** sudo -i

**root@dns-01:~#** curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
```sh
OK
```

**root@dns-01:~#** echo "deb https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list

**root@dns-01:~#** exit

```sh
rwhitear@dns-01:~$
```

**rwhitear@dns-01:~$** sudo apt-get update

**rwhitear@dns-01:~$** sudo apt-get install  yarn -y

```sh
Reading package lists... Done
Building dependency tree
Reading state information... Done
```

**rwhitear@dns-01:~$** sudo git clone https://github.com/ngoduykhanh/PowerDNS-Admin.git /opt/web/powerdns-admin


```sh
Cloning into '/opt/web/powerdns-admin'...
remote: Enumerating objects: 56, done.
remote: Counting objects: 100% (56/56), done.
remote: Compressing objects: 100% (47/47), done.
remote: Total 8684 (delta 34), reused 18 (delta 9), pack-reused 8628
Receiving objects: 100% (8684/8684), 32.23 MiB | 1.83 MiB/s, done.
Resolving deltas: 100% (3940/3940), done.
```

**rwhitear@dns-01:~$** sudo -i

```sh
root@dns-01:~#
```

**root@dns-01:~#** cd /opt/web/powerdns-admin

```sh
root@dns-01:/opt/web/powerdns-admin#
```

**root@dns-01:/opt/web/powerdns-admin#** python3 -m venv flask

**root@dns-01:/opt/web/powerdns-admin#** . ./flask/bin/activate

**(flask) root@dns-01:/opt/web/powerdns-admin#** python -V

```sh
Python 3.6.9
```

**(flask) root@dns-01:/opt/web/powerdns-admin#** pip -V

```sh
pip 9.0.1 from /opt/web/powerdns-admin/flask/lib/python3.6/site-packages (python 3.6)
```

**(flask) root@dns-01:/opt/web/powerdns-admin#** pip install -r requirements.txt

```sh
Collecting Flask==1.1.1 (from -r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/9b/93/628509b8d5dc749656a9641f4caf13540e2cdec85276964ff8f43bbb1d3b/Flask-1.1.1-py2.py3-none-any.whl (94kB)
    100% |████████████████████████████████| 102kB 1.1MB/s
Collecting Flask-Assets==0.12 (from -r requirements.txt (line 2))
...
```

**(flask) root@dns-01:/opt/web/powerdns-admin#** exit

```sh
logout
rwhitear@dns-01:~$
```

**rwhitear@dns-01:~$** mysql -u root -p

```sh
Enter password: <ENTER MARISDB ROOT PASSWORD>
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2070
Server version: 10.4.12-MariaDB-1:10.4.12+maria~bionic-log mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE powerdnsadmin;

MariaDB [(none)]> GRANT ALL PRIVILEGES ON powerdnsadmin.* TO 'pdnsadminuser'@'%' \
IDENTIFIED BY '<ENTER PASSWORD HERE';

MariaDB [(none)]> FLUSH PRIVILEGES;

MariaDB [(none)]> quit
```

**rwhitear@dns-01:~$** sudo -i

cd /opt/web/powerdns-admin/powerdnsadmin

**root@dns-01:/opt/web/powerdns-admin/powerdnsadmin#** cp ./default_config.py config.py

**root@dns-01:/opt/web/powerdns-admin/powerdnsadmin#** vi ./config.py

```sh
import os
basedir = os.path.abspath(os.path.abspath(os.path.dirname(__file__)))

### BASIC APP CONFIG
SALT = '$2b$12$yLUMTIfl21FKJQpTkRQXCu'
SECRET_KEY = 'e951e5a1f4b94151b360f47edf596dd2'
BIND_ADDRESS = '0.0.0.0'
PORT = 9191
HSTS_ENABLED = False

### DATABASE CONFIG
SQLA_DB_USER = 'powerdns'
SQLA_DB_PASSWORD = '<DB PASSWORD HERE>'
SQLA_DB_HOST = '127.0.0.1'
SQLA_DB_NAME = 'powerdns'
SQLALCHEMY_TRACK_MODIFICATIONS = True

### DATBASE - MySQL
SQLALCHEMY_DATABASE_URI = 'mysql://'+SQLA_DB_USER+':'+SQLA_DB_PASSWORD+'@'+SQLA_DB_HOST+'/'+SQLA_DB_NAME

### DATABSE - SQLite
# SQLALCHEMY_DATABASE_URI = 'sqlite:///' + os.path.join(basedir, 'pdns.db')

# SAML Authnetication
SAML_ENABLED = False
```

**root@dns-01:/opt/web/powerdns-admin/powerdnsadmin#** . ../flask/bin/activate

**(flask) root@dns-01:/opt/web/powerdns-admin/powerdnsadmin#** export FLASK_CONF=./config.py

(flask) root@dns-01:/opt/web/powerdns-admin# export FLASK_CONF=config.py
(flask) root@dns-01:/opt/web/powerdns-admin# export FLASK_APP=powerdnsadmin/__init__.py

(flask) root@dns-01:/opt/web/powerdns-admin# flask db upgrade
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 787bdba9e147, Init DB
INFO  [alembic.runtime.migration] Running upgrade 787bdba9e147 -> 59729e468045, Add view column to setting table
INFO  [alembic.runtime.migration] Running upgrade 59729e468045 -> 1274ed462010, Change setting.value data type
INFO  [alembic.runtime.migration] Running upgrade 1274ed462010 -> 4a666113c7bb, Adding Operator Role
INFO  [alembic.runtime.migration] Running upgrade 4a666113c7bb -> 31a4ed468b18, Remove all setting in the DB
INFO  [alembic.runtime.migration] Running upgrade 31a4ed468b18 -> 654298797277, Upgrade DB Schema
INFO  [alembic.runtime.migration] Running upgrade 654298797277 -> 0fb6d23a4863, Remove user avatar
INFO  [alembic.runtime.migration] Running upgrade 0fb6d23a4863 -> 856bb94b7040, Add comment column in domain template record table
INFO  [alembic.runtime.migration] Running upgrade 856bb94b7040 -> b0fea72a3f20, Update domain serial columns type
INFO  [alembic.runtime.migration] Running upgrade b0fea72a3f20 -> 3f76448bb6de, Add user.confirmed column


(flask) root@dns-01:/opt/web/powerdns-admin# yarn install --pure-lockfile
yarn install v1.22.4
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...

Done in 13.82s.

(flask) root@dns-01:/opt/web/powerdns-admin# flask assets build
Building bundle: generated/login.js
[2020-03-12 12:03:07,140] [script.py:167] INFO - Building bundle: generated/login.js
Building bundle: generated/validation.js
[2020-03-12 12:03:07,324] [script.py:167] INFO - Building bundle: generated/validation.js
Building bundle: generated/login.css
[2020-03-12 12:03:07,326] [script.py:167] INFO - Building bundle: generated/login.css
Building bundle: generated/main.js
[2020-03-12 12:03:41,910] [script.py:167] INFO - Building bundle: generated/main.js
Building bundle: generated/main.css
[2020-03-12 12:03:42,714] [script.py:167] INFO - Building bundle: generated/main.css

rwhitear@dns-01:~$ sudo vi /etc/systemd/system/powerdns-admin.service

```sh
[Unit]
Description=PowerDNS-Admin
After=network.target

[Service]
User=root
Group=root
WorkingDirectory=/opt/web/powerdns-admin
ExecStart=/opt/web/powerdns-admin/flask/bin/gunicorn --workers 2 --bind unix:/opt/web/powerdns-admin/powerdns-admin.sock app:app

[Install]
WantedBy=multi-user.target
```

rwhitear@dns-01:~$ sudo systemctl daemon-reload

rwhitear@dns-01:~$ sudo systemctl start powerdns-admin

rwhitear@dns-01:~$ sudo systemctl enable powerdns-admin
```sh
Created symlink /etc/systemd/system/multi-user.target.wants/powerdns-admin.service → /etc/systemd/system/powerdns-admin.service.
```


```sh
[Unit]
Description=PowerDNS-Admin
Requires=powerdns-admin.socket
After=network.target

[Service]
PIDFile=/run/powerdns-admin/pid
User=pdns
Group=pdns
WorkingDirectory=/opt/web/powerdns-admin
ExecStart=/opt/web/powerdns-admin/flask/bin/gunicorn --pid /run/powerdns-admin/pid --bind unix:/run/powerdns-admin/socket 'powerdnsadmin:create_app()'
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

rwhitear@dns-01:~$ sudo vi /etc/systemd/system/powerdns-admin.socket

```sh
[Unit]
Description=PowerDNS-Admin socket

[Socket]
ListenStream=/run/powerdns-admin/socket

[Install]
WantedBy=sockets.target
```

rwhitear@dns-01:~$ sudo vi /etc/tmpfiles.d/powerdns-admin.conf

```sh
d /run/powerdns-admin 0755 pdns pdns -
```

root@dns-01:/run# chmod 777 /run/powerdns-admin/



rwhitear@dns-01:~$ sudo systemctl daemon-reload
sudo systemctl start powerdns-admin.socket
sudo systemctl enable powerdns-admin.socket
rwhitear@dns-01:~$ sudo systemctl restart powerdns-admin




apt-get install pdns-recursor


sudo apt-get install nginx -y

rwhitear@dns-01:~$ sudo vi /etc/nginx/conf.d/powerdns-admin.conf

server {
  listen *:80;
  server_name               powerdns-admin.example.com www.powerdns-admin.example.com;

  index                     index.html index.htm index.php;
  root                      /opt/web/powerdns-admin;
  access_log                /var/log/nginx/powerdns-admin.local.access.log combined;
  error_log                 /var/log/nginx/powerdns-admin.local.error.log;

  client_max_body_size              10m;
  client_body_buffer_size           128k;
  proxy_redirect                    off;
  proxy_connect_timeout             90;
  proxy_send_timeout                90;
  proxy_read_timeout                90;
  proxy_buffers                     32 4k;
  proxy_buffer_size                 8k;
  proxy_set_header                  Host $host;
  proxy_set_header                  X-Real-IP $remote_addr;
  proxy_set_header                  X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_headers_hash_bucket_size    64;

  location ~ ^/static/  {
    include  /etc/nginx/mime.types;
    root /opt/web/powerdns-admin/app;

    location ~*  \.(jpg|jpeg|png|gif)$ {
      expires 365d;
    }

    location ~* ^.+.(css|js)$ {
      expires 7d;
    }
  }

  location / {
    proxy_pass            http://unix:/opt/web/powerdns-admin/powerdns-admin.sock;
    proxy_read_timeout    120;
    proxy_connect_timeout 120;
    proxy_redirect        off;
  }
}

rwhitear@dns-01:~$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

rwhitear@dns-01:~$ sudo systemctl restart nginx
rwhitear@dns-01:~$ sudo systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2020-03-12 18:35:34 GMT; 6s ago
     Docs: man:nginx(8)
  Process: 3823 ExecStop=/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid (code=exited, status=0/SUCCESS)
  Process: 3841 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
  Process: 3824 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
 Main PID: 3849 (nginx)
    Tasks: 3 (limit: 4666)
   CGroup: /system.slice/nginx.service
           ├─3849 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           ├─3850 nginx: worker process
           └─3851 nginx: worker process

Mar 12 18:35:34 dns-01 systemd[1]: Starting A high performance web server and a reverse proxy server...
Mar 12 18:35:34 dns-01 systemd[1]: Started A high performance web server and a reverse proxy server.
