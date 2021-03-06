# How to Compile PHP-5.6 on Centos 7

### Introduction

PHP is a general purpose scripting language that it's mainly used for web development. Today heavy load web sites such as: [Facebook](https://www.facebook.com) and [Wikipedia](https://www.wikipedia.org) are powered by PHP. If you are new in the PHP world and you are looking a programming language for your project you will find.

**Community features like:**
- Active development
- Great community
- Good documentation, books and blogs
 
**Languages features like:**
- Database drivers: MySQL, PostgreSQL, SQLite, Firebird, MSSQL, SQL Server ...
- Namespaces
- Object Oriented Programming
- Reflection
- File handlers
- Text processing (PCRE)
- Error handling
- Exception handling
- Generators
- Web scraping
- Variadic functions
- Argument unpacking
- Large file uploads
- Low learning curve and more
 
### Why compile from source

Compiling from source is a good option if you want to:

- Upgrade to the last version
- Get the last features
- Fix security issues
- Customize/Optimize your installation

### Goals

After finishing this tutorial you will be able to compile PHP by yourself and get some knowledge about interaction between NGINX and PHP-FPM service.

### Prerequisites

This tutorial assumes:

- You have some knowledges about GNU/Linux operating system
- You are familiar with console and shell commands
- You have some knowlegde about compilation process


### Download PHP packages

Go to http://php.net site Downloads section and select the PHP-6.X.X version that you want to compile (5.6.12 at time of writing this tutorial), select your nearest mirror and downloat it then upload to your server using **rsycn** or **scp**. Another simple solution is typing in your console from your server

```
# wget http://us1.php.net/distributions/php-5.6.12.tar.xz
```

### Unpacking PHP packages

In the previous step we download a php-5.6.12.tar.xz file so we can unpackage it using tar tool

```
# tar xJvf php-5.6.12.tar.xz
```

where

