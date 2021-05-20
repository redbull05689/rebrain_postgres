sudo apt update && apt -y install vim bash-completion wget
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
sudo apt update && apt install postgresql-11 postgresql-client-11 -y

systemctl enable postgresql && systemctl status postgresql

    publisher:~#
vim /etc/postgresql/11/main/pg_hba.conf
    host replication car_portal_app 0.0.0.0/0 md5

vim /etc/postgresql/11/main/postgresql.conf
    listen_addresses = '*'
    wal_level = logical
    max_replication_slots = 10
    max_wal_senders = 10 


$ psql -f /opt/schema.sql
$ psql -f /opt/data.sql

    psql
CREATE ROLE car_portal_app;
ALTER ROLE car_portal_app REPLICATION LOGIN ;
CREATE PUBLICATION car_portal FOR ALL TABLES;

        subscriber:~#
vim /etc/postgresql/11/main/pg_hba.conf
    host    all             all             ::1/128                 trust
    host    all     car_portal_app  0.0.0.0/0               trust

vim /etc/postgresql/11/main/postgresql.conf
    listen_addresses = '*'

scp root@167.172.189.29:/opt/schema.sql /opt/
    psql
CREATE ROLE car_portal_app;

psql -f /opt/schema.sql
    psql
CREATE SUBSCRIPTION car_portal CONNECTION 'dbname=car_portal host=167.172.189.29 user=car_portal_app' PUBLICATION car_portal;
