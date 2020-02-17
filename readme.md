# LaraBOX8

<!-- TOC -->

- [LaraBOX8](#larabox8)
    - [1. Initial CentOS 8 Vagrant box](#1-initial-centos-8-vagrant-box)
    - [2. Installing LAMP stack](#2-installing-lamp-stack)
        - [2.1. Prerequisites](#21-prerequisites)
        - [2.2 Apache 2.4.37](#22-apache-2437)
            - [Installing Apache](#installing-apache)
            - [Setting up Virtual Hosts](#setting-up-virtual-hosts)
        - [2.3 MySQL 8.0.19](#23-mysql-8019)
            - [Installing MySQL](#installing-mysql)
            - [Secure MySQL Server](#secure-mysql-server)
        - [2.4 PHP 7.4.2](#24-php-742)
    - [3. Installing Composer 1.9.3](#3-installing-composer-193)
    - [4. Installing NodeJS 10.16.3-2 and npm](#4-installing-nodejs-10163-2-and-npm)
    - [5. Installing Git 2.18.2 and Ungit 0.10.3](#5-installing-git-2182-and-ungit-0103)
        - [Installing Git](#installing-git)
        - [Installing Ungit](#installing-ungit)
    - [6. Rainloop + Dovecot + Postfix](#6-rainloop--dovecot--postfix)
        - [6.1. Postfix](#61-postfix)
        - [6.2. Dovecot](#62-dovecot)
        - [6.3. Install Rainloop](#63-install-rainloop)
    - [7. Adminer 4.7.6](#7-adminer-476)
    - [8. phpMyAdmin 5.0.1](#8-phpmyadmin-501)
    - [7. Samba 4 for File Sharing](#7-samba-4-for-file-sharing)

<!-- /TOC -->

## 1. Initial CentOS 8 Vagrant box

```cmd
vagrant init centos/8
truncate Vagrantfile -s 0
```

Modify the above `Vagrantfile` with the flowing contents. Remember change the `config.vm.hostname` and `config.vm.network` with your own setting.

```Ruby
Vagrant.configure("2") do |config|
  config.vm.box = "centos/8"
  config.vm.hostname = "larabox8"
  config.vm.network "public_network", ip: "192.168.0.100", netmask: "255.255.255.0", gateway: "192.168.0.1"

  config.vm.provider "virtualbox" do |vb|
     vb.cpus = 1
     vb.memory = "512"
  end
end
```

Startup vagrant box by command `vagrant up --provision`.

Some Vagrant usage command

```cmd
# Connection to box via SSH
vagrant ssh

# Reload box
vagrant reload --provision

# Check status current box
vagrant status

# Check status all box
vagrant global-status

# Shutdown box
vagrant halt

# Destroy box
vagrant destroy --force

# Remove box image
vagrant box remove <box_name>
```

How to ssh to vagrant box using `root` account

```cmd
vagrant ssh
```

Install vim editor first

```cmd
sudo su
dnf install vim
```

Edit ssh config

```cmd
vim /etc/ssh/sshd_config
```

> Find the line #73 change `PasswordAuthentication no` to `PasswordAuthentication yes`.

```cmd
systemctl restart sshd
exit

# SSH with user `root` and password `vagrant`
ssh root@192.168.0.100
```

Turn off selinux

```cmd
vim /etc/selinux/config
```

Change `SELINUX=enforcing` to `SELINUX=disabled` and reboot system. Run command `sestatus` to check status.

## 2. Installing LAMP stack

### 2.1. Prerequisites

> Start with CentOS version 8 you can using `dnf` also with `yum` old-school.

Enable both repositories on your system using the following commands on your CentOS 8 system.

```cmd
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
dnf install yum-utils
```

### 2.2 Apache 2.4.37

#### Installing Apache

```cmd
dnf install httpd
```

Check Apache version

```cmd
httpd -v

# Output
Server version: Apache/2.4.37 (centos)
Server built:   Dec 23 2019 20:45:34
```

Make Apache run at startup

```cmd
systemctl enable httpd
```

Starting Apache

```cmd
systemctl start httpd
```

Check Apache status

```cmd
systemctl status httpd
```

Output

```cmd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2020-02-06 14:56:46 UTC; 21min ago
     Docs: man:httpd.service(8)
 Main PID: 1084 (httpd)
   Status: "Total requests: 6; Idle/Busy workers 100/0;Requests/sec: 0.00472; Bytes served/sec: 256 B/sec"
    Tasks: 213 (limit: 2881)
   Memory: 11.1M
   CGroup: /system.slice/httpd.service
           ├─1084 /usr/sbin/httpd -DFOREGROUND
           ├─1086 /usr/sbin/httpd -DFOREGROUND
           ├─1087 /usr/sbin/httpd -DFOREGROUND
           ├─1088 /usr/sbin/httpd -DFOREGROUND
           └─1089 /usr/sbin/httpd -DFOREGROUND

Feb 06 14:56:46 larabox8 systemd[1]: Stopped The Apache HTTP Server.
Feb 06 14:56:46 larabox8 systemd[1]: Starting The Apache HTTP Server...
Feb 06 14:56:46 larabox8 systemd[1]: Started The Apache HTTP Server.
Feb 06 14:56:46 larabox8 httpd[1084]: Server configured, listening on: port 80
```

#### Setting up Virtual Hosts

```cmd
mkdir -p /var/www/vhosts
mkdir -p /etc/httpd/conf.d/vhosts
vim /etc/httpd/conf/http.conf
```

Modify the `http.conf` file with below contents

```apache
# Add
servername localhost
IncludeOptional conf.d/vhosts/*.conf
AddType application/x-httpd-php .php

# Change
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>
```

### 2.3 MySQL 8.0.19

#### Installing MySQL

Add the official repository of MySQL to install the MySQL community server.

```cmd
dnf install https://dev.mysql.com/get/mysql80-community-release-el8-1.noarch.rpm
```

Make sure the MySQL repository has been added and enabled by using the following command.

```cmd
dnf repolist all | grep mysql | grep enabled
```

Disable AppStream (default) repository temporarily to install MySQL from MySQL Dev Community

```cmd
dnf --disablerepo=AppStream install mysql-community-server
```

Check MySQL version

```cmd
mysql -V

# Output
mysql  Ver 8.0.19 for Linux on x86_64 (MySQL Community Server - GPL)
```

Make MySQL run at startup

```cmd
systemctl enable mysqld
```

Starting MySQL

```cmd
systemctl start mysqld
```

Check MySQL status

```cmd
systemctl status mysqld
```

Output

```cmd
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2020-02-06 15:16:54 UTC; 10s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 3754 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 3830 (mysqld)
   Status: "Server is operational"
    Tasks: 39 (limit: 2881)
   Memory: 356.6M
   CGroup: /system.slice/mysqld.service
           └─3830 /usr/sbin/mysqld

Feb 06 15:16:45 larabox8 systemd[1]: Starting MySQL Server...
Feb 06 15:16:54 larabox8 systemd[1]: Started MySQL Server.
```

#### Secure MySQL Server

In CentOS 8, the initial MySQL password can be found in `/var/log/mysqld.log`. You can use the below command to take the password from the log file.

```cmd
cat /var/log/mysqld.log | grep -i 'temporary password'
```

Output

```cmd
2020-02-06T15:16:48.833311Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: SqdWyk8&,>Ms
```

Now, you need to run `mysql_secure_installation` to secure your MySQL installation. This command takes care of setting the root password, removing anonymous users, disallow root login remotely, etc.

```cmd
mysql_secure_installation
```

Output

```cmd
Securing the MySQL server deployment.

Enter password for user root:

The existing password for the user account root has expired. Please set a new password.

New password:

Re-enter new password:
The 'validate_password' component is installed on the server.
The subsequent steps will run with the existing configuration
of the component.
Using existing password for root.

Estimated strength of the password: 100
Change the password for root ? ((Press y|Y for Yes, any other key for No) : N

 ... skipping.
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : Y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : N

 ... skipping.
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : Y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : Y
Success.

All done!
```

Login to MySQL server as the MySQL root user.

```cmd
mysql -u root -p
```

Output

```cmd
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 8.0.19 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

### 2.4 PHP 7.4.2

Enable the module stream for 7.4

```cmd
dnf module install php:remi-7.4
```

Install additional packages

```cmd
dnf install php-mysqlnd php-zip php-devel php-gd php-mcrypt php-curl php-pear php-bcmath
```

Check PHP version

```cmd
php --version

# Output
PHP 7.4.2 (cli) (built: Jan 21 2020 11:35:20) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
```

Check available extentions

```cmd
php --modules
```

## 3. Installing Composer 1.9.3

```cmd
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

Check Composer version

```cmd
composer

# Output
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 1.9.3 2020-02-04 12:58:49

Usage:
...
```

## 4. Installing NodeJS 10.16.3-2 and npm

```cmd
dnf install nodejs
```

Check NodeJS and NPM version

```cmd
node -v
npm -v
```

## 5. Installing Git 2.18.2 and Ungit 0.10.3

### Installing Git

```cmd
dnf install git
```

### Installing Ungit

```cmd
npm install -g ungit@0.10.3
```

Make Ungit run at startup

Create `ungit.service` file

```cmd
touch /etc/systemd/system/ungit.service
chmod 664 ungit.service
```

Add `ungit.service` file with the following contents

```cmd
[Unit]
Description=ungit-service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/home/vagrant
ExecStart=/usr/bin/ungit
TimeoutSec=15
Restart=always

[Install]
WantedBy=multi-user.target
```

Start ungit service

```cmd
systemctl enable ungit
systemct daemon-reload
systemctl start ungit
systemctl status ungit
```

## 6. Rainloop + Dovecot + Postfix

### 6.1. Postfix

For some unknown reasons, main repo of RHEL doesn't support postfix-pcre. So that we need use 3rd repo.

```cmd
rpm -Uvh http://mirror.ghettoforge.org/distributions/gf/el/8/testing/x86_64//postfix3-3.4.9-1.gf.el8.x86_64.rpm
rpm -Uvh http://mirror.ghettoforge.org/distributions/gf/el/8/testing/x86_64//postfix3-pcre-3.4.9-1.gf.el8.x86_64.rpm
```

Add new `catchall` user

```cmd
adduser catchall
passwd catchall
```

PCRE regrex

```cmd
vim /etc/aliases.regexp

# Add below contents
/(?!^root$|^catchall$)^.*$/ catchall
```

Edit `main.cf` file

```cmd
cd /etc/postfix/
cp main.cf main.cf.bak
vim main.cf
```

Add below contents

```conf
# uncomment
home_mailbox = Maildir/

# change this line
alias_maps = hash:/etc/aliases, pcre:/etc/aliases.regexp

# add newline
transport_maps = pcre:/etc/postfix/transport_maps
```

Edit `/etc/aliases` file

```cmd
# find postmaster:\troot replace by postmaster:\tcatchall
sed -i "s/postmaster:\troot/postmaster:\tcatchall/" /etc/aliases
```

Create `/etc/postfix/transport_maps` file

```cmd
vim /etc/postfix/transport_maps

# Add below contents
/^.*@.*$/ local
```

### 6.2. Dovecot

```cmd
dnf install dovecot
```

Edit `10-mail.conf` file

```conf
vim /etc/dovecot/conf.d/10-mail.conf

# Add below contents
mail_location = maildir:~/Maildir
```

Reload the services

```cmd
postalias /etc/aliases
postmap /etc/postfix/transport
systemctl restart postfix
systemctl restart dovecot
systemctl enable dovecot
```

### 6.3. Install Rainloop

Download the latest rainloop

```cmd
curl -o rainloop-latest.zip https://www.rainloop.net/repository/webmail/rainloop-latest.zip
unzip
```

Install unzip and extract the archive

```cmd
dnf install unzip
mkdir -p /var/www/vhosts/rainloop
unzip rainloop-latest.zip -d /var/www/vhosts/rainloop/
```

Config rainloop folder

```cmd
find /var/www/vhosts/rainloop -type d -exec chmod 755 {} \;
find /var/www/vhosts/rainloop -type f -exec chmod 644 {} \;
chown -R apache:apache /var/www/vhosts/rainloop/
```

Configure Apache web Server

```cmd
vim /etc/httpd/conf.d/vhosts/rainloop.conf
```

Edit the `rainloop.conf`

```conf
Alias /rainloop /var/www/vhosts/rainloop
<Directory "/var/www/vhosts/rainloop">
    Options Indexes FollowSymLinks MultiViews
    AllowOverride all
    Require all granted
</Directory>
```

Restart Apache service

```cmd
systemctl restart httpd
```

Config the rainloop

Access domain `vagrant_ipaddress/rainloop/?admin` and login with info `admin|12345`

Config `Domains` sections

![rainloop](/img/Screenshot&#32;from&#32;2019-08-19&#32;22-21-33.png)

Config `Login` section

![rainloop](/img/Screenshot&#32;from&#32;2019-08-19&#32;22-14-00.png)

Install `mailx`

```cmd
dnf install mailx
```

Send the test mail. Press `Ctr + D` to send mail.

```cmd
mail -s "This is Subject" -r "sender<sender@mail.com>" someone@example.com
```

## 7. Adminer 4.7.6

Download Adminer

```cmd
mkdir -p /var/www/vhosts/adminer
cd /var/www/vhosts/adminer
dnf install wget
wget https://github.com/vrana/adminer/releases/download/v4.7.6/adminer-4.7.6.php -O index.php
```

Create `adminer.conf` vhosts file

```cmd
vim /etc/httpd/conf.d/vhosts/adminer.conf
```

Add contents

```conf
Alias /adminer /var/www/vhosts/adminer/index.php
```

Change the permission vhosts

```cmd
chown -R apache:apache /var/www/vhosts/adminer
chmod -R 755 /var/www/vhosts/adminer
```

Restart Apache service

```cmd
systemctl restart httpd
```

## 8. phpMyAdmin 5.0.1

Download the latest release

```cmd
curl -o phpMyAdmin-5.0.1-english.tar.gz https://files.phpmyadmin.net/phpMyAdmin/5.0.1/phpMyAdmin-5.0.1-english.tar.gz
```

Extract downloaded archive

```cmd
tar xvf phpMyAdmin-5.0.1-english.tar.gz
```

Move the folder to `/usr/share/phpmyadmin`

```cmd
mv phpMyAdmin-5.0.1-english*/ /usr/share/phpmyadmin
```

Create directory for phpMyAdmin temp files.

```cmd
mkdir -p /var/lib/phpmyadmin/tmp
chown -R apache:apache /var/lib/phpmyadmin
```

Create directory for phpMyAdmin configuration files.

```cmd
mkdir /etc/phpmyadmin/
```

Create phpMyAdmin configuration file.

```cmd
cp /usr/share/phpmyadmin/config.sample.inc.php /usr/share/phpmyadmin/config.inc.php
```

Edit the file `config.inc.php`

```conf
# Set a secret passphrase – Needs to be 32 chars long
$cfg['blowfish_secret'] = 'TPfb1qcZoAzrO8UtGBD4qC6wMjc9jQoS';

# Configure temp directory
$cfg['TempDir'] = '/var/lib/phpmyadmin/tmp';
```

Configure Apache web Server

```cmd
vim /etc/httpd/conf.d/vhosts/phpmyadmin.conf
```

Edit the `phpmyadmin.conf`

```conf
Alias /phpmyadmin /usr/share/phpmyadmin/
Alias /phpmyAdmin /usr/share/phpmyadmin/
<Directory "usr/share/phpmyadmin">
    Options Indexes FollowSymLinks MultiViews
    AllowOverride all
    Require all granted
</Directory>
```

## 7. Samba 4 for File Sharing

Install `Samba`

```cmd
dnf install samba samba-client
```

Startup and enable `smb` and `nmb` daemons at boot

```cmd
systemctl enable smb
systemctl enable nmb
systemctl start smb
systemctl start nmb
```

Configuring a shared directory accessible by guests

Edit the `/etc/samba/smb.conf`

```cmd
vim /etc/samba/smb.conf
[global]
        workgroup = WORKGROUP
        map to guest = bad user
        force user = root
[vhosts]
        path = /var/www/vhosts
        browsable = yes
        writable = yes
        guest ok = yes
        read only = no
        follow symlinks = yes
```

Restart `smb` daemons

```cmd
systemctl restart smb
systemctl restart nmb
```