---
layout: post
title: Install Cacti monitoring from source
description: Install Cacti monitoring from source
summary: Install Cacti monitoring from source
comments: false
---

Figured this must help anyone trying to install Cacti monitoring from source on Ubuntu Server 18.04.

* https://www.cacti.net/
* https://github.com/Cacti/cacti

----- 

#### Before we start, do the following:

Update `/etc/apt/sources.list` file that lists the 'sources' from which packages can be obtained.

```
sudo vim /etc/apt/sources.list
 
  deb http://archive.ubuntu.com/ubuntu bionic main multiverse restricted universe
  deb http://archive.ubuntu.com/ubuntu bionic-security main multiverse restricted universe
  deb http://archive.ubuntu.com/ubuntu bionic-updates main multiverse restricted universe
```

```
sudo apt update
```

#### INSTALL DEPENDENCIES

```          
sudo apt -y install wget patch unzip zip bash-completion apache2 build-essential dos2unix dh-autoreconf help2man libssl-dev libmysql++-dev librrds-perl libsnmp-dev 

sudo apt -y install php7.2 php7.2-dev php-pear libapache2-mod-php7.2 php7.2-snmp php7.2-xml php7.2-mbstring php7.2-json php7.2-gd php7.2-gmp php7.2-zip php7.2-ldap   
```

#### INSTALL MyCrypt IN PHP 7.2.0

The mcrypt extension is an interface to the mcrypt cryptography library. This extension is useful for allowing PHP code using mcrypt to run on PHP 7.2+.

The main problem with mcrypt extension is that it is based on libmcrypt that hasn’t been developped since 2007 and has thus become a security alert for many system administrators.

Because of the end of the mcrypt extension’s development, the extention was also removed from PHP 7.2 and moved to an unofficial PECL repository. However, you can still find the mcrypt extention in PHP 5.4 through PHP 7.1. The arrival of PHP 7.2 has been announced but it won’t contain mcrypt extention. For PHP 7.2+, PHP instead uses libsodium as a cryptography library.

```
sudo apt-get -y install gcc make autoconf libc-dev pkg-config libmcrypt-dev
```

```
sudo pecl install mcrypt-1.0.1

Build process completed successfully
Installing '/usr/lib/php/20170718/mcrypt.so'
install ok: channel://pecl.php.net/mcrypt-1.0.1
configuration option "php_ini" is not set to php.ini location
You should add "extension=mcrypt.so" to php.ini
```

**NOTE** If you get the following error, ensure you have `php7.2-dev` installed.

```
Created directory: /var/lib/snmp/mib_indexes
WARNING: channel "pecl.php.net" has updated its protocols, use "pecl channel-update pecl.php.net" to update
downloading mcrypt-1.0.1.tgz ...
Starting to download mcrypt-1.0.1.tgz (33,782 bytes)
.........done: 33,782 bytes
6 source files, building
running: phpize
sh: 1: phpize: not found
ERROR: `phpize' failed
```

```
sudo bash -c "echo extension=/usr/lib/php/20170718/mcrypt.so > /etc/php/7.2/cli/conf.d/mcrypt.ini"
sudo bash -c "echo extension=/usr/lib/php/20170718/mcrypt.so > /etc/php/7.2/apache2/conf.d/mcrypt.ini"
```

```
sudo chmod +x /etc/php/7.2/cli/conf.d/mcrypt.ini 
sudo chmod +x /etc/php/7.2/apache2/conf.d/mcrypt.ini 
```

#### VERIFY EXTENSION WAS INSTALLED

```
php -i | grep "mcrypt"

/etc/php/7.2/cli/conf.d/mcrypt.ini
...
Registered Stream Filters => zlib.*, string.rot13, string.toupper, string.tolower, string.strip_tags, convert.*, consumed, dechunk, convert.iconv.*, mcrypt.*, mdecrypt.*
mcrypt
mcrypt support => enabled
mcrypt_filter support => enabled
mcrypt.algorithms_dir => no value => no value
mcrypt.modes_dir => no value => no value
```

----- 

#### CONFIGURE PHP

Configure `date.timezone` and ensure `file_uploads = On`:

```
grep file_uploads /etc/php/7.2/apache2/php.ini 

  file_uploads = On
  max_file_uploads = 20

grep date.timezone /etc/php/7.2/apache2/php.ini 

  ; http://php.net/date.timezone
  date.timezone = Europe/London
```

```
grep file_uploads /etc/php/7.2/cli/php.ini 

  file_uploads = On
  max_file_uploads = 20
  
grep date.timezone /etc/php/7.2/cli/php.ini 

  ; http://php.net/date.timezone
  date.timezone = Europe/London
