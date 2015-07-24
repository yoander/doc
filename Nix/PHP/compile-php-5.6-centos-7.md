# How to Compile PHP-5.6 on Centos 7

PHP is a general purpose scripting language but it's mainly used for web development. Today heavy load web sites such as: Facebook and Wikipedia are powered by PHP. If you are new in the PHP world and you are looking a programming language for your project you will find.

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
$ wget http://us1.php.net/distributions/php-5.6.11.tar.xz
```

## Unpacking PHP packages

In the previous step we download a tar.xz file so we can unpackage it using tar tool

```bash
$ tar xJvf php-5.6.11.tar.xz
```

