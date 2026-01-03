# Install and setup Node Exporter + Prometheus in Ubuntu

#### Prerequisite
- Install and setup grafana
- You can get all latest Prometheus related releases here : [Download Page](https://prometheus.io/download/#node_exporter)
- Copy the latest tar.gz file

#### Installation on TARGET Server
- Download Node Exporter in linux server that you want to monitor
```
cd /tmp

wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz


tar xvf node_exporter-1.10.2.linux-amd64.tar.gz
```
- Create a separate linux user for node exporter
```
sudo useradd --no-create-home --shell /bin/false node_exporter
```
- Setup systemd for node exporter 
```
sudo mv node_exporter-1.10.2.linux-amd64/node_exporter /usr/local/bin/

sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

sudo nano /etc/systemd/system/node_exporter.service
```
- Add the following configs in the systemd file 
```
[Unit]
Description=Prometheus Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
- Enable the service 
```
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```
- Verify the running metrics. It should run at port 9100 
```
curl http://localhost:9100/metrics
```
- Additionally, you can whitelist the grafana / promethus server IP to access port 9100 and restrict all other ips


#### Installation on MONITOR Server
- Create a monitor user to use for all monitoring tools 
```
sudo useradd --no-create-home --shell /bin/false monitor
```
- Create directories
```
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown monitor:monitor /var/lib/prometheus
```
- Download Prometheus LTS in linux server where Grafana is installed
```
cd /tmp

wget https://github.com/prometheus/prometheus/releases/download/v3.8.1/prometheus-3.8.1.linux-amd64.tar.gz


tar xvf prometheus-3.8.1.linux-amd64.tar.gz
```
- Move installation to proper directories 
```
sudo mv prometheus-3.8.1.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-3.8.1.linux-amd64/promtool /usr/local/bin/
sudo chown monitor:monitor /usr/local/bin/prometheus /usr/local/bin/promtool
```
- Add configs in yml `sudo nano /etc/prometheus/prometheus.yml`
```
global:
  scrape_interval: 30s
  evaluation_interval: 30s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets:
          - "localhost:9090"

  - job_name: "node_exporter"
    static_configs:
      - targets:
          - "localhost:9100"
        labels:
          role: "backend"
          env: "prod"

      - targets:
          - "localhost:9100"
        labels:
          role: "frontend"
          env: "prod"

```
- Create systemd file : `sudo nano /etc/systemd/system/prometheus.service`
```
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=monitor
Group=monitor
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
- Enable and verify installation 
```
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```


#### Grafana Configs
- Add prometheus data to grafana
  - Settings → Data Sources
  - Add → Prometheus
  - Add url : http://localhost:9090
  - Save & Test

- Add grafana dashboard
  - Grafana → Dashboards → Import
  - Enter 1860
  - Select Prometheus datasource
