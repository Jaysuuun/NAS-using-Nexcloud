# 🗃️ NAS-using-Nexcloud
**📌 Overview**

  A personal NAS (Network Attached Storage) setup using Nextcloud, hosted on lubuntu virtual machine running on virtual box, for private file sync, sharing, and media storage.

**Guest access**

  [website link](cloud.jaysun.site)
  
  **username:** guestaccount\
  **password:** nextcloudguestaccount
  
  _Please note that there maybe intances where the server is down_

## 🛠️ Hardware & System Specs
  - **Device:** Personal Computer _(Virtual Box for the server)_
  - **CPU:** Intel i5 10500  _(3 cores allocated for the vm)_
  - **RAM:** 8GB _(4GB allocated for vm)_
  - **Storage:** Virtual Memory _50GB_
  - **OS:** Lubuntu

## 🧰 Software Stack
  - **Nextcloud Version:** Nextcloud Version 31.0.5
  - **Web Server:** Apache2 Version 2.4.63
  - **Database:** mariaDB Version 12.0.1
  - **SSL:** Cloudflare Origin Certificate

Since my ISP does not allow router access, I was unable to set up traditional port forwarding. Instead, I tunneled the connection using Cloudflare Tunnel. The domain used for this project is managed via Cloudflare DNS, and SSL is handled using a Cloudflare Origin Certificate.


## 📦 SETUP
If you want to replicate the setup here is a setup guide for hosting your own NAS using nextcloud.\
_Please note that you will need to purchase yourown domain if you plan to deploy it remotely_\
You have the option to use a VM, personal computer/laptop, Virtual Private Server (VPS), or a dedicated server machine as your host. This setup guide uses lubuntu which is a linux distro, if you plan on host this locally using a windows you might need to look at different guides online as this setup is intended for linux users.\

### 1. Configure system hostname and host

  edit it using a text editor:
  
    nano /etc/hostname

  inside you will see _localhost_ change it to your domain _(e.g. cloud.jaysun.site)_

  edit the host file:
  
    nano /etc/hosts

  inside the file you will see 127.0.0.1 local host. Under it type 127.0.1.1 your domain name hostname _(e.g. 127.0.1.1 cloud.jaysun.site jay)_

