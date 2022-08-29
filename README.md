# MySQL InnoDB Cluster Deployment

## 1. Create databases (2nd and 3rd nodes)
Prepare directories
```
mkdir -p /home/opc/db/3307/data /home/opc/db/3307/innodb_data_home_dir /home/opc/db/3307/innodb_undo_directory /home/opc/db/3307/innodb_temp_tablespace_dir /home/opc/db/3307/innodb_temp_data_file_path /home/opc/db/3307/innodb_log_group_home_dir /home/opc/db/3307/log_bin

mkdir -p /home/opc/db/3308/data /home/opc/db/3308/innodb_data_home_dir /home/opc/db/3308/innodb_undo_directory /home/opc/db/3308/innodb_temp_tablespace_dir /home/opc/db/3308/innodb_temp_data_file_path /home/opc/db/3308/innodb_log_group_home_dir /home/opc/db/3308/log_bin
```
Create option file / configuration file for the database with the following content (command: vi /home/opc/db/3307/my.cnf)
```
[mysqld]
datadir=/home/opc/db/3307/data
binlog-format=ROW
log-bin=/home/opc/db/3307/log_bin/bin
innodb_data_home_dir=/home/opc/db/3307/innodb_data_home_dir
innodb_undo_directory=/home/opc/db/3307/innodb_undo_directory
innodb_temp_tablespaces_dir=/home/opc/db/3307/innodb_temp_tablespace_dir 
innodb_temp_data_file_path=/home/opc/db/3307/innodb_temp_data_file_path/ibtmp1:12M:autoextend
innodb_log_group_home_dir=/home/opc/db/3307/innodb_log_group_home_dir
port=3307
server_id=20
socket=/home/opc/db/3307/data/mysqld.sock
log-error=/home/opc/db/3307/data/mysqld.log
enforce_gtid_consistency = ON
gtid_mode = ON
log_slave_updates = ON
innodb_buffer_pool_size=1G
innodb_buffer_pool_instances=1
innodb_log_file_size=1G
innodb_log_files_in_group=3
innodb_flush_log_at_trx_commit=1
```
Create option file / configuration file for the database with the following content (command: vi /home/opc/db/3308/my.cnf)
```
[mysqld]
datadir=/home/opc/db/3308/data
binlog-format=ROW
log-bin=/home/opc/db/3308/log_bin/bin
innodb_data_home_dir=/home/opc/db/3308/innodb_data_home_dir
innodb_undo_directory=/home/opc/db/3308/innodb_undo_directory
innodb_temp_tablespaces_dir=/home/opc/db/3308/innodb_temp_tablespace_dir 
innodb_temp_data_file_path=/home/opc/db/3308/innodb_temp_data_file_path/ibtmp1:12M:autoextend
innodb_log_group_home_dir=/home/opc/db/3308/innodb_log_group_home_dir
port=3308
server_id=30
socket=/home/opc/db/3308/data/mysqld.sock
log-error=/home/opc/db/3308/data/mysqld.log
enforce_gtid_consistency = ON
gtid_mode = ON
log_slave_updates = ON
innodb_buffer_pool_size=1G
innodb_buffer_pool_instances=1
innodb_log_file_size=1G
innodb_log_files_in_group=3
innodb_flush_log_at_trx_commit=1
```
Create database with empty root password
```
mysqld --defaults-file=/home/opc/db/3307/my.cnf --initialize-insecure
mysqld --defaults-file=/home/opc/db/3308/my.cnf --initialize-insecure
```
Start database
```
mysqld_safe --defaults-file=/home/opc/db/3307/my.cnf &
mysqld_safe --defaults-file=/home/opc/db/3308/my.cnf &
```
## 2. Create InnoDB Cluster:
Run configure Instance on all 3 databases
```
mysqlsh -- dba configure-instance { --host=127.0.0.1 --port=3306 --user=root --instance-password=root } --clusterAdmin=gradmin --clusterAdminPassword='grpass' --interactive=false --restart=true

mysqlsh -- dba configure-instance { --host=127.0.0.1 --port=3312 --user=root } --clusterAdmin=gradmin --clusterAdminPassword='grpass' --interactive=false --restart=true

mysqlsh -- dba configure-instance { --host=127.0.0.1 --port=3313 --user=root } --clusterAdmin=gradmin --clusterAdminPassword='grpass' --interactive=false --restart=true
```
Create cluster on 3306
```
mysqlsh gradmin:grpass@localhost:3306 -- dba createCluster mycluster --consistency=BEFORE_ON_PRIMARY_FAILOVER
```
Add instance 3307 into the cluster using "clone"
```
mysqlsh gradmin:grpass@localhost:3306 -- cluster add-instance gradmin:grpass@localhost:3307 --recoveryMethod=clone
```
Add instance 3308 into the cluster using "clone"
```
mysqlsh gradmin:grpass@localhost:3306 -- cluster add-instance gradmin:grpass@localhost:3308 --recoveryMethod=clone
```
Check cluster status
```
mysqlsh gradmin:grpass@localhost:3306 -- cluster status
```
## Install MySQL Router
```
mysqlrouter --bootstrap gradmin:grpass@localhost:3306 --directory router --account myrouter --account-create always --force

router/start.sh
```


