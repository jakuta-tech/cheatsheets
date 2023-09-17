### Create new user and password
```
htpasswd -c /etc/apache2/.htpasswd <user>
```

### Basic authentication
```
vim /etc/apache2/apache2.conf

<Directory /var/www/html>
    AuthType Basic
    AuthName "Restricted Content"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Directory>
```

### Redirect to port 443
```
<VirtualHost *:80>
    ServerName <domain>

    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteCond %{REQUEST_METHOD} ^TRACE
        RewriteRule .* - [F]
        RewriteCond %{REQUEST_METHOD} ^TRACK
        RewriteRule .* - [F]

        #redirect all port 80 traffic to 443
        RewriteCond %{SERVER_PORT} !^443$
        RewriteRule ^/?(.*) https:/<domain>/$1 [L,R]
    </IfModule>
</VirtualHost>
```

### Filter User-Agents
```
vim /etc/apache2/apache2.conf

<Directory /var/www/html>
    RewriteEngine on
    RewriteCond %{HTTP_USER_AGENT} "Windows Installer"
    RewriteRule ^.*$ %{request_filename} [L]
    RewriteRule ^ - [F,L]
</Directory>
```

### Disable packages
```
a2disconf <package>
```

### Prevent clickjacking
```
a2enmod headers
sed -i '$ a\Header always append X-Frame-Options DENY' /etc/apache2/apache2.conf
```

### Remove apache version
```
vim /etc/apache2/apache2.conf

Header unset Server
ServerSignature Off
ServerTokens Prod
```

### Install PHP
```
apt-get install apache2 php libapache2-mod-php
```

### Load Balancer
```
a2ensite forward_proxy.conf

vim /etc/apache2/mods-enabled/proxy.conf

ProxyRequests On
<Proxy *>
   AddDefaultCharset off
   Require all denied
   #Require local
</Proxy>


vim /etc/apache2/sites-available/forward_proxy.conf

<VirtualHost *:80>
    <Proxy balancer://someName>
        BalancerMember http://<rhost>:8080 route=worker1
        BalancerMember http://<rhost>:8080 route=worker2
    </Proxy>

    ProxyRequests On
    ProxyVia On
    <Proxy "*">
        Require ip 192.168
    </Proxy>
    ErrorLog ${APACHE_LOG_DIR}/error_forward_proxy.log
    CustomLog ${APACHE_LOG_DIR}/access_forward_proxy.log combined
</VirtualHost>
```

