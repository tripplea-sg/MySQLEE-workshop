# InnoDB Cluster Workshop

## Create Databases: 
Do not set root password.
```
mysqlsh -e "dba.deploySandboxInstance(3311)"
mysqlsh -e "dba.deploySandboxInstance(3312)"
mysqlsh -e "dba.deploySandboxInstance(3313)"
```
## Upload data into instance 3311
```
mysql -uroot -h127.0.0.1 -P3311 -e "source sakila-schema.sql"
mysql -uroot -h127.0.0.1 -P3311 -e "source sakila-data.sql"
mysql -uroot -h127.0.0.1 -P3311 -e "source world_x.sql"

# check data
mysql -uroot -h127.0.0.1 -P3311 -e "show databases"

```
## Backup using MySQL Enterprise Backup
Run MEB to backup instance 3311
```
# create backup location
mkdir -p backup/MEB

# Get datadir location
mysql -uroot -h127.0.0.1 -P3311 -e "show variables like 'datadir'"

# run backup
mysqlbackup --user=root --host=127.0.0.1 --port=3311 --datadir=/home/opc/mysql-sandboxes/3311/sandboxdata --backup-dir=/home/opc/backup --with-timestamp backup-and-apply-log

# check backup
ls /home/opc/backup
```
