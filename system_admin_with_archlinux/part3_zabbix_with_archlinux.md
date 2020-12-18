# Part 3: Zabbix with ArchLinux

## Installation

Zabbix packages can be installed from the main ArchLinux repository. The process is described at https://wiki.archlinux.org/index.php/Zabbix.

1. Install *zabbix-server*.

```
 $ pacman -S zabbix-server
 ```

 2. Instal mariadb.

 ```
 $ pacman -S mariadb
 ```

3. Install apache

```
$ pacman -S apache
```

4. Installe zabbix-frontend-php

```
$ pacman -S zabbix-frontend-php
```

5. Install PHP implementation

```
$ pacman -S php-fpm
```

## Configuration

1. Symlink the Zabbix web application directory to your http document root

```
$ ln -s /usr/share/webapps/zabbix /srv/http/zabbix
```

2. Follow the instructions at https://wiki.archlinux.org/index.php/Apache_HTTP_Server#Using_php-fpm_and_mod_proxy_fcgi to enable PHP.

3. Make sure to adjust following variables to these minimal values in your /etc/php/php.ini: 

```
extension=bcmath
extension=gd
extension=sockets
extension=mysqli
extension=gettext
post_max_size = 16M
max_execution_time = 300
max_input_time = 300
date.timezone = "UTC"
```

4. Create the MySQL database

```
 mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
 ```

5. Start MySQL

```
systemctl start mysql
```

6. Create user for zabbix.

```
$ mysql -u root -p -e "create database zabbix character set utf8 collate utf8_bin"
$ mysql -u root -p -e "grant all on zabbix.* to zabbix@localhost identified by 'test'"
$ mysql -u zabbix -p zabbix < /usr/share/zabbix-server/mysql/schema.sql
$ mysql -u zabbix -p zabbix < /usr/share/zabbix-server/mysql/images.sql
$ mysql -u zabbix -p zabbix < /usr/share/zabbix-server/mysql/data.sql
```

7. Now edit /etc/zabbix/zabbix_server.conf with the database settings:

```
/etc/zabbix/zabbix_server.conf

DBName=zabbix
DBUser=zabbix
DBPassword=test
LogType=system
```

## Starting

1. Start PHP service

```
$ systemctl start php-fpm.service
```

2. Start apache

```
$ systemctl start httpd
```

3. Start mysql

```
$ systemctl start mysql
```

4. Start zabbix

```
$ systemctl start zabbix-server-mysql.service
 ```

5. Start zabbix-agent

$ systemctl start zabbix-agent

 ## Accessing

 1. Navigate to http://localhost/zabbix

 2. Login as Admin/zabbix