# AWS-Practice
Practicing AWS features
📊 Grafana & Prometheus, Node Exporter Installation on AWS EC2
🔹 Environment Details
Component	Value
Cloud	AWS
Service	EC2
Instance Type	t3.medium
OS	Amazon Linux 2023 (kernel-6.1)
Key Pair	grafana.pem
User	ec2-user
SSD	60GB
🔹 AWS Security Group Configuration
Service	Port	Protocol
Grafana	3000	TCP
Prometheus	9090	TCP
Node Exporter	9100	TCP
SSH	22	TCP
🧩 Architecture Overview
EC2 → Hosts both services
Grafana → Visualization layer (Port 3000)
Prometheus → Metrics collection & storage (Port 9090)
Systemd → Manages Prometheus & Grafana as services
🔹 Step 1: Update EC2 System
sudo yum update -y
📊 Node Exporter Setup (Linux)
🔹 Overview
Node Exporter is a component of Prometheus used to collect system-level metrics like CPU, memory, disk, and network from Linux servers.

🔹 Step 2: Download & Extract
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz
tar xvfz node_exporter-1.10.2.linux-amd64.tar.gz
cd node_exporter-1.10.2.linux-amd64
🔹 Step 3: Run Manually (Test)
./node_exporter
👉 Access in browser:

http://EC2-PUBLIC-IP:9100/metrics
🔹 Step 4: Install Binary & Create User
sudo cp node_exporter /usr/local/bin
sudo useradd node_exporter --no-create-home --shell /bin/false
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
🔹 Step 5: Create Systemd Service
sudo nano /etc/systemd/system/node_exporter.service
Add below content:
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
🔹 Step 6: Start & Enable Service
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
sudo systemctl status node_exporter
🔹 Step 7: Verify
Open browser:

http://EC2-PUBLIC-IP:9100/metrics
✔ You should see system metrics output

🔹 Step 8: Install Grafana (Enterprise Edition)
Install Grafana RPM
sudo yum update -y
sudo yum install wget tar -y
sudo yum install make -y
sudo yum install -y https://dl.grafana.com/grafana-enterprise/release/12.2.1/grafana-enterprise_12.2.1_18655849634_linux_amd64.rpm
Verify Grafana Version
grafana-server --version
🔹 Step 9: Start & Enable Grafana Service
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
✅ Grafana Web UI

http://EC2-PUBLIC-IP:3000
Default Credentials

Username: admin
Password: admin
WinSCP >> Install
https://winscp.net/eng/download.php

Download Prometheus Setup | LTS/Linux
https://prometheus.io/download/

🔹 Step 10: Copy Prometheus Binary to EC2
SCP from Local Machine (macOS/Linux)
scp -i /Users/atul/Downloads/grafana.pem \
/Users/atul/Downloads/prometheus-3.5.3.linux-amd64.tar.gz \
ec2-user@ec2-98-82-185-137.compute-1.amazonaws.com:/home/ec2-user/
🔹 Step 11: Move & Extract Prometheus
sudo mv prometheus-3.5.3.linux-amd64.tar.gz /opt
cd /opt
sudo tar -xvf prometheus-3.5.3.linux-amd64.tar.gz
sudo mv prometheus-3.5.3.linux-amd64 prometheus
sudo rm prometheus-3.5.3.linux-amd64.tar.gz
🔹 Step 12: Create Prometheus System User
sudo useradd --no-create-home --shell /bin/false prometheus
🔹 Step 13: Configure Prometheus Directories
cd /opt/prometheus

sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/

sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

sudo cp prometheus.yml /etc/prometheus/
Set Permissions
sudo chown -R prometheus:prometheus /etc/prometheus 
sudo chown -R prometheus:prometheus /var/lib/prometheus 
sudo chown prometheus:prometheus /usr/local/bin/prometheus 
sudo chown prometheus:prometheus /usr/local/bin/promtool 
sudo chmod -R 755 /etc/prometheus 
sudo chmod -R 755 /var/lib/prometheus
🔹 Step 14: Create Prometheus systemd Service
sudo nano /etc/systemd/system/prometheus.service
📄 Paste the Following Configuration
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
🔹 Step 15: Update Prometheus Config
remove config file

sudo rm /etc/prometheus/prometheus.yml
edit config file

sudo nano /etc/prometheus/prometheus.yml
# my global config
global:
  scrape_interval: 15s
  evaluation_interval: 15s

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# Scrape configurations
scrape_configs:

  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
        labels:
          app: "prometheus"

  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
Start Prometheus Service

sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
✅ Prometheus Web UI

http://<EC2-PUBLIC-IP>:9090
🔹 Common Metrics Collected
CPU usage
Memory usage
Disk I/O
Network traffic
File system stats
🔹 Integration with Prometheus
Add this in Prometheus config:

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['<SERVER-IP>:9100']
🔹 Quick Notes (Exam / Interview)
Default port → 9100
Stateless exporter
Pull-based monitoring (Prometheus scrapes metrics)
Works with Grafana dashboards
🔹 Troubleshooting
Port not accessible → Check firewall (port 9100)
Service not starting → journalctl -u node_exporter
Binary issue → Ensure correct architecture (amd64)
🎯 Final Validation Checklist
✔ Grafana running ✔ Prometheus running ✔ systemd services enabled ✔ Ports opened in Security Group ✔ Accessible via browser

🚀 Recommended flow
Add Prometheus as Grafana Data Source
Install Node Exporter
Import Grafana Dashboards
Enable Alertmanager
Add TLS + Authentication
Convert setup into Terraform / Ansible
