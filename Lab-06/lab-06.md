### on both
sudo yum -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum update -y && yum -y install vim bash-completion wget postgresql13 postgresql13-server

sudo /usr/pgsql-13/bin/postgresql-13-setup initdb

systemctl enable postgresql-13 && systemctl start postgresql-13
systemctl status postgresql-13

chmod 666 ~/.ssh/authorized_keys

sudo groupadd rebrainssh
sudo useradd -m -g rebrainssh -d /home/rebrainssh -s /bin/bash -c "rebrainssh allow " rebrainssh

su postgres

ssh-keygen -t rsa -P ""
cat ~/.ssh/id_rsa.pub 
scp ~/.ssh/id_rsa.pub postgres@master_or_slave:~/.ssh/authorized_keys

restorecon -Rv ~/.ssh

### on master

su postgres
    vi ~/13/data/pg_hba.conf
#slavedb
host    replication     postgres        0.0.0.0/0               trust

    vi ~/13/data/postgresql.conf
listen_addresses = '*' 
wal_level = replica 
max_wal_senders = 2 
max_replication_slots = 1 
wal_keep_size = 100
archive_mode = on
hot_standby = on

$ psql
CREATE DATABASE task06;
\c task06;
create table test (id int not null primary key, name varchar (20));
insert into test(id, name) values('1', 'test1');

### on slave
su postgres
    vi ~/13/data/pg_hba.conf
#slavedb
host    replication     postgres        0.0.0.0/0               trust

pg_basebackup -h 159.89.8.177 -U postgres -p 5432 -D ~/13/backups/ -Fp -Xs -P -R

cat ~/13/backups/postgresql.auto.conf #For check purpose

mv ~/13/data/ ~/13/data_orig
mv ~/13/backups/ ~/13/data/