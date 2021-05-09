scp rebrain_courses_db.sql root@188.166.164.183:/tmp

sudo apt update && apt -y install vim bash-completion wget
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
sudo apt update && apt install postgresql-13 postgresql-client-13 -y
systemctl enable postgresql
systemctl status postgresql


#Enable external connection
vim /etc/postgresql/13/main/postgresql.conf 
    listen_addresses = '*'
vim /etc/postgresql/13/main/pg_hba.conf
    host    all             all             0.0.0.0/0            md5

systemctl restart postgresql

psql
    \password postgres

CREATE USER root WITH SUPERUSER PASSWORD 'postgres';
CREATE DATABASE rebrain_courses_db WITH OWNER root;



chmod 777 /tmp/rebrain_courses_db.sql
cp /tmp/rebrain_courses_db.sql /tmp/rebrain_courses_db.sql.bqp
root# psql -U root -d rebrain_courses_db -f /tmp/rebrain_courses_db.sql

CREATE USER rebrain_admin WITH  SUPERUSER PASSWORD 'postgres';
GRANT ALL privileges on database rebrain_courses_db to rebrain_admin;

CREATE ROLE backup;

psql -U root -h 127.0.0.1 -p 5432 rebrain_courses_db

GRANT USAGE ON SCHEMA public TO rebrain_admin;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO backup;

psql -U rebrain_admin -h 127.0.0.1 -p 5432 rebrain_courses_db;

CREATE TABLE blog(
id SERIAL PRIMARY KEY NOT NULL, -- Primary Key
user_id INT NOT NULL, -- Foreign Key to table users
blog_text TEXT NOT NULL,
CONSTRAINT fk_user_id
FOREIGN KEY (user_id)
REFERENCES users(user_id)
);

INSERT INTO blog(user_id,blog_text)
VALUES (1,'We are studying at the REBRAIN PostgreSQL Workshop');

psql -U root -h 127.0.0.1 -p 5432 rebrain_courses_db

CREATE ROLE rebrain_group_select_access;
GRANT USAGE ON SCHEMA public TO rebrain_group_select_access;
ALTER DEFAULT privileges IN SCHEMA public GRANT SELECT ON TABLES TO rebrain_group_select_access;

CREATE ROLE rebrain_user;
GRANT rebrain_group_select_access TO rebrain_user;

CREATE ROLE rebrain_portal;
CREATE SCHEMA rebrain_portal;
GRANT USAGE ON SCHEMA rebrain_portal TO rebrain_portal;

GRANT ALL ON SCHEMA rebrain_portal TO rebrain_portal;