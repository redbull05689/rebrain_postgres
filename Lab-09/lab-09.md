apt update && apt -y install vim bash-completion wget make
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -
echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" | tee  /etc/apt/sources.list.d/pgdg.list
apt update && apt install postgresql-13 postgresql-client-13 -y

systemctl enable postgresql && systemctl status postgresql

su postgres
psql
CREATE USER root WITH SUPERUSER PASSWORD 'postgres';
CREATE USER rebrain_monitoring WITH SUPERUSER PASSWORD 'postgres';
CREATE USER rebrain_admin WITH SUPERUSER PASSWORD 'postgres';
CREATE DATABASE rebrain_courses_db WITH OWNER rebrain_admin;
GRANT ALL privileges on database rebrain_courses_db to rebrain_admin;



mkdir prometheus && cd $_
wget https://github.com/prometheus/prometheus/releases/download/v2.24.1/prometheus-2.24.1.linux-amd64.tar.gz
tar -xvf prometheus-2.24.1.linux-amd64.tar.gz
mv prometheus-2.24.1.linux-amd64 prometheus-files

useradd --no-create-home --shell /bin/false prometheus
mkdir /etc/prometheus
mkdir /var/lib/prometheus
chown prometheus:prometheus /etc/prometheus
chown prometheus:prometheus /var/lib/prometheus

cp prometheus-files/prometheus /usr/local/bin/
cp prometheus-files/promtool /usr/local/bin/
chown prometheus:prometheus /usr/local/bin/prometheus
chown prometheus:prometheus /usr/local/bin/promtool

cp -r prometheus-files/consoles /etc/prometheus
cp -r prometheus-files/console_libraries /etc/prometheus
chown -R prometheus:prometheus /etc/prometheus/consoles
chown -R prometheus:prometheus /etc/prometheus/console_libraries

    vi /etc/prometheus/prometheus.yml
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']
  - job_name: postgresql
    static_configs:
      - targets: ['localhost:9187']
        labels:
          alias: postgres


chown prometheus:prometheus /etc/prometheus/prometheus.yml

    vi /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \ 
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target

systemctl daemon-reload
systemctl start prometheus
systemctl enable prometheus

apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_7.3.7_amd64.deb
dpkg -i grafana_7.3.7_amd64.deb

systemctl start grafana-server
systemctl enable grafana-server

cd /tmp
curl -LO https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz

tar -xvf node_exporter-0.18.1.linux-amd64.tar.gz

mv node_exporter-0.18.1.linux-amd64/node_exporter /usr/local/bin/
useradd --no-create-home --shell /bin/false node_exporter
chown node_exporter:node_exporter /usr/local/bin/node_exporter


    vi /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
After=network.target
 
[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
 
[Install]
WantedBy=multi-user.target


systemctl daemon-reload
systemctl start node_exporter
systemctl enable node_exporter

wget https://github.com/prometheus-community/postgres_exporter/releases/download/v0.9.0/postgres_exporter-0.9.0.linux-amd64.tar.gz -O - | tar -xzv -C /tmp
cp /tmp/postgres_exporter-0.9.0.linux-amd64/postgres_exporter /usr/local/bin/prometheus-postgres-exporter


chown -R postgres:postgres /usr/local/bin/prometheus-postgres-exporter

    vi /etc/systemd/system/prometheus-postgres-exporter.service
[Unit]
Description=Prometheus PostgreSQL Exporter
After=network.target

[Service]
Type=simple
Restart=always
User=postgres
Group=postgres
Environment=DATA_SOURCE_NAME="user=postgres host=/var/run/postgresql/ sslmode=disable"
ExecStart=/usr/local/bin/prometheus-postgres-exporter
[Install]
WantedBy=multi-user.target


systemctl daemon-reload && systemctl start prometheus-postgres-exporter.service && systemctl enable prometheus-postgres-exporter.service







https://mcs.mail.ru/help/ru_RU/cases-monitoring/case-psql-exporter