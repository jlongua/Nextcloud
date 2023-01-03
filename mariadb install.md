### Install MariaDB 10.8 database
```sh
wget -O- https://mariadb.org/mariadb_release_signing_key.asc \
     | gpg --dearmor | sudo tee /usr/share/keyrings/mariadb-keyring.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/mariadb-keyring.gpg] \
     https://mirror.kumi.systems/mariadb/repo/10.8/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/mariadb.list
```
apt update  
apt install mariadb-server  

mysql_secure_installation  

systemctl stop mariadb  
```sh
mv /etc/mysql/my.cnf /etc/mysql/my.cnf.bak  
```
nano /etc/mysql/my.cnf  
```sh
[client]
default-character-set = utf8mb4
port = 3306
socket = /var/run/mysqld/mysqld.sock
[mysqld_safe]
log_error=/var/log/mysql/mysql_error.log
nice = 0
socket = /var/run/mysqld/mysqld.sock
[mysqld]
basedir = /usr
bind-address = 127.0.0.1
binlog_format = ROW
bulk_insert_buffer_size = 16M
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci
concurrent_insert = 2
connect_timeout = 5
datadir = /var/lib/mysql
default_storage_engine = InnoDB
expire_logs_days = 2
general_log_file = /var/log/mysql/mysql.log
general_log = 0
innodb_buffer_pool_size = 2G
innodb_buffer_pool_instances = 1
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 32M
innodb_max_dirty_pages_pct = 90
innodb_file_per_table = 1
innodb_open_files = 400
innodb_io_capacity = 4000
innodb_flush_method = O_DIRECT
innodb_read_only_compressed=OFF
key_buffer_size = 128M
lc_messages_dir = /usr/share/mysql
lc_messages = en_US
log_bin = /var/log/mysql/mariadb-bin
log_bin_index = /var/log/mysql/mariadb-bin.index
log_error = /var/log/mysql/mysql_error.log
log_slow_verbosity = query_plan
log_warnings = 2
long_query_time = 1
max_allowed_packet = 16M
max_binlog_size = 100M
max_connections = 200
max_heap_table_size = 64M
myisam_recover_options = BACKUP
myisam_sort_buffer_size = 512M
port = 3306
pid-file = /var/run/mysqld/mysqld.pid
query_cache_limit = 2M
query_cache_size = 64M
query_cache_type = 1
query_cache_min_res_unit = 2k
read_buffer_size = 2M
read_rnd_buffer_size = 1M
skip-external-locking
skip-name-resolve
slow_query_log_file = /var/log/mysql/mariadb-slow.log
slow-query-log = 1
socket = /var/run/mysqld/mysqld.sock
sort_buffer_size = 4M
table_open_cache = 400
thread_cache_size = 128
tmp_table_size = 64M
tmpdir = /tmp
transaction_isolation = READ-COMMITTED
#unix_socket=OFF
user = mysql
wait_timeout = 600
[mysqldump]
max_allowed_packet = 16M
quick
quote-names
[isamchk]
key_buffer = 16M
```
### Restart the MariaDB database and create the Nextcloud database and user
systemctl restart mariadb
```sh
mysql -uroot -p -e "CREATE DATABASE nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci; CREATE USER nextcloud@localhost identified by 'nextcloud'; GRANT ALL PRIVILEGES on nextcloud.* to nextcloud@localhost; FLUSH privileges;"

```
