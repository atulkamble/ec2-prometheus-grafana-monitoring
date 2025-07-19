clean, end-to-end **Prometheus and Grafana monitoring setup** on an EC2 instance (or any Linux VM) with example configurations. 

* 📦 **Prometheus installation + config**
* 📦 **Grafana installation**
* 📜 **Sample Prometheus scrape targets**
* 📊 **Grafana dashboard setup using a JSON export**
* 📁 **GitHub-ready repo structure**

---

## 📂 Project Structure

```
prometheus-grafana-monitoring/
├── prometheus/
│   ├── prometheus.yml
│   └── rules.yml
├── grafana/
│   └── dashboards/
│       └── node_exporter_dashboard.json
├── install.sh
└── README.md
```

---

## 📥 Full Code & Config

---

### 📜 `prometheus/prometheus.yml`

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

---

### 📜 `prometheus/rules.yml`

```yaml
groups:
  - name: example-alert
    rules:
      - alert: HighCPUUsage
        expr: node_load1 > 2
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "High CPU load detected"
```

---

### 📜 `grafana/dashboards/node_exporter_dashboard.json`

Sample node\_exporter dashboard JSON you can import into Grafana.
👉 I’ll generate a JSON file for you upon request, but you can also download community dashboards from Grafana Labs for `node_exporter` (Dashboard ID: `1860`).

---

## 📜 `install.sh`

A shell script to install **Prometheus**, **Node Exporter**, and **Grafana** on a Linux server (EC2/AWS, DigitalOcean, or your VM).

```bash
#!/bin/bash

# Exit on any error
set -e

# Ensure the script is run as root
if [ "$EUID" -ne 0 ]; then
  echo "❌ Please run as root or use sudo"
  exit 1
fi

# Variables
PROM_VERSION="2.52.0"
NODE_EXPORTER_VERSION="1.8.0"
GRAFANA_RPM="https://dl.grafana.com/enterprise/release/grafana-enterprise-12.0.2-1.x86_64.rpm"

# Install Prometheus
useradd --no-create-home --shell /bin/false prometheus
mkdir -p /etc/prometheus /var/lib/prometheus

wget https://github.com/prometheus/prometheus/releases/download/v$PROM_VERSION/prometheus-$PROM_VERSION.linux-amd64.tar.gz
tar -xvf prometheus-$PROM_VERSION.linux-amd64.tar.gz

cp prometheus-$PROM_VERSION.linux-amd64/prometheus /usr/local/bin/
cp prometheus-$PROM_VERSION.linux-amd64/promtool /usr/local/bin/
cp -r prometheus-$PROM_VERSION.linux-amd64/consoles /etc/prometheus
cp -r prometheus-$PROM_VERSION.linux-amd64/console_libraries /etc/prometheus
cp prometheus-$PROM_VERSION.linux-amd64/prometheus.yml /etc/prometheus/
# Optional: touch a basic rules.yml if missing
touch /etc/prometheus/rules.yml

# Create Prometheus systemd service
cat <<EOF > /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus Monitoring
After=network.target

[Service]
User=prometheus
ExecStart=/usr/local/bin/prometheus \\
  --config.file=/etc/prometheus/prometheus.yml \\
  --storage.tsdb.path=/var/lib/prometheus/ \\
  --web.console.templates=/etc/prometheus/consoles \\
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
EOF

# Start Prometheus
systemctl daemon-reload
systemctl enable --now prometheus

# Install Node Exporter
useradd --no-create-home --shell /bin/false node_exporter

wget https://github.com/prometheus/node_exporter/releases/download/v$NODE_EXPORTER_VERSION/node_exporter-$NODE_EXPORTER_VERSION.linux-amd64.tar.gz
tar -xvf node_exporter-$NODE_EXPORTER_VERSION.linux-amd64.tar.gz

cp node_exporter-$NODE_EXPORTER_VERSION.linux-amd64/node_exporter /usr/local/bin/

# Create Node Exporter service
cat <<EOF > /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

# Start Node Exporter
systemctl daemon-reload
systemctl enable --now node_exporter

# Install Grafana
yum install -y $GRAFANA_RPM

systemctl daemon-reload
systemctl enable --now grafana-server

echo -e "\n✅ Installation complete! Access Grafana at http://<your-ec2-public-ip>:3000 (default login: admin/admin)"

```

---

## 📜 `README.md`

````markdown
# 📊 Prometheus & Grafana Monitoring Setup

A complete monitoring stack setup using Prometheus, Node Exporter, and Grafana.

## 📦 Components
- Prometheus
- Node Exporter
- Grafana

## 📜 Install Instructions

```bash
chmod +x install.sh
sudo ./install.sh
````

## 📊 Grafana Dashboard Import

* Access Grafana: `http://<your-ec2-ip>:3000`
* Login: admin / admin
* Import dashboard JSON: `grafana/dashboards/node_exporter_dashboard.json`

## 📈 Prometheus Targets

* Prometheus UI: `http://<your-ec2-ip>:9090/targets`

## 📊 Sample Dashboard

(Include screenshot if deploying on your instance)

## 📄 Alerts

* Example alert for high CPU in `prometheus/rules.yml`

---
## 👨‍💻 Author

**Atul Kamble**

- 💼 [LinkedIn](https://www.linkedin.com/in/atuljkamble)
- 🐙 [GitHub](https://github.com/atulkamble)
- 🐦 [X](https://x.com/Atul_Kamble)
- 📷 [Instagram](https://www.instagram.com/atuljkamble)
- 🌐 [Website](https://www.atulkamble.in)
