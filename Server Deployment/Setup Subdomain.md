# Setup subdomain with nginx webserver

- Create a new index.html file at `~/civic.ditinex.com/index.html` and add a dummy html code : `<h1>Hello Civic Site</h1>`
- Name the directory same as the subdomain for easy understanding
- In our case, default user is **ubuntu** so our file `~/civic.ditinex.com/index.html` is actually located at `home/ubuntu/civic.ditinex.com/index.html`
- **OPTIONAL :** Nginx need execution permission in *home/ubuntu* directory. If you see 403 error after all setup then that is because of execution permission. To fix it, enable executable permission for */home/ubuntu* 
```
sudo chmod o+x /home
sudo chmod o+x /home/ubuntu
```


  #### Setup Nginx for Subdomain
  - Create a config file : `sudo nano /etc/nginx/sites-available/civic.ditinex.com`
  - Add the nginx configuration :
  ```
  server {

	server_name civic.ditinex.com;

	root /home/ubuntu/civic.ditinex.com;
	index index.html;

#	location / {
#           proxy_pass http://localhost:3001;
#           proxy_http_version                 1.1;
#           proxy_cache_bypass                 $http_upgrade;

           # Proxy headers
#           proxy_set_header Upgrade           $http_upgrade;
#           proxy_set_header Host              $host;

           # Proxy timeouts
#           proxy_connect_timeout              60s;
#           proxy_send_timeout                 3600s;
#           proxy_read_timeout                 3600s;
           #try_files $uri $uri/ =404;
#        }

#        location /api/ {
#           proxy_pass http://localhost:4001;
#           proxy_http_version                 1.1;
#           proxy_cache_bypass                 $http_upgrade;

           # Proxy headers
#           proxy_set_header Upgrade           $http_upgrade;
#           proxy_set_header Host              $host;

           # Proxy timeouts
#           proxy_connect_timeout              60s;
#           proxy_send_timeout                 3600s;
#           proxy_read_timeout                 3600s;
           #try_files $uri $uri/ =404;
#        }

    listen [::]:80 ipv6only=on;
    listen 80;
}
  ```
  - Go to `cd /etc/nginx/sites-enabled` and create symlink of the config file : `sudo ln -s /etc/nginx/sites-available/civic.ditinex.com`
  - Verify the syntax error `sudo nginx -t`
  - Restart nginx `sudo systemctl restart nginx`
  - Now the domain should work and render the index file : `http://civic.ditinex.com/`
  
  
  #### Install SSL for this subdomain
  - Install by command : `sudo certbot --nginx -d civic.ditinex.com`. Replace the domain name.
  - Now you can access the domain name with https