### 2. Download Nextcloud

  using wget:
  
    wget https://download.nextcloud.com/server/releases/nextcloud-31.0.5.zip
  manually dowload at:
  
  [https://download.nextcloud.com/server/releases/nextcloud-31.0.5.zip](https://download.nextcloud.com/server/releases/nextcloud-31.0.5.zip)
  
### 3. Dowload and Setup MariaDB Database

  download and install:
  
    sudo apt install mariadb-server

  check status:
  
    systemctl status mariadb

  This is a security script included with MySQL and MariaDB that helps you quickly harden your database installation by walking you through basic security steps.
  
    sudo mysql_secure_installation
    
  🔐 What it does:

  - Set or change the root password (if not already set).\
  - Remove anonymous users (to prevent anyone from logging in without a user account).\
  - Disallow remote root login (optional but recommended for security).\
  - Remove the test database (which is often installed by default and accessible to all users).\
  - Reload privilege tables (to ensure changes take effect immediately).\
    
  It’s a good first step after installing MySQL or MariaDB, especially on a production or public-facing server.

  After entring the command say yes (type y) for unix_socket authnetication then create a root password for you mariadb server and finaly say Yes to all the following prompts or just press enter.\

### 4. Create a databse for nextcloud

  Enter mariadb:
  
    sudo mariadb
    
  Create a databse:
  
    CREATE DATABASE nextcloud;

  Grant All previlage on current user for the databse:
  
    GRANT ALL PREVILEGES ON nextcloud.* TO 'currentuser'@'localhost' IDENTIFIED BY 'mypassword';
    
    FLUSH PREVILEGES;

  To exit press ctrl D.\
  
  Feel free to use whatever name you want for your db if don't want to name it nextcloud.\

### 5. Install Apache2 and PHP

  Install apache and php:
  
    sudo apt install php php-apcu php=-bcmath php-cli php-common php-curl php-gd php-gmp php-imagick php-intl php-mbstring php-mysql php-zip php-xml apache2
    
  you can ommit apache2 since php installs it for you anyway but leave it if you have trust issues. 

  Check if its running:
  
    systemctl status apache2

  Open you browser and type your domain or localhost you should be seeing an Apchache2 default page. 

### 6. Install Nexcloud Files

  First unzip the nextcloud zip file, you might want to install this first:

    sudo apt install unzip
    
  then unzip the file, I suggest you rename the file to something shorter

    unzip latest.zip

  feel free to change the name of the unzipped folder to whatever you want or most preferably change it to your domain name:

    mv latest.zip cloud.jaysun.site

  Change the ownership and group of the Nextcloud folder to the current user or to the user your Apache server is running as.

    sudo chown www-data:www-data -R cloud.jaysun.site/

  dont run nextcloud from your home dir move it to somewhere else

    sudo mv cloud.jaysun.site /var/www/

  make sure to use `sudo` in case your logged in as a different user as the owner of the dir

    
### 7. Setup Apache _(make sure to change cloud.jaysun.site to whatever your domain is)_

  First things first we must disable the apache2 default page because we will be replacing that with the nextcloud page instead.

    sudo a2dissite 000-default 

  Create a config file using your domain name to tell apache which page to server:

    sudo nano /etc/apache2/sites-available/cloud.jaysun.site.conf

  Inside the file should contain this make sure to change _cloud.jaysun.site_ to whatever your domain is:

    <VirtualHost *:80>
           ServerName cloud.jaysun.site
           DocumentRoot "/var/www/cloud.jaysun.site/"
      
           <IfModule mod_headers.c>
              Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains"
          </IfModule>
           
           <Directory "/var/www/cloud.jaysun.site/">
               Options MultiViews FollowSymLinks
               AllowOverride All
               Require all granted
           </Directory>
      
           Transferlog /var/log/apache2/cloud.jaysun.site_access.log
           ErrorLog /var/log/apache2/cloud.jaysun.site_error.log

    </VirtualHost>

  After this save it and enable the config file:

      sudo a2ensite cloud.jaysun.site.conf


### 8. Setup PHP

  Configure php

      sudo nano /etc/php/8.3/apache2/php.ini

  There will be some values that we will change and uncomment (remove semi-colon ";") find these option in the ini file and change it into the following:

      memory_limit = 512M
      upload_max_filesize = 1GB (feel free to change to your prefered size)
      max_execution_time = 360
      post_max_size = 200M
      date.timezone = Asia/Manila (find this and uncomment it and set it to your timezone)
      opcache.enable = 1 (uncomment this and make sure it is set to one) 
      opache.interned_strings_buffer = 16 (uncomment and set to 16)
      opache.max_accelerated_files = 10000 (uncomment and leave it at 10000)
      opcache.memory_consumption = 128 (uncomment and leave it at 128)
      opcache.save_comments = 1 (uncomment and leave it at 1)
      opcache.revalidate_freq = 1 (uncomment and set it to 1)

  Save the file

### 9. Enable required Apache Modules

      sudo a2enmod dir env headers mime rewrite ssl

  Restart apache

      sudo systemctl restart apache2

  Now try to search to your browser:

      localhost

  you should see a nextcloud page instead of the default apache
      

If you are using a VPS, dedicated server or a personal machine and **is able** to **port forward** you might want to follow and contibue the setup at [28:12](https://www.youtube.com/watch?v=fpr37FJSgrw&t=1872s&ab_channel=LearnLinuxTV&t=1692s) otherwise if you are unable or is not allowed by your ISP to port forward you can use tunneling services like [zrok](https://zrok.io/), [ngrok](https://ngrok.com/) and etc. For this project I used [cloudflared tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/) for the free SSL certificate.

## Setup Cloudflare Tunnel

  1. Install cloudflared tunnel

          sudo apt install cloudflared

  2. Authenticate with cloudflare

          cloudflared login
     
  This opens a browser window. Choose your domain and authorize.
  3. Create a cloudflare tunnel

          cloudflared tunnel create my-tunnel
        
  4. Configure Tunnel to Local App and create a config file in `~/.cloudflared/config.yml`

          sudo nano ~/.cloudflared/config.yml

  it should look like this

          tunnel: tunnel-name
          credentials-file: /home/jaysun13/.cloudflared/tunnel-name.json

          ingress:
            - hostname: cloud.jaysun.site
              service: http://localhost
            - service: http_status:404

  5. Create a DNS route

          cloudflared tunnel route dns tunnel-name cloud.jaysun.site

  6. Run the tunnel

          cloudflared tunnel run my-tunnel


## 🔐 Security Notes

  - Fail2Ban configured: ✅
    
  - Firewall (ufw): enabled and port rules set ✅



### Please Note

  If you have trouble following this tutorial feel free to what [here](www.youtube.com/watch?v=fpr37FJSgrw&t=1110s&ab_channel=LearnLinuxTV) and follow it instead.

