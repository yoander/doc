# How to Compile PHP-5.6 on Centos 7

PHP is a general purpose scripting language but it's mainly used for web development. Today heavy load web sites such as: [Facebook](https://www.facebook.com) and [Wikipedia](https://www.wikipedia.org) are powered by PHP. If you are new in the PHP world and you are looking a programming language for your project you will find.

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
- Low learning curve

If you want to get the last features from PHP such us: variadic functions, argument unpacking, large file uploads and more then you need to compile by yourself because mayor distributions like CentOS, Debian, Ubuntu never has the las version of PHP.

## Download PHP packages

Go to http://php.net site Downloads section and select the PHP-6.X.X version that you want to compile (5.6.11 at time of writing this articule), select your nearest mirror and downloat it then upload to your server using **rsycn** or **scp**. Another simple solution is typing in your console from your server

```bash
# wget http://us1.php.net/distributions/php-5.6.11.tar.xz
```

## Unpacking PHP packages

In the previous step we download a php-5.6.11.tar.xz file so we can unpackage it using tar tool

```bash
# tar xJvf php-5.6.11.tar.xz
```
where

**x** = extract,
**J** = handle [xz](https://en.wikipedia.org/wiki/Xz) file format,
**v** = verbose,
**f** = file

## Installing GNU Compiler Collection and essential tools for compiling process
```bash
# yum -y install gcc make automake autoconf bison flex libtool libstdc++-devel
```

## Installing dependencies

We will compile PHP with XML, SSL, PCRE, SQLite, bzip2, curl, GD, mcrypt ... support. mcrypt is available in [EPEL](https://fedoraproject.org/wiki/EPEL) repo then you need to type

```
# yum install epel-release
```

After

```
# yum -y libxml2-devel openssl openssl-devel pcre-devel sqlite-devel bzip2-devel\
libcurl-devel libicu-devel gd-devel readline-devel libmcrypt-devel
```

## Download php-build.sh script from [Github](https://raw.githubusercontent.com/yoander/sysadmin/master/shscript/php-build.sh)

php-build.sh script is helper for PHP compilation process. Enable the most used extensions as: curl, openssl, intl, mysql, pcre, ... and allows to install PHP in custom dir, offers options to compile PHP with Apache (prefork or worker) or fpm support. If you compile PHP with fpm support you must edit php-build.sh and set the user and group under the fpm process will be run

``` bash
# wget https://raw.githubusercontent.com/yoander/sysadmin/master/shscript/php-build.sh
```

## Build PHP with fpm support

First create necessary DIR that will be used during compilation processs

```bash
# mkdir -p /etc/php /etc/php/conf.d /usr/lib/php/modules /usr/share/pear
```

Then

```
# ./php-build.sh -f php-5.6.11
```

Script will notify you must c
