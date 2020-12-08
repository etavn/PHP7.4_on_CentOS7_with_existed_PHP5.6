# How to install PHP7.4 version co-exist with PHP5.6 on CentOS7
## Install prerequisites (Let's skip when you installed)
Install Apache
```
yum install httpd
```

Install Epel Release and Yum Utils
```
yum install -y epel-release && \
yum install -y yum-utils
```

You must add and activate the REMI repository to install PHP 7
```
yum install -y http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

## Install PHP7.4 Version
Install PHP7.4
```
yum install -y php74
```

Install PHP7.4-FPM
```
yum install -y php74-php-fpm
```
`FastCGI Process Manager (PHP-FPM) is an alternative FastCGI daemon for PHP that allows a website to handle high loads`

Change port default FPM
```
sed -i 's/:9000/:9074/' /etc/opt/remi/php74/php-fpm.d/www.conf
```
## Setting FPM
Disable path_info. [Why?](http://kaiwangchen.github.io/2012/10/02/understand-the-cgi-fix_pathinfo-security-issue.html)
```
echo "cgi.fix_pathinfo=0" >> /etc/opt/remi/php74/php.ini
```

For performance reasons, we will use a UNIX socket rather than a TCP socket for communications between Apache and PHP-FPM services. Change setting listen FPM
```
sed -i "s#listen = 127.0.0.1:9074#listen = /run/php74/php74-fpm.sock#" /etc/opt/remi/php74/php-fpm.d/www.conf
```

Create sock locate for FPM
```
mkdir /run/php74
```

Setting permissions
```
echo "listen.owner = apache" >> /etc/opt/remi/php74/php-fpm.d/www.conf && \
echo "listen.group = apache" >> /etc/opt/remi/php74/php-fpm.d/www.conf && \
echo "listen.mode = 0660" >> /etc/opt/remi/php74/php-fpm.d/www.conf
```

Restart PHP74-FPM
```
systemctl start php74-php-fpm
```

## Config VirtualHost Apache
You should to declare Proxy when create a new virtual host config, and then redirect the matching to proxy
```
# Proxy declaration
<Proxy "unix:/run/php74/php74-fpm.sock|fcgi://php74-fpm">
        ProxySet disablereuse=off
</Proxy>

...

# Redirect to the proxy
<FilesMatch \.php$>
        SetHandler proxy:fcgi://php74-fpm
</FilesMatch>
```

**Full Example**
```
<VirtualHost *:80>
        ServerName php74.local
        DocumentRoot /var/www/php74

        # Proxy declaration
        <Proxy "unix:/run/php74/php74-fpm.sock|fcgi://php74-fpm">
                ProxySet disablereuse=off
        </Proxy>

        # Redirect to the proxy
        <FilesMatch \.php$>
                SetHandler proxy:fcgi://php74-fpm
        </FilesMatch>
</VirtualHost>
```

## Commands useful
```
apachectl configtest # Test config Apache
systemctl stop php74-php-fpm
systemctl start php74-php-fpm
systemctl restart php74-php-fpm
```

## Notice
Setting permission Apache for source code directories  
Ex:
```
chown -R apache:apache /var/www/php74
```
