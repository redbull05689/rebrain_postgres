yum -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm && \
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm && \
yum -y update && yum -y install vim bash-completion wget postgresql13 postgresql13-server postgresql13-contrib

/usr/pgsql-13/bin/postgresql-13-setup initdb

systemctl enable postgresql-13 && systemctl start postgresql-13 && systemctl status postgresql-13

su postgres
psql
CREATE DATABASE task12;
\c task12;
CREATE TABLE pgsql(
     id INT PRIMARY KEY, name TEXT NOT NULL 
);

INSERT INTO pgsql SELECT n , md5 (random()::text) FROM generate_series (1, 100000) AS foo(n);

EXPLAIN SELECT * FROM pgsql;

vim /opt/cost_preview.txt
    cost=0.00..1893.18 rows=105918 width=36

su postgres
psql
\c task12;
ANALYZE pgsql;

EXPLAIN SELECT * FROM pgsql;

vim /opt/cost.txt
    cost=0.00..1834.00 rows=100000 width=37


EXPLAIN ANALYZE SELECT * FROM pgsql WHERE id >= 10 and id < 20;
vim /opt/explain_cost.txt
    cost=0.29..8.47 rows=9 width=37) (actual time=0.044..0.052 rows=10 loops=1


su postgres
psql
\c task12;
EXPLAIN SELECT * FROM pgsql WHERE upper(id::text)::int < 20;



vim /opt/expression.txt
    cost=0.00..3334.00 rows=33333 width=37



CREATE TABLE success_practice(
     id INT PRIMARY KEY, 
     description text ,
     pgsql_id int references pgsql(id)
);
INSERT INTO pgsql SELECT n , md5 (random()::text) FROM generate_series (1, 100000) AS foo(n);
     

INSERT INTO success_practice (id, description, pgsql_id) SELECT n, md5(n::text), random()*99999+1 FROM generate_series(1,200000) AS foo(n);

EXPLAIN ANALYZE SELECT * FROM pgsql inner JOIN success_practice on pgsql.id = success_practice.pgsql_id WHERE pgsql_id = 1000;

vim /opt/execution_without_index.txt
    cost=0.00..4370.00 rows=3 width=41) (actual time=37.496..45.944 rows=2 loops=1


CREATE index on success_practice (pgsql_id);

EXPLAIN ANALYZE SELECT * FROM pgsql inner JOIN success_practice on pgsql.id = success_practice.pgsql_id WHERE pgsql_id = 1000;

vim /opt/execution_with_index.txt
    cost=0.00..4.44 rows=3 width=0) (actual time=0.103..0.103 rows=2 loops=1