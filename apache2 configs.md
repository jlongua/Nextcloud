### apache2 configuration
```sh
a2enmod rewrite headers env dir mime
```
nano /etc/apache2/mods-available/http2.conf
```sh
# mod_http2 doesn't work with mpm_prefork
<IfModule !mpm_prefork>
Protocols h2 h2c http/1.1
H2Direct on
H2StreamMaxMemSize 5120000000
[...]
```
systemctl restart apache2.service  
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/001-nextcloud.conf  
a2dissite 000-default.conf
```
nano /etc/apache2/sites-available/001-nextcloud.conf
```sh
<VirtualHost *:80>
ServerName nextcloud.plan9.com
ServerAlias nextcloud.plan9.com
ServerAdmin nextcloud.plan9.com
DocumentRoot /var/www/html/nextcloud
ErrorLog /var/log/apache2/error.log
CustomLog /var/log/apache2/access.log combined
RewriteEngine on
RewriteCond %{SERVER_NAME} = nextcloud.plan9.com
RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
```

a2ensite 001-nextcloud.conf
systemctl restart apache2.service

mv /etc/apache2/sites-available/001-nextcloud-le-ssl.conf /etc/apache2/sites-available/001-nextcloud-le-ssl.conf.bak

nano /etc/apache2/sites-available/001-nextcloud-le-ssl.conf
```sh
<IfModule mod_ssl.c>
SSLUseStapling on
SSLStaplingCache shmcb:/var/run/ocsp(128000)
<VirtualHost *:443>
SSLCertificateFile /etc/zersossl/nextcloud.plan9.com/fullchain.pem
SSLCACertificateFile /etc/zerossl/nextcloud.plan9.com/fullchain.pem
SSLCertificateKeyFile /etc/zerossl/nextcloud.plan9.com/privkey.pem

Protocols h2 h2c http/1.1
Header add Strict-Transport-Security: "max-age=15552000;includeSubdomains"
ServerAdmin mail@plan9.com
ServerName nextcloud.plan9.com
ServerAlias nextcloud.plan9.com
SSLEngine on
SSLCompression off
SSLOptions +StrictRequire
SSLProtocol -all +TLSv1.3 +TLSv1.2
SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
SSLHonorCipherOrder off
SSLSessionTickets off
ServerSignature off
SSLStaplingResponderTimeout 5
SSLStaplingReturnResponderErrors off
SSLOpenSSLConfCmd Curves X448:secp521r1:secp384r1:prime256v1
SSLOpenSSLConfCmd ECDHParameters secp384r1
LogLevel warn
CustomLog /var/log/apache2/access.log combined
ErrorLog /var/log/apache2/error.log
DocumentRoot /var/www/html/nextcloud
<Directory /var/www/html/nextcloud/>
Options Indexes FollowSymLinks
AllowOverride All
Require all granted
Satisfy Any
</Directory>
<IfModule mod_dav.c>
Dav off
</IfModule>
<Directory /var/nc_data/>
Require all denied
</Directory>
<Files ".ht*">
Require all denied
</Files>
TraceEnable off
RewriteEngine On
RewriteCond %{REQUEST_METHOD} ^TRACK
RewriteRule .* - [R=405,L]
SetEnv HOME /var/www/html/nextcloud
SetEnv HTTP_HOME /var/www/html/nextcloud
<IfModule mod_reqtimeout.c>
RequestReadTimeout body=0
</IfModule>
</VirtualHost>
</IfModule>
```
openssl dhparam -dsaparam -out /etc/ssl/certs/dhparam.pem 4096  
cat /etc/ssl/certs/dhparam.pem >> /etc/zerossl/nextcloud.plan9.com/fullchain.pem  

nano /etc/apache2/apache2.conf
```sh
ServerName nextcloud.plan9.com
[...]
<Directory /var/www/>
Options FollowSymLinks MultiViews
AllowOverride All
Require all granted
</Directory>
```
systemctl restart apache2.service
