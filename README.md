🖥️ Infrastructure Monitoring Setup: Prometheus + Grafana + Node Exporter
This guide walks you through setting up a basic monitoring stack using Prometheus, Grafana, and Node Exporter.

1. Install Node Exporter on Each Target Server
🔧 Create Node Exporter User & Download Binary
bash
Copy
Edit
sudo useradd -rs /bin/false node_exporter
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
tar xvf node_exporter-1.9.1.linux-amd64.tar.gz
sudo cp node_exporter-1.9.1.linux-amd64/node_exporter /usr/local/bin
⚙️ Create systemd Service
bash
Copy
Edit
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
▶️ Start and Enable the Service
bash
Copy
Edit
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
✅ Test Node Exporter
bash
Copy
Edit
curl http://localhost:9100/metrics
2. Install Prometheus on Monitoring Server
🔧 Create Prometheus User and Directories
bash
Copy
Edit
sudo useradd -rs /bin/false prometheus
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
📥 Download and Install Prometheus
bash
Copy
Edit
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.48.1/prometheus-2.48.1.linux-amd64.tar.gz
tar xvf prometheus-2.48.1.linux-amd64.tar.gz
cd prometheus-2.48.1.linux-amd64
sudo cp prometheus promtool /usr/local/bin/
sudo cp -r consoles console_libraries /etc/prometheus
⚙️ Create Prometheus Config
bash
Copy
Edit
sudo tee /etc/prometheus/prometheus.yml > /dev/null <<EOF
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporters'
    static_configs:
      - targets: ['192.168.1.10:9100', '192.168.1.11:9100']
EOF
🔧 Create systemd Service for Prometheus
bash
Copy
Edit
sudo tee /etc/systemd/system/prometheus.service > /dev/null <<EOF
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \\
  --config.file /etc/prometheus/prometheus.yml \\
  --storage.tsdb.path /var/lib/prometheus \\
  --web.console.templates=/etc/prometheus/consoles \\
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
EOF
▶️ Start and Enable the Service
bash
Copy
Edit
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
3. Install Grafana on Monitoring Server
🐧 For Ubuntu/Debian Systems
bash
Copy
Edit
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install grafana
▶️ Start and Enable the Service
bash
Copy
Edit
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
4. Configure Grafana
➕ Add Data Source
URL: http://localhost:9090

Type: Prometheus

📊 Import Dashboards (Go to Dashboards → + Import)
ID 1860 – Node Exporter Full

ID 14513 – Linux/Windows

ID 7587 – Disk I/O & Network

ID 3662 – Prometheus Internal Stats

5. Verify Services Status
bash
Copy
Edit
sudo systemctl status node_exporter
sudo systemctl status prometheus
sudo systemctl status grafana-server
