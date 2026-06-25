# 📊 Grafana + Prometheus + Node Exporter Installation on AWS EC2

A complete step-by-step guide to installing **Grafana**, **Prometheus**, and **Node Exporter** on an **AWS EC2 (Amazon Linux 2023)** instance for infrastructure monitoring.

---

# 📑 Table of Contents

- Overview
- Architecture
- Environment Details
- Prerequisites
- AWS Security Group Configuration
- Install Node Exporter
- Install Grafana
- Install Prometheus
- Configure Prometheus
- Verify Installation
- Grafana Integration
- Common Metrics
- Troubleshooting
- Interview Notes
- Future Improvements

---

# 📌 Overview

This project demonstrates how to build a complete monitoring stack using:

- **Grafana** – Visualization Dashboard
- **Prometheus** – Metrics Collection & Storage
- **Node Exporter** – Linux System Metrics Exporter

The setup runs entirely on a single AWS EC2 instance.

---

# 🏗 Architecture

```
                    +----------------------+
                    |      AWS EC2         |
                    | Amazon Linux 2023    |
                    +----------+-----------+
                               |
          -----------------------------------------
          |                  |                    |
          |                  |                    |
+---------v------+   +-------v-------+   +--------v--------+
| Node Exporter  |   | Prometheus    |   | Grafana         |
| Port : 9100    |-->| Port : 9090   |-->| Port : 3000     |
| System Metrics |   | Scrapes Data  |   | Dashboards      |
+----------------+   +---------------+   +-----------------+
```

---

# 🖥 Environment Details

| Component | Value |
|------------|-------|
| Cloud | AWS |
| Service | EC2 |
| Instance Type | t3.medium |
| Operating System | Amazon Linux 2023 |
| Kernel | 6.1 |
| User | ec2-user |
| Storage | 60 GB SSD |
| Key Pair | grafana.pem |

---

# 🔐 AWS Security Group Configuration

| Service | Port | Protocol |
|----------|------|----------|
| SSH | 22 | TCP |
| Grafana | 3000 | TCP |
| Prometheus | 9090 | TCP |
| Node Exporter | 9100 | TCP |

---

# 📦 Step 1 – Update the Server

```bash
sudo yum update -y
```

---

# 📊 Install Node Exporter

## Download

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz

tar xvfz node_exporter-1.10.2.linux-amd64.tar.gz

cd node_exporter-1.10.2.linux-amd64
```

---

## Test Manually

```bash
./node_exporter
```

Open:

```
http://EC2-PUBLIC-IP:9100/metrics
```

You should see Prometheus metrics.

---

## Install Binary

```bash
sudo cp node_exporter /usr/local/bin

sudo useradd node_exporter \
--no-create-home \
--shell /bin/false

sudo chown node_exporter:node_exporter \
/usr/local/bin/node_exporter
```

---

## Create Systemd Service

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Paste:

```ini
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
```

---

## Start Service

```bash
sudo systemctl daemon-reload

sudo systemctl start node_exporter

sudo systemctl enable node_exporter

sudo systemctl status node_exporter
```

---

## Verify

```
http://EC2-PUBLIC-IP:9100/metrics
```

---

# 📈 Install Grafana

## Install Dependencies

```bash
sudo yum update -y

sudo yum install wget tar make -y
```

---

## Install Grafana Enterprise

```bash
sudo yum install -y \
https://dl.grafana.com/grafana-enterprise/release/12.2.1/grafana-enterprise_12.2.1_18655849634_linux_amd64.rpm
```

---

## Verify Version

```bash
grafana-server --version
```

---

## Start Grafana

```bash
sudo systemctl start grafana-server

sudo systemctl enable grafana-server

sudo systemctl status grafana-server
```

---

## Access Grafana

```
http://EC2-PUBLIC-IP:3000
```

Default Login

```
Username : admin
Password : admin
```

---

# 📥 Download Prometheus

Download the latest Linux release:

https://prometheus.io/download/

---

# 📤 Copy Prometheus to EC2

Linux/macOS example:

```bash
scp -i grafana.pem \
prometheus-3.5.3.linux-amd64.tar.gz \
ec2-user@EC2-PUBLIC-IP:/home/ec2-user/
```

Windows users can use **WinSCP**.

---

# 📦 Install Prometheus

Move and extract:

```bash
sudo mv prometheus-3.5.3.linux-amd64.tar.gz /opt

