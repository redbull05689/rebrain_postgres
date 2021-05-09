scp rebrain_courses_db.sql root@68.183.70.114:/tmp

sudo apt update && apt -y install vim bash-completion wget
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
sudo apt update && apt install postgresql-12 postgresql-client-12 -y

systemctl enable postgresql
systemctl status postgresql


CREATE USER root WITH SUPERUSER PASSWORD 'postgres';
CREATE USER extuser WITH SUPERUSER PASSWORD 'postgres';
ALTER ROLE extuser SET client_encoding TO 'UTF8';
ALTER ROLE extuser SET default_transaction_isolation TO 'read committed';
ALTER ROLE extuser SET timezone TO 'UTC';

CREATE DATABASE rebrain OWNER extuser;

#sudo apt install postgresql-12-cron -y

#vim /etc/postgresql/12/main/postgresql.conf
    shared_preload_libraries = 'pg_cron'
    cron.database_name='rebrain'

#systemctl restart postgresql

\c rebrain
CREATE EXTENSION pg_cron;

GRANT USAGE ON SCHEMA cron TO extuser;

INSERT INTO cron.job (schedule , command, nodename, nodeport, database , username) VALUES ('0 2 * * *', 'VACUUM', 'localhost', 5432 , 'rebrain', 'extuser');

#Check results
\dx
SELECT * FROM cron.job;