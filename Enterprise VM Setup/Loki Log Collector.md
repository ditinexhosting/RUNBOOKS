# Install and setup Grafana Loki in Ubuntu

#### Prerequisite
- Install and setup grafana and prometheus, refer other readme.

#### Installation on MONITOR Server
- Download Loki in monitor linux server. Refer [Loki releases](https://github.com/grafana/loki/releases/)
- Download amd64 for ubuntu
```
cd /tmp

wget https://github.com/grafana/loki/releases/download/v3.6.3/loki-linux-amd64.zip

unzip loki-linux-amd64.zip
sudo chmod +x loki-linux-amd64

sudo mv loki-linux-amd64 /usr/local/bin/loki
sudo mkdir -p /etc/loki
sudo mkdir -p /var/lib/loki/{chunks,rules}
```
- We already have monitor user, set the ownership to monitor.
```
sudo chown -R monitor:monitor /var/lib/loki /etc/loki
```
- Setup the loki config `sudo nano /etc/loki/loki-config.yaml` and paste the config. You can use the base config from (here)[https://github.com/grafana/loki/blob/main/cmd/loki/loki-local-config.yaml]
```
auth_enabled: false

server:
  http_listen_port: 9200
  grpc_listen_port: 9096
  log_level: debug
  grpc_server_max_concurrent_streams: 1000

common:
  instance_addr: 127.0.0.1
  path_prefix: /var/lib/loki
  storage:
    filesystem:
      chunks_directory: /var/lib/loki/chunks
      rules_directory: /var/lib/loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

query_range:
  results_cache:
    cache:
      embedded_cache:
        enabled: true
        max_size_mb: 100

limits_config:
  metric_aggregation_enabled: true
  enable_multi_variant_queries: true

schema_config:
  configs:
    - from: 2020-10-24
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

pattern_ingester:
  enabled: true
  metric_aggregation:
    loki_address: localhost:9200

ruler:
  alertmanager_url: http://localhost:9093

frontend:
  encoding: protobuf

analytics:
  reporting_enabled: false
```
- Create systemd service `sudo nano /etc/systemd/system/loki.service` with content
```
[Unit]
Description=Loki Log Aggregation System
Documentation=https://grafana.com/docs/loki/latest/
After=network-online.target

[Service]
Type=simple
User=monitor
Group=monitor
ExecStart=/usr/local/bin/loki -config.file=/etc/loki/loki-config.yaml
Restart=on-failure
RestartSec=5s

# Security settings
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/loki
PrivateTmp=true

[Install]
WantedBy=multi-user.target

```
- Reload and enable
```
sudo systemctl daemon-reload
sudo systemctl enable loki
sudo systemctl start loki
sudo systemctl status loki
```
- Verify whether it is ready
```
# Check if Loki is running
curl http://localhost:9200/ready

# Check metrics endpoint
curl http://localhost:9200/metrics

# View logs
sudo journalctl -u loki -f
```
- Send a test log entry
```
curl -X POST http://localhost:9200/loki/api/v1/push \
  -H "Content-Type: application/json" \
  -d '{
    "streams": [
      {
        "stream": {
          "job": "test"
        },
        "values": [
          ["'"$(date +%s)000000000"'", "test log message"]
        ]
      }
    ]
  }'
```
- Query log entry
```
curl -G -s "http://localhost:9200/loki/api/v1/query_range" \
  --data-urlencode 'query={job="test"}' \
  --data-urlencode 'limit=10' | jq
```
- Add loki Data source in grafana : `http://localhost:9200`
- Import Log viewer dashboard : `https://grafana.com/grafana/dashboards/13639-logs-app/`


##### Install Alloy to collect logs
- Install Grafana Alloy
```
sudo mkdir -p /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/grafana.asc https://apt.grafana.com/gpg-full.key
sudo chmod 644 /etc/apt/keyrings/grafana.asc
echo "deb [signed-by=/etc/apt/keyrings/grafana.asc] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

sudo apt-get update

sudo apt-get install alloy

sudo systemctl enable alloy
sudo systemctl start alloy
sudo systemctl status alloy
sudo systemctl restart alloy
```
- Check the alloy logs 
```
sudo journalctl -u alloy
```
- Refer to the (configuration docs)[https://grafana.com/docs/alloy/latest/configure/linux/]
- Example Config to tail pm2 error and out log and handle json as well as non json error. Edit the config file : `sudo nano /etc/alloy/config.alloy`
```
logging {
  level = "info"
}

loki.write "default" {
  endpoint {
    url = "https://stats.imeld.ai/loki/api/v1/push" 
  }
}

// --------------------------------------------------
// ONLY active (non-rotated) PM2 logs
// --------------------------------------------------
local.file_match "pm2_logs" {
  path_targets = [
    {
      __path__ = "/home/mark/.pm2/logs/dev.imeld.ai-error.log",
      job      = "dev.imeld.ai",
      env      = "frontend",
      host     = "error",
      stream   = "stderr",
    },
    {
      __path__ = "/home/mark/.pm2/logs/dev.imeld.ai-out.log",
      job      = "dev.imeld.ai",
      env      = "frontend",
      host     = "out",
      stream   = "stdout",
    },
  ]
}

loki.source.file "loki_pm2" {
  targets    = local.file_match.pm2_logs.targets
  forward_to = [loki.write.default.receiver]
  tail_from_end = true
}

// ===== FastAPI Logs =====
local.file_match "fastapi_logs" {
  path_targets = [
    {
      __path__ = "/var/log/fastapi/dev.imeld.ai-error.log",
      job      = "dev.imeld.ai",
      env      = "backend",
      host     = "error",
      stream   = "stderr",
    },
    {
      __path__ = "/var/log/fastapi/dev.imeld.ai-access.log",
      job      = "dev.imeld.ai",
      env      = "backend",
      host     = "out",
      stream   = "stdout",
    },
  ]
}

loki.source.file "loki_fastapi" {
  targets       = local.file_match.fastapi_logs.targets
  forward_to    = [loki.write.default.receiver]
  tail_from_end = true
}
```
- Reload config
```
sudo systemctl reload alloy
```
- Create grafana dashboard using `https://grafana.com/grafana/dashboards/13639-logs-app/`

#### Aditional tips to view API calls / usage 
- To see nice logs graph of the stats about api calls, usage and response time from nginx access logs.
- Add in nginx.conf
```
log_format api_json escape=json
'{'
 '"time":"$time_iso8601",'
 '"remote_addr":"$remote_addr",'
 '"x_forwarded_for":"$http_x_forwarded_for",'
 '"method":"$request_method",'
 '"uri":"$request_uri",'
 '"status":$status,'
 '"body_bytes_sent":$body_bytes_sent,'
 '"request_time":$request_time,'
 '"user_agent":"$http_user_agent"'
'}';
```
- Add new log file in the domain config fro only api blocks
```
access_log /var/log/nginx/access_json.log api_json;
```
- Alloy config for stats
```
// ===== API Access Logs =====
local.file_match "api_access_logs" {
  path_targets = [
    {
      __path__ = "/var/log/nginx/access_json.log",
      job      = "dev.imeld.ai",
      env      = "api_access",
      host     = "access",
      stream   = "stdout",
    },
  ]
}

loki.process "api_access" {

  stage.json {
    drop_malformed = false
    expressions = {
      time             = "time",
      remote_addr      = "remote_addr",
      x_forwarded_for  = "x_forwarded_for",
      method           = "method",
      uri              = "uri",
      status           = "status",
      request_time     = "request_time",
      body_bytes_sent  = "body_bytes_sent",
      user_agent       = "user_agent",
    }
  }

  // OPTIONAL but HIGHLY recommended to reduce cardinality
  stage.replace {
    source     = "uri"
    expression = "/[0-9a-fA-F-]{36}"
    replace    = "/:id"
  }


  stage.labels {
    values = {
      method = "method",
      status = "status",
    }
  }

  stage.timestamp {
    source = "time"
    format = "RFC3339"
  }

stage.metrics {
    metric.counter {
      name        = "http_requests_total"
      description = "Total HTTP requests"
      match_all   = true
      action      = "inc"
    }
    metric.histogram {
      name        = "http_request_duration_seconds"
      description = "HTTP request latency"
      source      = "request_time"
      buckets     = [0.1, 0.3, 0.5, 1, 2, 3, 5, 10]
    }
  }


  forward_to = [loki.write.default.receiver]
}


loki.source.file "api_access_logs" {
  targets       = local.file_match.api_access_logs.targets
  forward_to    = [loki.process.api_access.receiver]
  tail_from_end = true
}
```