```

Restart Apache:

```
systemctl restart apache2

systemctl status apache2
```

------ 

#### BUILD CACTI DATABASE

Cacti stores its data in an RDBMS database. To make that happen, we'll configure Cacti to work with MariaDB. Install this database with the command:

```
sudo apt install mariadb-server php7.2-mysql
```

Next we secure the MariaDB `root` account with the following commands:

```
sudo mysql -h localhost
...
MariaDB [(none)]> use mysql;
MariaDB [mysql]> update user set plugin='' where user='root';
MariaDB [mysql]> flush privileges;
MariaDB [mysql]> exit

Bye
```

Running `mysql_secure_installation` will prompt you to answer a few questions. The first will be to enter the current password for the root user. Since there is no password, hit Enter on your keyboard and then type y to change the root password. 

Type and verify the new password, and then answer the remainder of the questions with the default answers.
  
```
sudo mysql_secure_installation 

Set root password? [Y/n] y
Remove anonymous users? [Y/n] 
Disallow root login remotely? [Y/n] 
Remove test database and access to it? [Y/n] 
Reload privilege tables now? [Y/n] 
...
Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

Now create the cacti database.

```
mysql -h localhost -u root -p

...
MariaDB [(none)]> create database cacti;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> grant all on cacti.* to 'cactiuser'@'localhost' identified by 'cactoid';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye
```

Set permissions to the new database user according to the correct time zone.

```
mysql -u root -p mysql < /usr/share/mysql/mysql_test_data_timezone.sql
mysql -u root -p -e 'grant select on mysql.time_zone_name to root@localhost'
mysql -u root -p -e 'grant select on mysql.time_zone_name to cactiuser@localhost'
```

Append the following lines to the `/etc/mysql/mariadb.conf.d/50-server.cnf` file:

```
vim /etc/mysql/mariadb.conf.d/50-server.cnf

max_heap_table_size = 98M
tmp_table_size = 64M
join_buffer_size = 64M
innodb_buffer_pool_size = 485M
innodb_doublewrite = off
innodb_additional_mem_pool_size = 80M
innodb_flush_log_at_timeout = 3
innodb_read_io_threads = 32
innodb_write_io_threads = 16
```

Restart Apache and MySQL:

```
sudo systemctl restart mysql apache2
```

----- 

#### CONFIGURE SNMP

Install and configure the SNMP service.

```
sudo apt install snmp snmpd snmp-mibs-downloader
```

Install RRDtool.

```
sudo apt install rrdtool
```

Comment out the line `mibs :` in `/etc/snmp/snmp.conf`.

```
vim /etc/snmp/snmp.conf

#mibs :
```

Configure `/etc/snmp/snmpd.conf` as the following:

```
vim /etc/snmp/snmpd.conf`

#agentAddress udp:127.0.0.1:161
agentAddress udp:161,udp6:[::1]:161
```

In the same file (`/etc/snmp/snmpd.conf` add the following lines underneath `rocommunity6 public default -V systemonly`:

```
...
rocommunity6 public default -V systemonly
rocommunity snmp_string localhost
rocommunity snmp_string 192.168.1.0/24
```

Restart the SNMP service.

```
systemctl restart snmpd.service
```

**If you have a firewall running, open up the proper port with the command:**

```
sudo ufw allow 161/udp
```

----- 

#### INSTALL Cacti-Spine

Cacti-Spine a tool that replaces the default cmd.php poller.

```
root@kea:~# wget https://www.cacti.net/downloads/spine/cacti-spine-latest.tar.gz

--2019-03-03 15:00:51--  https://www.cacti.net/downloads/spine/cacti-spine-latest.tar.gz
Resolving www.cacti.net (www.cacti.net)... 104.28.9.127, 104.28.8.127, 2606:4700:30::681c:87f, ...
Connecting to www.cacti.net (www.cacti.net)|104.28.9.127|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2310298 (2.2M) [application/x-gzip]
Saving to: ‘cacti-spine-latest.tar.gz’

cacti-spine-latest.tar. 100%[============================>]   2.20M  1.88MB/s    in 1.2s    

2019-03-03 15:00:52 (1.88 MB/s) - ‘cacti-spine-latest.tar.gz’ saved [2310298/2310298]
```

Extract and build:

```
root@kea:~# tar xfz cacti-spine-latest.tar.gz 

root@kea:~# cd cacti-spine-1.2.2/

root@kea:~/cacti-spine-1.2.2# ./bootstrap 

root@kea:~/cacti-spine-1.2.2# ./configure

root@kea:~/cacti-spine-1.2.2# make

root@kea:~/cacti-spine-1.2.2# make install

