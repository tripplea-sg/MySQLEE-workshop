# InnoDB Cluster / Replicaset Workshop

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
mysqlbackup --user=root --host=127.0.0.1 --port=3311 --datadir=/home/opc/mysql-sandboxes/3311/sandboxdata --backup-dir=/home/opc/backup/MEB --with-timestamp backup-and-apply-log

# check backup
ls /home/opc/backup/MEB

# check binlog position for point-in-time recovery
cat /home/opc/backup/MEB/<directory>/meta/backup_variables.txt
```

## Restore backup to instance 3312 using MEB
```
# stop instance 3312
mysqlsh -e "dba.stopSandboxInstance(3312)"

# delete all database files (instance 3312)
rm -Rf /home/opc/mysql-sandboxes/3312/sandboxdata/*

# Restore backup from backup location
mysqlbackup --defaults-file=/home/opc/mysql-sandboxes/3312/my.cnf --backup-dir=/home/opc/backup/MEB/<directory> --datadir=/home/opc/mysql-sandboxes/3312/sandboxdata copy-back-and-apply-log

# check database files
ls mysql-sandboxes/3312/sandboxdata/

# start instance 3312
mysqlsh -e "dba.startSandboxInstance(3312)"

# check database schema
mysql -uroot -h127.0.0.1 -P3312 -e "show databases"
```
## Create InnoDB Cluster:
Run configure Instance on all 3 databases
```
mysqlsh -- dba configure-instance { --host=127.0.0.1 --port=3311 --user=root } --clusterAdmin=gradmin --clusterAdminPassword='grpass' --interactive=false --restart=true

mysqlsh -- dba configure-instance { --host=127.0.0.1 --port=3312 --user=root } --clusterAdmin=gradmin --clusterAdminPassword='grpass' --interactive=false --restart=true

mysqlsh -- dba configure-instance { --host=127.0.0.1 --port=3313 --user=root } --clusterAdmin=gradmin --clusterAdminPassword='grpass' --interactive=false --restart=true
```
Create cluster on 3311
```
mysqlsh gradmin:grpass@localhost:3311 -- dba createCluster mycluster --consistency=BEFORE_ON_PRIMARY_FAILOVER
```
Add instance 3312 into the cluster using "incremental"
```
mysqlsh gradmin:grpass@localhost:3311 -- cluster add-instance gradmin:grpass@localhost:3312 --recoveryMethod=incremental
```
Add instance 3313 into the cluster using "clone"
```
mysqlsh gradmin:grpass@localhost:3311 -- cluster add-instance gradmin:grpass@localhost:3313 --recoveryMethod=clone
```
Check cluster status
```
mysqlsh gradmin:grpass@localhost:3311 -- cluster status
```
## Install MySQL Router
```
mysqlrouter --bootstrap gradmin:grpass@localhost:3311 --directory router --account myrouter --account-create always --force

router/start.sh
```
## Create InnoDB Replicaset:
Dissolve cluster
```
# stop router
router/stop.sh

# delete router
rm -Rf /home/opc/router

# dissove cluster
mysqlsh gradmin:grpass@localhost:3311 -- cluster dissove

# stop instance 3313
mysqlsh -e "dba.stopSandboxInstance(3313)"

# Create replicaset on 3311
mysqlsh gradmin:grpass@localhost:3311 -- dba createReplicaSet myRs

# Add replicas
mysqlsh gradmin:grpass@localhost:3311 -- rs add-instance localhost:3312

# check replicaset status
mysqlsh gradmin:grpass@localhost:3311 -- rs status
```
Setup router
```
mysqlrouter --bootstrap gradmin:grpass@localhost:3311 --directory router --account myrouter --force
```
start router
```
router/start.sh
```


