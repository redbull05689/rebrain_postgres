
На сервера etcd-1, etcd-2 и etcd-3:

    vim /root/etcd_installer.sh


    bash /root/etcd_installer.sh
    cp /task13/etcd/etcd* /usr/local/bin



mkdir -p /etc/etcd/ && \
mkdir -p /task13/etcd/data-dir && \
mkdir -p /task13/etcd/wal-dir 


chmod -R 777 /task13/

    
    # Смотри etcd.yml 
    vim /etc/etcd/etcd.yml

    useradd etcd

chown -R etcd:etcd /task13/etcd/ && \
chown -R etcd:etcd /etc/etcd/

hostnamectl set-hostname rebrain-etcd-node-2 --static
hostname rebrain-etcd-node-2
tee -a /etc/hosts<<EOF
157.230.103.201 rebrain-etcd-node-1
157.230.112.92 rebrain-etcd-node-2
134.122.72.218 rebrain-etcd-node-3
EOF

vim /etc/systemd/system/etcd.service

systemctl daemon-reload && systemctl enable etcd && reboot
    
etcdctl --endpoints="157.230.103.201:2379,157.230.112.92:2379,134.122.72.218:2379" endpoint status


### На сервера patroni-1, patroni-2 и patroni-3 :

apt update && \
apt -y install python3-pip python3-dev libpq-dev && \
pip3 install --upgrade setuptools pip && \
pip install psycopg2-binary && \
pip install patroni[etcd]


wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - && \
echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" | tee  /etc/apt/sources.list.d/pgdg.list && \
apt update && apt install postgresql-13 -y

systemctl stop postgresql && systemctl disable postgresql

mkdir /etc/patroni && \
mkdir -p /task13/patroni && \
chown -R postgres:postgres /etc/patroni && \
chown -R postgres:postgres /task13/patroni

vim /etc/patroni/patroni.yml

vim /etc/systemd/system/patroni.service

systemctl daemon-reload && systemctl enable patroni

### Для проверки
patronictl -c /etc/patroni/patroni.yml list

### Haproxy установка

apt update && \
apt install haproxy -y

vim /etc/haproxy/haproxy.cfg