# AI-Based-Predictive-Maintenance-System-for-Server-Infrastructure

nfrastructure Monitoring Setup: Prometheus + Grafana + Node Exporter
1. Install Node Exporter on each target server
------------------------------------------------------
Create user & download binary:
sudo useradd -rs /bin/false node_exporter
cd /tmp
wget
https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar
.gz
tar xvf node_exporter-1.9.1.linux-amd64.tar.gz
sudo cp node_exporter-1.9.1.linux-amd64/node_exporter /usr/local/bin
Create systemd service:
sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<EOF
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
[Install]
WantedBy=multi-user.target
EOF
Start & enable:
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
Test:
curl http://localhost:9100/metrics
2. Install Prometheus on monitoring server
------------------------------------------------------
Create user & directories:
sudo useradd -rs /bin/false prometheus
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
Download & install:
cd /tmp
wget
https://github.com/prometheus/prometheus/releases/download/v2.48.1/prometheus-2.48.1.linux-amd64.tar.gz
tar xvf prometheus-2.48.1.linux-amd64.tar.gz
cd prometheus-2.48.1.linux-amd64
sudo cp prometheus promtool /usr/local/bin/
sudo cp -r consoles console_libraries /etc/prometheus
Create config:
sudo tee /etc/prometheus/prometheus.yml > /dev/null <<EOF
global:
scrape_interval: 15s
scrape_configs:
- job_name: 'node_exporters'
static_configs:
- targets: ['192.168.1.10:9100', '192.168.1.11:9100']
EOF
Create systemd service:
sudo tee /etc/systemd/system/prometheus.service > /dev/null <<EOF
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target
[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus --config.file /etc/prometheus/prometheus.yml --storage.tsdb.path
/var/lib/prometheus --web.console.templates=/etc/prometheus/consoles
--web.console.libraries=/etc/prometheus/console_libraries
[Install]
WantedBy=multi-user.target
EOF
Start & enable:
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
3. Install Grafana on monitoring server
------------------------------------------------------
(Ubuntu/Debian)
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install grafana
Start & enable:
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
4. In Grafana
------------------------------------------------------
Add data source: Prometheus at http://localhost:9090
Import dashboards (go to Dashboards > + Import):
- ID 1860 (Node Exporter Full)
- ID 14513 (Linux/Windows)
- ID 7587 (Disk IO/Network)
- ID 3662 (Prometheus internal)
5. Check services
------------------------------------------------------
sudo systemctl status node_exporter
sudo systemctl status prometheus
sudo systemctl status grafana-server
