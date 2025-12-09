# Setup a blank new server (VPS)

## Setup DNS
- Login to Domain Name Server (DNS) and create a A record pointing to the server to setup a domain / subdomain. (We are using Godaddy)
- (We are adding subdomain) Go to DNS Manager and create a A record with subdomain - civic.ditinex.com
- From the server details, you will get the public IP / elastic IP, use it to create the A record.
- Verify the DNS record from [DNS Checker](https://dnschecker.org/#A/civic.ditinex.com)


## Setup Server
- SSH into the server using the server credentials. i.e : `ssh ubuntu@51.178.52.75`
- If it enforce to change password then do it, if it doesn't enforce then also change the auto generated password. 
- For security reasons, disable password based login and enable SSH login.

  #### Enable SSH Key based login
  - Create new key with command `ssh-keygen` (for mac / linux). A new private key **id_rsa** and a new public key **id_rsa.pub** will get created in `~/.ssh`
  - Copy the content of the public key `id_rsa.pub`
  - Login into the server, paste the content in a new line without removing any existing line (if any) under file : `~/.ssh/authorized_keys`
  - Now verify by logging out and login again. This time it should not ask for password. 

  #### Disable password based login
  - Inside the server, edit file : `sudo nano /etc/ssh/sshd_config`
  - Update the following values : 
  ```
  PubkeyAuthentication yes
  PasswordAuthentication no
  ```
  - Make sure SSH port is enabled in firewall (we have ufw). 
  ```
  ubuntu@vps-f351c468:~$ sudo ufw app list
  Available applications:
  OpenSSH
  ```
  ```
  sudo ufw allow "OpenSSH"
  sudo ufw enable
  ```
  - Restart the ssh service : `sudo systemctl restart ssh`

  #### Update server package and install necessary packages
  - Update package manager : `sudo apt-get update`
  - Install Nodejs, **Do not use NVM** , it create issue with CI/CD pipeline. Installing Node 22.
  ```
    cd ~
    curl -sL https://deb.nodesource.com/setup_22.x -o /tmp/nodesource_setup.sh
    sudo bash /tmp/nodesource_setup.sh
    sudo apt install nodejs -y
    node -v
  ```
  - Install PM2 service 
  ```
  sudo npm install pm2 -g
  ```
  - Install Nginx Webserver
  ```
  sudo apt update
  sudo apt install nginx -y
  sudo systemctl status nginx
  ```
  - Enable ports in firewall (our case its UFW)
  ```
  sudo ufw app list
  sudo ufw allow "Nginx Full"
  ```
  - Now the subdomain we added should resolve and show default nginx page : `http://civic.ditinex.com/` without https.