root@kea:~/cacti-spine-1.2.2# chown root:root /usr/local/spine/bin/spine 

root@kea:~/cacti-spine-1.2.2# chmod +s /usr/local/spine/bin/spine 
```

Next we configure Cacti-Spine to use our new database. 

Open the file `/usr/local/spine/etc/spine.conf` and edit the database credentials according to what you setup during the database installation/configuration. 

You'll need to change `DB_User` and `DB_Pass`.

```
root@kea:~/cacti-spine-1.2.2# sudo vim /usr/local/spine/etc/spine.conf.dist 

DB_Host       localhost
DB_Database   cacti
DB_User       cactiuser
DB_Pass       cactoid
DB_Port       3306
```

----- 

#### Install Cacti

Finally the installation of Cacti. This is done with the following commands:

```
root@kea:~# wget https://www.cacti.net/downloads/cacti-latest.tar.gz
--2019-03-03 15:09:53--  https://www.cacti.net/downloads/cacti-latest.tar.gz
Resolving www.cacti.net (www.cacti.net)... 104.28.9.127, 104.28.8.127, 2606:4700:30::681c:97f, ...
Connecting to www.cacti.net (www.cacti.net)|104.28.9.127|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 23800723 (23M) [application/x-gzip]
Saving to: ‘cacti-latest.tar.gz’

cacti-latest.tar.gz     100%[============================>]  22.70M  5.99MB/s    in 5.3s    

2019-03-03 15:09:59 (4.31 MB/s) - ‘cacti-latest.tar.gz’ saved [23800723/23800723]


tar xfz cacti-latest.tar.gz

root@kea:~# cp -rf cacti-1.2.2/* /var/www/html/
root@kea:~# chown -R www-data:www-data /var/www/html/*
```

In similar fashion to Cacti-Spine, the Cacti configuration must be setup to connect to the database. Open the file `/var/www/html/include/config.php` and change the credentials as per the database install/configuration. 

You'll need to change `database_username` and `database_password`. 

```
$database_type     = 'mysql';
$database_default  = 'cacti';
$database_hostname = 'localhost';
$database_username = 'cactiuser';
$database_password = 'cactoid';
$database_port     = '3306';
$database_ssl      = false;
```

In that same file, amend the line `$url_path = '/cacti/';` with:

```
$url_path = "/";
```

Populate the cacti database with the command:

```
mysql -u cactiuser cacti -p < /var/www/html/cacti.sql
```

Issue the command `mysql -u cactiuser cacti -p -e 'show tables'` and you should see the newly populated data:

Before moving on to the web installation, issue the following commands:

```
rm /var/www/html/index.html
touch /var/www/html/log/cacti.log
chown -R www-data:www-data /var/www/html/
```


------ 


#### TIMEZONE ERROR

Grant the Cacti database account access to the MySQL TimeZone database and ensure that database is populated with global TimeZone information.

```
mysql -u root -p mysql 

MariaDB [mysql]> grant select on mysql.time_zone_name to root@localhost;

MariaDB [mysql]> grant select on mysql.time_zone_name to cactiuser@localhost;

MariaDB [mysql]> flush privileges;

MariaDB [mysql]> exit

Bye
```

Set system timezone:

```
root@kea:~# timedatectl 
                      Local time: Wed 2019-03-06 17:22:37 UTC
                  Universal time: Wed 2019-03-06 17:22:37 UTC
                        RTC time: Wed 2019-03-06 17:22:37
                       Time zone: Etc/UTC (UTC, +0000)
       System clock synchronized: yes
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
                 
root@kea:~# sudo timedatectl set-timezone Europe/London
```

------ 

#### POLLER ISSUES

Ensure the poller files exist and are in the correct directory.

```
root@kea:~# sudo find / -iname poller.php 2>/dev/null
/var/www/html/poller.php
/var/www/html/lib/poller.php
/home/lawrence/cacti-1.2.2/poller.php
/home/lawrence/cacti-1.2.2/lib/poller.php
```

```
touch /var/www/html/poller-error.log
chown www-data:www-data /var/www/html/poller-error.log
```

I noticed that a cronjob is not installed in the latest version of cacti, therefore I added it myself.

```
root@kea:~# crontab -e

*/5 * * * * cactiuser php -q /var/www/html/poller.php  &>/dev/null
```

```
cactiuser php /var/www/html/poller.php > /dev/null 2>&1
```

If still not plotting, go to 'Console>Utilities>Rebuild Poller Cache' and then: 
```
root@kea:~$ cactiuser php -q /var/www/html/poller.php > /dev/null 2>&1
root@kea:~# chown -R www-data:www-data /var/www/html/
```