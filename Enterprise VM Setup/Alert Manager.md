# Install and setup Prometheus Alert Manager in Ubuntu

#### Prerequisite
- Install and setup grafana and prometheus, refer other readme.

#### Installation on MONITOR Server
- Download Alert Manager in monitor linux server.
```
cd /tmp

wget https://github.com/prometheus/alertmanager/releases/download/v0.30.0/alertmanager-0.30.0.linux-amd64.tar.gz

tar xvf alertmanager-0.30.0.linux-amd64.tar.gz
```
- Move to local/bin
```
sudo mv alertmanager-0.30.0.linux-amd64/alertmanager /usr/local/bin/
sudo mv alertmanager-0.30.0.linux-amd64/amtool /usr/local/bin/

sudo mkdir -p /etc/alertmanager
sudo mkdir -p /var/lib/alertmanager

sudo mv alertmanager-0.30.0.linux-amd64/alertmanager.yml /etc/alertmanager/alertmanager.yml
```
- Create systemd service `sudo nano /etc/systemd/system/alertmanager.service` with content
```
[Unit]
Description=Prometheus Alertmanager
After=network.target

[Service]
User=monitor
Group=monitor
Type=simple
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager

Restart=always

[Install]
WantedBy=multi-user.target

```
- Reload and enable
```
sudo systemctl daemon-reload
sudo systemctl enable alertmanager
sudo systemctl start alertmanager
sudo systemctl status alertmanager
```
- Configure Prometheus `sudo nano /etc/prometheus/prometheus.yml`
```
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - "localhost:9093"

```
- Restart and create rules 
```
sudo systemctl restart prometheus
sudo mkdir -p /etc/prometheus/rules
sudo nano /etc/prometheus/rules/node_alerts.yml
```
- Grafana compatible rules 
```
groups:
  - name: node-alerts
    rules:
      - alert: NodeDown
        expr: up{job="server_metrics"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Node down ({{ $labels.instance }})"
          description: "Node Exporter is down on {{ $labels.instance }}"

      - alert: HighCPUUsage
        expr: avg by (instance) (rate(node_cpu_seconds_total{mode!="idle"}[5m])) > 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage > 10% for 5 minutes"

      - alert: HighMemoryUsage
        expr: |
          (
            (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes)
            / node_memory_MemTotal_bytes
          ) > 0.85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High RAM usage on {{ $labels.instance }}"
          description: "Memory usage > 85% for 5 minutes"
 
      - alert: LowDiskSpace
        expr: node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"} < 0.15
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Disk space < 15%"


```
- Add slack alerts in alert config `sudo nano /etc/alertmanager/alertmanager.yml`. Replace your slack webhook url.
```
global:
  resolve_timeout: 5m

route:
  receiver: "slack-notifications"
  group_by: ["alertname", "instance"]
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h

receivers:
  - name: "slack-notifications"
    slack_configs:
      - api_url: "https://hooks.slack.com/services/XXXXXXXX/XXXXXXX/XXXXXXX"
        channel: "aiqwip"
        title: "{{ .CommonAnnotations.summary }}"
        text: >-
          {{ range .Alerts }}
          *Alert:* {{ .Annotations.summary }}
          *Description:* {{ .Annotations.description }}
          *Instance:* {{ .Labels.instance }}
          *Severity:* {{ .Labels.severity }}
          {{ end }}
        send_resolved: true
```

