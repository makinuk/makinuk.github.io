---		
layout: post		
title: LEMP-CentOS 7 Installation		
published: true		
categories: [Centos7]		
date: 2017-03-11 20:45:35		
---		
 
## Prepare Machine
 
```sh
sudo yum install epel-release
sudo yum update
```
 
## Install Nginx
 
```sh
sudo yum -y install nginx
sudo systemctl start nginx
sudo firewall-cmd --permanent --zone=public --add-service=http 
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
sudo systemctl enable nginx
sudo systemctl start nginx
```
 
## Snmp Basic Configuration 
- install snmp `yum -y install net-snmp net-snmp-utils`
- keep orginal config file `cp /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.BAK`
- clear current config file `echo "" > /etc/snmp/snmpd.conf` and paste below text with changing SERVERHOST
```
rocommunity  public  *SERVERHOST*
rocommunity  public   127.0.0.1
syslocation  "HUS, DTM"
syscontact  m.akin@medyatakip.com
 
disk /
disk /home
 
includeAllDisks 10%
```
 
- Open `vi /etc/firewalld/services/snmp.xml` and copy below text
```
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>SNMP</short>
  <description>SNMP protocol</description>
  <port protocol="udp" port="161"/>
</service>
```
- reload fierwall  `firewall-cmd --reload`
- dd the service to your public zone `firewall-cmd --zone=public --add-service snmp --permanent`
- again reload fierwall `firewall-cmd --reload`
 
 
 
## Install MariaDB

```sh
sudo yum install mariadb-server mariadb -y
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo mysql_secure_installation
```
 
## Install PHP
### PHP Latest Stable For Centos

```sh
sudo yum install -y php php-mysql php-fpm php-intl php-common php-opcache php-dom php-mcrypt
```

### PHP 7 
```sh
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
sudo yum install -y php70w php70w-mysql php70w-fpm php70w-intl php70w-common php70w-opcache php70w-dom php70w-mcrypt
```
 
```sh
sudo systemctl enable php-fpm
sudo systemctl start php-fpm
```
 
#### Configure PHP
- Open `sudo vi /etc/php.ini`
- set `;cgi.fix_pathinfo=1` to `cgi.fix_pathinfo=0`
- set `date.timezone =` to `date.timezone = "Europe/Istanbul"`
- Open `sudo vi /etc/php-fpm.d/www.conf`
- Find the line that specifies the `listen` parameter, and change to `listen = /var/run/php-fpm/php-fpm.sock`
- set `;listen.owner = nobody` to `listen.owner = nobody`
- set `;listen.group = nobody` to `listen.group = nobody`
- set `user = apache` to `user = nginx`
- set `group = apache` to `group = nginx`
- restart php-fpm `sudo systemctl restart php-fpm`
 
## Composer Install
 
```sh
curl -sS https://getcomposer.org/installer | php
chmod +x composer.phar
mv composer.phar /usr/local/bin/composer
```
 
## Configure Nginx to Process PHP Pages 
- open `sudo vi /etc/nginx/conf.d/default.conf`
- add below nginx configuration
 
```
server {
    listen       80;
    server_name  domainname.com;

    # note that these lines are originally from the "location /" block
    root   /var/www/public;
    index index.php index.html index.htm;
 
    location / {
        try_files $uri $uri/ =404;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```
 
> be careful about **SELINUX**