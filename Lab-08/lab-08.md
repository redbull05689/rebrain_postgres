yum -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm 
yum -y update && yum -y install vim bash-completion wget postgresql13 postgresql13-server python3 lzop pv gcc python36-setuptools python36 python36-devel
alias pip=pip3 

/usr/pgsql-13/bin/postgresql-13-setup initdb

systemctl enable postgresql-13 && systemctl start postgresql-13
systemctl status postgresql-13

pip3 install wal-e envdir boto

python3 -c 'import boto; print(boto.__path__[0])' | xargs -I{} sudo chmod -R a+rx {}

# For postgres user
sudo mkdir -m 0750 -p /etc/wal-e/env
sudo chgrp -R postgres /etc/wal-e

sudo -i
umask u=rwx,g=r,o=
echo 'AKIAZMJMH6DA7MLVRDPA' > /etc/wal-e/env/AWS_ACCESS_KEY_ID 
echo 'LzgBItujVQBI300aoJ/fxF+CMT+ZtTFNYb3VGDn3' > /etc/wal-e/env/AWS_SECRET_ACCESS_KEY
echo 's3://5832f60adfdcffcb0d5ddfc396f2b0f4' > /etc/wal-e/env/WALE_S3_PREFIX
echo 'eu-central-1' > /etc/wal-e/env/AWS_REGION
chgrp postgres /etc/wal-e/env/*

su postgres

vi ~/13/data/postgresql.conf
    wal_level = 'replica' 
    archive_mode = 'on' 
    archive_command = 'envdir /etc/wal-e/env wal-e wal-push %p' 
    archive_timeout = '60'

psql -f /opt/schema.sql
psql -f /opt/data.sql

systemctl restart postgresql-13

sudo -u postgres bash -c '/usr/local/bin/envdir /etc/wal-e/env /usr/local/bin/wal-e backup-list' > /tmp/file

# Check logs
tail -f ~/13/data/log/postgresql-Mon.log 


# Check aws S3
aws configure
aws s3api list-buckets --query "Buckets[].Name"
aws s3 ls s3://5832f60adfdcffcb0d5ddfc396f2b0f4