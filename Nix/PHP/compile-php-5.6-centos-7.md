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

Compiling from source is a good option if you want

- Upgrade to the last version
- Get the last features
- Fix security issues
- Customize/Optimize your installation

### Prerequisites

This tutorial assumes:

- You have some knowledges about GNU/Linux operating system
- You are familiar with console and shell commands
- You have some knowlegde about compilation process


### Download PHP packages

Go to http://php.net site Downloads section and select the PHP-6.X.X version that you want to compile (5.6.12 at time of writing this tutorial), select your nearest mirror and downloat it then upload to your server using **rsycn** or **scp**. Another simple solution is typing in your console from your server

```bash
# wget http://us1.php.net/distributions/php-5.6.12.tar.xz
```

### Unpacking PHP packages

In the previous step we download a php-5.6.12.tar.xz file so we can unpackage it using tar tool

```bash
# tar xJvf php-5.6.12.tar.xz
```

where

**x** = extract,
**J** = handle [xz](https://en.wikipedia.org/wiki/Xz) file format,
**v** = verbose,
**f** = file

### Installing GNU Compiler Collection and essential tools

```bash
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

[php-build.sh](https://raw.githubusercontent.com/yoander/sysadmin/master/shscript/php-build.sh) script is helper for PHP compilation process. Enable the most used extensions as: curl, openssl, intl, mysql, pcre, ... and allows to install PHP in custom dir, offers options to compile PHP with Apache (prefork or worker) or fpm support. 

You can modify [php-build.sh](https://raw.githubusercontent.com/yoander/sysadmin/master/shscript/php-build.sh) according your neeeded.

If you compile PHP with fpm support you must edit [php-build.sh](https://raw.githubusercontent.com/yoander/sysadmin/master/shscript/php-build.sh) and set the user and group under the fpm process will be run

``` bash
# wget https://raw.githubusercontent.com/yoander/sysadmin/master/shscript/php-build.sh && \
chmod a+x php-build.sh
```

### Build PHP with fpm support and systemd integration

PHP-FPM (FastCGI Process Manager) is an alternative FastCGI implementation for PHP, bundled with the official PHP distribution since version 5.3.3. Adjust fpm values according your needed

systemd is replacement for SysV initialization. systemd also is a suite of system management daemons, libraries, and utilities designed as a central management and configuration for GNU/Linux operating system.

First create necessary DIR that will be used during compilation process

```bash
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

where **-f** = fpm support, **s** = systemd integration

### Install

```bash
#  cd php-5.6.12 && make install
```

### Creating configuration file

PHP source code comes with 2 ini files versions: development and production how we're compiling for PROD server we type the following command. Adjust the ini directives according your needed

```bash
# cp -v php.ini-production /etc/php/php.ini
```

### Creating PHP-FPM configuration file

```bash
# cp -pv /etc/php/php-fpm.conf.default /etc/php/php-fpm.conf
```

### Enable OpCache

OPcache improves PHP performance by storing precompiled script bytecode in shared memory serving request in fastest way.

```bash
# echo "zend_extension=opcache.so" > /etc/php/conf.d/20-opcache.ini
```

### Creating PHP-FPM init service

```bash
# cp -v ./sapi/fpm/php-fpm.service /usr/lib/systemd/system/php-fpm.service
```

If the user and group under the fpm process will be run does not exists then you must created it before start php-fpm service.

```bash
# groupadd --system www-data && \ 
useradd --system -m -d /var/www  -s /usr/sbin/nologin -g www-data www-data
```

Above command create the system group and system user: www-data where:

- **-m** = create home DIR if not exsist
- **-d** = user home DIR
- **-s** = login shell (no login shell for this case because www-data was not created for login purpose)
- **-g** = what groups this user belongs to. 

For security reasons we will change the owner, group and perms for /var/www

```bash
# chown root:root -c /var/www/ && chmod 755  /var/www/ 
```
### Start php-fpm service at system boot time
```bash
# systemctl enable php-fpm
```

### Start php-fpm service
```bash
# systemctl start php-fpm
```

### Check fpm-service status
```bash
# systemctl status php-fpm
``` 

### NGINX

Nginx is a web server with excelent performance and low memory usage also it can be used as a reverse proxy server for HTTP, HTTPS, SMTP, POP3, and IMAP protocols as well as a load balancer and an HTTP cache. For this example we will used NGINX as reverse proxy for PHP-FPM service


#### Installing NGINX

NGINX is available in [EPEL](https://fedoraproject.org/wiki/EPEL) repo

```bash
# yum -y install epel-release && yum -y install nginx
```

#### Autostart NGINX service at system boot time
```bash
# systemctl enable nginx
```

#### Start NGINX service

```bash
# systemctl start nginx
```

#### Check NGINX status service
```bash
# systemctl status nginx
```

## Opening port 80 (nginx is listen on) and 9000 (php-fpm is listen on)

[FirewallD](https://fedoraproject.org/wiki/FirewallD) is the default firewall in CentOS 7.

Adding IP range that will access to php-fpm service in trusted zone. For this example we consider entire LAN has access to php-fpm service

```bash
#  firewall-cmd --permanent --zone=trusted --add-source=192.168.0.0/24
```

Reload firewall rules

```bash
#   firewall-cmd --reload
```
