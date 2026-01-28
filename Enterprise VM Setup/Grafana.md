
# Install and setup Grafana in Ubuntu

#### Prerequisite
- Install the prerequisite packages
```
sudo apt-get install -y apt-transport-https wget
```
- Import GPG
```
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
```
- Add stable release
```
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
- Install grafana
```
sudo apt-get update

sudo apt-get install grafana
```
- Reference grafana configs : [Grafana Official open source configs](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/)


#### Configs
- Run / Start   
```
sudo systemctl start grafana-server
sudo systemctl status grafana-server
sudo systemctl enable grafana-server.service
```
- Change port and other configs. Bind it to localhost 
```
sudo nano /etc/grafana/grafana.ini
```
- Find and change these values. I want to run all monitoring tools on ports starting 9000,9001,9002 etc
```
http_port = 9000
http_addr = 127.0.0.1
```
- Restart grafana `sudo systemctl restart grafana-server`
- Reverse proxy domain with port 9000 to access grafana. Install SSL on domain
```
location / {
           proxy_pass http://localhost:9000;
           proxy_http_version                 1.1;
           proxy_cache_bypass                 $http_upgrade;

           # Proxy headers
           proxy_set_header Upgrade           $http_upgrade;
           proxy_set_header Host              $host;
           proxy_set_header Connection        $connection_upgrade;

           # Proxy timeouts
           proxy_connect_timeout              60s;
           proxy_send_timeout                 3600s;
           proxy_read_timeout                 3600s;
           #try_files $uri $uri/ =404;
        }
```
- Open Grafana, login with "admin" as both username and password. Then change password
