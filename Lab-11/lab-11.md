yum -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm 
yum -y update && yum -y install vim bash-completion wget postgresql13 postgresql13-server postgresql13-contrib

/usr/pgsql-13/bin/postgresql-13-setup initdb

systemctl enable postgresql-13 && systemctl start postgresql-13 && systemctl status postgresql-13

su postgres
psql
CREATE USER root WITH SUPERUSER PASSWORD 'postgres';
CREATE DATABASE pgbench WITH OWNER root;
CREATE DATABASE root WITH OWNER root;

cp /usr/pgsql-13/bin/pgbench /usr/bin

pgbench -i -U root -h localhost pgbench 
pgbench --initialize --scale=100 pgbench

pgbench -t 1000 -c 15 -f /opt/task11.sql -n pgbench > /opt/result_1.txt


su postgres
psql
ALTER SYSTEM RESET ALL;
ALTER SYSTEM SET fsync to off;

systemctl restart postgresql-13


pgbench -t 1000 -c 15 -f /opt/task11.sql -n pgbench > /opt/result_2.txt


su postgres
psql
ALTER SYSTEM RESET ALL;
ALTER SYSTEM SET synchronous_commit to off;
ALTER SYSTEM SET commit_delay to 100000;

pgbench -t 1000 -c 15 -f /opt/task11.sql -n pgbench > /opt/result_3.txt

systemctl restart postgresql-13
su postgres

vi ~/13/data/postgresql.conf
    shared_buffers 256MB
    work_mem 16MB
    max_connections 32