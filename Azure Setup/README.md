# Azure Setup

#### Setup for VM and postgress
- Create a resource group from **Home > Resource Manager**
- Create a SSH key for this group from **Home > SSH keys**
- Create a VM with required specs, preferrably Linux Ubuntu latest available from **Home > Compute Infrastructure > Virtual Machines**. Select the previously created SSH key and make sure to have a public IP.
- You can verify staic IPs from **Home > Public IP**. 
- Create a postgress DB instance in Azure from **Azure Database for PostgreSQL servers**
  - From **Settings > Networking** enable public ip access and add firewall rule : `Add 0.0.0.0 - 255.255.255.255` to access it from pg admin.
  - Go to **Settings > Connect** to get the connection strings for various stack tech
  - Connect with PG Admin and create a new DB instead of using default postgres : 
    - Right-click Databases → Create → Database or with query
    ```
    CREATE DATABASE albertsons
    WITH
    OWNER = albertsons
    ENCODING = 'UTF8'
    LOCALE_PROVIDER = 'libc'
    CONNECTION LIMIT = -1
    IS_TEMPLATE = False;
    ```