**x** = extract,
**J** = handle [xz](https://en.wikipedia.org/wiki/Xz) file format,
**v** = verbose,
**f** = file

### Installing GNU Compiler Collection and essential tools

```
# yum -y install gcc gcc-c++ make automake autoconf bison flex libtool libstdc++-devel
```

### Installing dependencies

We will compile PHP with XML, SSL, PCRE, SQLite, bzip2, curl, GD, mcrypt ... support. mcrypt is available in [EPEL](https://fedoraproject.org/wiki/EPEL) repo then you need to type

```
# yum -y install epel-release
```

After

```
# yum -y install libxml2-devel openssl openssl-devel pcre-devel sqlite-devel bzip2-devel \
libcurl-devel libicu-devel gd-devel readline-devel libmcrypt-devel systemd-devel
```

### Download [php-build.sh](https://raw.githubusercontent.com/yoander/sysadmin/master/shscript/php-build.sh) script from Github

[php-build.sh](https://raw.githubusercontent.com/yoander/sysadmin/master/shscript/php-build.sh) script is helper for PHP compilation process. Enable the most used extensions as: curl, openssl, intl, mysql, pcre, ... and it allows to install PHP in custom DIR, it offers options to compile PHP with Apache (prefork or worker) or fpm support. 

You can modify [php-build.sh](https://raw.githubusercontent.com/yoander/sysadmin/master/shscript/php-build.sh) according your needs.

If you compile PHP with fpm support you must edit [php-build.sh](https://raw.githubusercontent.com/yoander/sysadmin/master/shscript/php-build.sh) and set the user and group under the fpm process will be run

```
# wget https://raw.githubusercontent.com/yoander/sysadmin/master/shscript/php-build.sh && \
chmod a+x php-build.sh
```

### Build PHP with FPM support and systemd integration

PHP-FPM (FastCGI Process Manager) is an alternative FastCGI implementation for PHP, bundled with the official PHP distribution since version 5.3.3.

systemd is replacement for SysV initialization. systemd also is a suite of system management daemons, libraries, and utilities designed as a central management and configuration for GNU/Linux operating system.

First create necessary DIR that will be used during compilation process.

```
# mkdir -p /etc/php/conf.d /usr/lib/php/modules /usr/share/pear
```

Create PCRE symbolic link to avoid: "configure: error: Could not find libpcre.(a|so) in /usr"

```
# ln -s /usr/lib64/libpcre.so.1.2.0 /usr/lib/libpcre.so
```

Then

```
# ./php-build.sh -fs php-5.6.12
```

where **-f** = FPM support, **s** = systemd integration

### Install

```
#  cd php-5.6.12 && make install
```

### Creating configuration file

PHP source code comes with 2 .ini files versions: development and production. As we're compiling for PROD server we type the following command.

```
# cp -v php.ini-production /etc/php/php.ini
```

Adjust the ini directives according your needs.

### Creating PHP-FPM configuration file

```
# cp -pv /etc/php/php-fpm.conf.default /etc/php/php-fpm.conf
```

### Edit /etc/php/php-fpm.conf

To allow php-fpm listen on every interface you must look for the listen directive and set its value. Ex

```
listen 9000
```

Restrict access to php-fpm service. Only specific IPs can connect to php-fpm service, look for listen.allowed_clients directive and set its value to

```
listen.allowed_clients = 127.0.0.1,192.168.0.106,192.168.0.107,192.168.0.108
```

### Enable OpCache

OPcache improves PHP performance by storing precompiled script bytecode in shared memory serving request in fastest way.

```
# echo "zend_extension=opcache.so" > /etc/php/conf.d/20-opcache.ini
```

### Creating PHP-FPM init service

```
# cp -v ./sapi/fpm/php-fpm.service /usr/lib/systemd/system/php-fpm.service
```

If the user and group under the php-fpm process will be run does not exists then you must created them before start php-fpm service.

```
# groupadd --system www-data && \ 
useradd --system -m -d /var/www  -s /usr/sbin/nologin -g www-data www-data
```

Above command create the system group and system user www-data where:

- **-m** = create home DIR if not exsist
- **-d** = user home DIR
- **-s** = login shell (no login shell for this case because www-data was not created for login purpose)
- **-g** = what groups this user belongs to. 

For security reasons we will change the owner, group and perms for /var/www

```
# chown root:root -c /var/www/ && chmod 755  /var/www/ 
```

### Start PHP-FPM service at system boot time
```
# systemctl enable php-fpm
```

### Start PHP-FPM service
```
# systemctl start php-fpm
```

### Check PHP-FPM status
```
# systemctl status php-fpm
``` 

### Create info.php file
```
# echo '<?php phpinfo();'>/var/www/info.php
```

### NGINX

NGINX is a web server with excelent performance and low memory footprint also it can be used as a reverse proxy server for HTTP, HTTPS, SMTP, POP3, and IMAP protocols as well as load balancer. For this example we will used NGINX as reverse proxy for testing our PHP-FPM service.

#### Installing NGINX

NGINX is available through [EPEL](https://fedoraproject.org/wiki/EPEL) repo

```
# yum -y install epel-release && yum -y install nginx
```

#### Edit /etc/nginx/nginx.conf

Look for server section and add the following section

```nginx
location ~ \.php$ {
            root         /var/www;
            fastcgi_split_path_info ^(.+.php)(/.+)$;
            fastcgi_pass 192.168.0.104:9000;
            fastcgi_index index.php;
            include fastcgi.conf;
	}
```

192.168.0.104 is the IP where php-fpm service is listen on

#### Start NGINX service at system boot time
```
# systemctl enable nginx
```

#### Start NGINX service

```
# systemctl start nginx
```

#### Check NGINX status service
```
# systemctl status nginx
```

### Opening ports 80 (NGINX) and 9000 (php-fpm)

[FirewallD](https://fedoraproject.org/wiki/FirewallD) is the default firewall in CentOS 7.

Adding IP range that will access to php-fpm service in trusted zone. For this example we consider entire LAN has access to php-fpm service.

```
#  firewall-cmd --permanent --zone=trusted --add-source=192.168.0.0/24
```

Reload firewall rules

```
# firewall-cmd --reload
```

### Testing

Open your browser with the IP where is listening your NGINX server

```
$ firefox http://192.168.0.106/info.php
```

### Conclusion

Now that you have learned how to compile PHP by yourself you can get all advantages (bug fixes, performance improvements and lastest features) for each version of PHP without waiting for the chosen distribution packager.
