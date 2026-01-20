# Deploy React + Vite Frontend

#### Prerequisite
- If your server do not have your github account access then follow the below steps. If server is already connected then skip this Prerequisite
- Create a new ssh key inside the server `ssh-keygen`
- Copy the content of public key created at *~/.ssh/id_ed25519.pub*
- Go to [Github profile settings](https://github.com/settings/keys)
- Add the public key there under *SSH keys*


#### Deploy react repository
- Go to `/home/ubuntu/civic.ditinex.com` - the directory we created earlier for subdomain
- Run `git clone git@github.com:ditinexhosting/civic-info-frontend.git` where we clone the git repo
- Go to your cloned directory, rename the directory if needed for naming convention. Set the values of .env
- In package.json , configure to run it in port of your choice. We are using 2000 for FE and 2001 for BE
- Run `npm install`
- Run `npm run build`
- After completing build , Run `mv dist build`
- Now try running the build with `npm run production`
- If it successfully runs then set it in pm2 service : `pm2 start "npm run production" --name "civic.ditinex.com-FE" --time`
- Run `pm2 save` so that it doesn't go away on restart.
- Update the nginx config with the following to enable reverse proxy in port 2000 in the earlier created config file `/etc/nginx/sites-available/civic.ditinex.com` : 
```
location / {
           proxy_pass http://localhost:2000;
           proxy_http_version                 1.1;
           proxy_cache_bypass                 $http_upgrade;

           # Proxy headers
           proxy_set_header Upgrade           $http_upgrade;
           proxy_set_header Host              $host;

           # Proxy timeouts
           proxy_connect_timeout              60s;
           proxy_send_timeout                 3600s;
           proxy_read_timeout                 3600s;
           #try_files $uri $uri/ =404;
        }
```
- Restart nginx `sudo systemctl restart nginx`
 
