# nextcloud

### Install packages
apt update   
apt upgrade  
```sh
apt install \
apt-transport-https bash-completion bzip2 ca-certificates cron curl dialog \
dirmngr ffmpeg ghostscript git gpg gnupg gnupg2 htop jq libfile-fcntllock-perl \
libfontconfig1 libfuse2 locate lsb-release net-tools rsyslog screen smbclient \
socat software-properties-common ssl-cert tree ubuntu-keyring unzip wget zip
```
systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target  
reboot now  
mkdir -p /var/www /var/nc_data  
chown -R www-data:www-data /var/nc_data /var/www  

### Nextcloud download
```sh
wget https://download.nextcloud.com/server/releases/latest.zip
unzip latest.zip
mv nextcloud/ /var/www/html/
chown -R www-data:www-data /var/www/html/nextcloud
```

### nextcloud configuration
```sh
sudo -u www-data php /var/www/html/nextcloud/occ maintenance:install --database "mysql" --database-name "nextcloud" --database-user "nextcloud" --database-pass "nextcloud" --admin-user "Nextcloud-Admin" --admin-pass "Nextcloud-Admin-Passswort" --data-dir "/var/nc_data"
```
config.php korrigieren – fügen Sie die Zeilen hinzu:

sudo -u www-data nano /var/www/html/nextcloud/config/config.php
```sh
[...]
),
'datadirectory' => '/var/nc_data',
'overwrite.cli.url' => 'https://ihre.domain.de/',
'htaccess.RewriteBase' => '/',
[...]
Nextcloud .htaccess korrigieren
sudo -u www-data php /var/www/html/nextcloud/occ maintenance:update:htaccess
Nextcloud-Anpassungen
mkdir -p /var/log/nextcloud/
chown -R www-data:www-data /var/log/nextcloud
sudo -u www-data cp /var/www/html/nextcloud/config/config.php /var/www/html/nextcloud/config/config.php.bak
sudo -u www-data sed -i 's/^[ ]*//' /var/www/html/nextcloud/config/config.php
sudo -u www-data sed -i '/);/d' /var/www/html/nextcloud/config/config.php
```
```sh
sudo -u www-data cat <<EOF >>/var/www/html/nextcloud/config/config.php
'activity_expire_days' => 14,
'allow_local_remote_servers' => true,
'auth.bruteforce.protection.enabled' => true,
'blacklisted_files' => 
array (
0 => '.htaccess',
1 => 'Thumbs.db',
2 => 'thumbs.db',
),
'cron_log' => true,
'default_phone_region' => 'DE',
'defaultapp' => 'files,dashboard',
'enable_previews' => true,
'enabledPreviewProviders' => 
array (
0 => 'OC\Preview\PNG',
1 => 'OC\Preview\JPEG',
2 => 'OC\Preview\GIF',
3 => 'OC\Preview\BMP',
4 => 'OC\Preview\XBitmap',
5 => 'OC\Preview\Movie',
6 => 'OC\Preview\PDF',
7 => 'OC\Preview\MP3',
8 => 'OC\Preview\TXT',
9 => 'OC\Preview\MarkDown',
),
'filesystem_check_changes' => 0,
'filelocking.enabled' => 'true',
'htaccess.RewriteBase' => '/',
'integrity.check.disabled' => false,
'knowledgebaseenabled' => false,
'logfile' => '/var/log/nextcloud/nextcloud.log',
'loglevel' => 2,
'logtimezone' => 'Europe/Berlin',
'log_rotate_size' => '104857600',
'maintenance' => false,
'maintenance_window_start' => 1,
'memcache.local' => '\OC\Memcache\APCu',
'memcache.locking' => '\OC\Memcache\Redis',
'overwriteprotocol' => 'https',
'preview_max_x' => 1024,
'preview_max_y' => 768,
'preview_max_scale_factor' => 1,
'profile.enabled' => false,
'redis' => 
array (
'host' => '/var/run/redis/redis-server.sock',
'port' => 0,
'timeout' => 0.5,
'dbindex' => 1,
),
'quota_include_external_storage' => false,
'share_folder' => '/Freigaben',
'skeletondirectory' => '',
'theme' => '',
'trashbin_retention_obligation' => 'auto, 7',
'updater.release.channel' => 'stable',
);
EOF
```
```sh
sudo -u www-data php /var/www/html/nextcloud occ config:system:set remember_login_cookie_lifetime --value="1800"
sudo -u www-data php /var/www/html/nextcloud occ config:system:set simpleSignUpLink.shown --type=bool --value=false
sudo -u www-data php /var/www/html/nextcloud occ config:system:set versions_retention_obligation --value="auto, 365"
sudo -u www-data php /var/www/html/nextcloud occ config:system:set loglevel --value=2
sudo -u www-data php /var/www/html/nextcloud/occ config:system:set trusted_domains 1 --value=ihre.domain.de
sudo -u www-data php /var/www/html/nextcloud/occ config:app:set settings profile_enabled_by_default --value="0"

### Optional Nextcloud Office (bitte haben Sie Geduld – Download von ca. ~ 400MB):
```sh
sudo -u www-data /usr/bin/php /var/www/html/nextcloud/occ app:install richdocuments
sudo -u www-data /usr/bin/php /var/www/html/nextcloud/occ app:install richdocumentscode
```
```sh
a2enmod ssl
a2ensite 001-nextcloud.conf 001-nextcloud-le-ssl.conf
```
systemctl restart php8.1-fpm.service redis-server.service apache2.service

### Cronjob
crontab -u www-data -e

*/5 * * * * php -f /var/www/html/nextcloud/cron.php > /dev/null 2>&1

sudo -u www-data php /var/www/html/nextcloud/occ background:cron

### security
```sh
a2dismod status
nano /etc/apache2/conf-available/security.conf
[...]
ServerTokens Prod
[...]
ServerSignature Off
[...]
TraceEnable Off
[...]
systemctl restart php8.1-fpm.service redis-server.service apache2.service
```

### fail2ban install
```
apt install -y fail2ban ufw
touch /etc/fail2ban/filter.d/nextcloud.conf
cat <<EOF >/etc/fail2ban/filter.d/nextcloud.conf
[Definition]
_groupsre = (?:(?:,?\s*"\w+":(?:"[^"]+"|\w+))*)
failregex = ^\{%(_groupsre)s,?\s*"remoteAddr":"<HOST>"%(_groupsre)s,?\s*"message":"Login failed:
            ^\{%(_groupsre)s,?\s*"remoteAddr":"<HOST>"%(_groupsre)s,?\s*"message":"Trusted domain error.
datepattern = ,?\s*"time"\s*:\s*"%%Y-%%m-%%d[T ]%%H:%%M:%%S(%%z)?"
EOF
```

nano /etc/fail2ban/jail.d/nextcloud.local  
```sh
[nextcloud]
backend = auto
enabled = true
port = 80,443
protocol = tcp
filter = nextcloud
maxretry = 5
bantime = 3600
findtime = 36000
logpath = /var/log/nextcloud/nextcloud.log
service fail2ban restart
```

### iptables and ipset block lists