cd /opt

sudo tar -xvf prometheus-3.5.3.linux-amd64.tar.gz

sudo mv prometheus-3.5.3.linux-amd64 prometheus

sudo rm prometheus-3.5.3.linux-amd64.tar.gz
```

---

# 👤 Create Prometheus User

```bash
sudo useradd \
--no-create-home \
--shell /bin/false \
prometheus
```

---

# 📁 Configure Directories

```bash
cd /opt/prometheus

sudo cp prometheus /usr/local/bin/

sudo cp promtool /usr/local/bin/

sudo mkdir /etc/prometheus

sudo mkdir /var/lib/prometheus

sudo cp prometheus.yml /etc/prometheus/
```

---

## Set Permissions

```bash
sudo chown -R prometheus:prometheus /etc/prometheus

sudo chown -R prometheus:prometheus /var/lib/prometheus

sudo chown prometheus:prometheus /usr/local/bin/prometheus

sudo chown prometheus:prometheus /usr/local/bin/promtool

sudo chmod -R 755 /etc/prometheus

sudo chmod -R 755 /var/lib/prometheus
```

---

# ⚙ Create Prometheus Systemd Service

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Paste:

```ini
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
```

---

# 📝 Configure Prometheus

Replace the default configuration:

```bash
sudo rm /etc/prometheus/prometheus.yml

sudo nano /etc/prometheus/prometheus.yml
```

Paste:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets:

rule_files:

scrape_configs:

  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
        labels:
          app: "prometheus"

  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

---

# ▶ Start Prometheus

```bash
sudo systemctl daemon-reload

sudo systemctl enable prometheus

sudo systemctl start prometheus

sudo systemctl status prometheus
```

---

## Access Prometheus

```
http://EC2-PUBLIC-IP:9090
```

---

# 🔗 Connect Grafana with Prometheus

1. Login to Grafana
2. Go to **Connections → Data Sources**
3. Click **Add Data Source**
4. Select **Prometheus**
5. URL:

```
http://localhost:9090
```

6. Click **Save & Test**

---

# 📊 Common Metrics Available

- CPU Usage
- Memory Usage
- Disk Utilization
- Disk I/O
- Network Traffic
- Filesystem Usage
- Load Average
- System Uptime

---

# 📈 Monitor Multiple Servers

To monitor another Linux server running Node Exporter:

```yaml
scrape_configs:
  - job_name: node_exporter

    static_configs:
      - targets:
          - SERVER-IP:9100
```

Reload Prometheus after updating the configuration.

---

# 🔍 Troubleshooting

### Port not accessible

- Check AWS Security Group
- Verify firewall rules

---

### Service failed to start

```bash
sudo systemctl status prometheus

sudo journalctl -u prometheus
```

or

```bash
sudo systemctl status node_exporter

sudo journalctl -u node_exporter
```

---

### Binary not found

Verify installation:

```bash
which prometheus

which node_exporter
```

---

### Check Listening Ports

```bash
sudo ss -tulpn
```

---

# ✅ Validation Checklist

- [x] EC2 instance running
- [x] Node Exporter installed
- [x] Prometheus installed
- [x] Grafana installed
- [x] Systemd services enabled
- [x] Security Group ports opened
- [x] Prometheus collecting metrics
- [x] Grafana connected to Prometheus

---

# 💡 Interview Notes

### Node Exporter

- Default Port: **9100**
- Stateless exporter
- Exposes Linux system metrics
- Prometheus pulls metrics using HTTP

### Prometheus

- Default Port: **9090**
- Pull-based monitoring
- Stores time-series data
- Uses YAML configuration

### Grafana

- Default Port: **3000**
- Visualization platform
- Supports multiple data sources
- Alerting and dashboards

---

# 🚀 Future Enhancements

- Add Alertmanager
- Configure Email Alerts
- Enable HTTPS/TLS
- Configure Authentication
- Monitor Multiple EC2 Instances
- Deploy using Docker
- Automate using Terraform
- Automate using Ansible
- Install Nginx Reverse Proxy
- Enable Persistent Storage

---

# 📚 Tech Stack

- AWS EC2
- Amazon Linux 2023
- Grafana Enterprise
- Prometheus
- Node Exporter
- Systemd

---

## ⭐ If you found this project useful, consider giving it a Star!
