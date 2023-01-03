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


### iptables and ipset block lists
