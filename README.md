
# How to deploy MERN App on a single DigitalOcean droplet using Nginx. Wild card & subdomain

In this article, we'll learn how to deploy a React JS app using a simple server block and a Node.js app using reverse proxy server blocks, on a single DigitalOcean droplet using Nginx. Also we will learn about wild card hosting and subdomain hosting.
Step 1- Login to DigitalOcean and create a new droplet

## Access server using root
Open your terminal and copy your ip_address of your droplet and write the commend below:


```bash
  ssh root@server_ip_address
```

Now, enter your password, and you are logged into the server.

It's time to set up the Firewall.

## Basic Firewall Set up
For security reasons, we have to add a basic firewall.
Ubuntu servers use UFW firewall. It is a very easy process to set up a basic Firewall.

We can see which applications our Firewall currently allows by using the following command:
```bash
 sudo ufw app list
```
You should see the following output
```bash
Available applications
  OpenSSH
```
We have to allow SSH connections by typing:
```bash
sudo ufw allow OpenSSH
```
We have to allow SSH connections by typing:
```bash
sudo ufw allow OpenSSH
```
and then we'll enable the Firewall:
```bash
sudo ufw enable
```
Press y and ENTER.

We can see our Firewall status by using the following command:
```bash
sudo ufw status
```
Now in the next step, we'll configure the domain name.

## Step 2 - Configure Domain Name
In this section, we'll configure the domain name that will be used for our React application.

For this purpose, we have to purchase a domain(please visit GoDaddy or any other domain provider) and link your domain to the DigitalOcean.

We will be doing this step by step.

In DigitalOcean, in the "Add a Domain" section, write your domain like: sample.com. It should not www.sample.com and click the add domain button.

![see](https://res.cloudinary.com/practicaldev/image/fetch/s--0Xe1r4lW--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/afrzzzmiw40w8f2vru98.png)


After that, you have to add NS records for your domain.

We'll be adding two A records, which maps an IP4 address to the domain name.

For the first A record, enter @ in HOSTNAME and server(ie: droplet) you want to point to your domain,

![see](https://res.cloudinary.com/practicaldev/image/fetch/s--6_t6qobi--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/g8xz2c519y79k0vgyfsf.png)

For the second A record write www in HOSTNAME and select the same server

![see](https://res.cloudinary.com/practicaldev/image/fetch/s--EFDY8QzJ--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/lttjlcq8s8e2sungqthp.png)

Now go to your domain provider in my case I am using GoDaddy.
Go to your profile and in the Domain section click DNS.

In the Nameservers section click "change" and enter the following nameservers:

- ns1.digitalocean.com
- ns2.digitalocean.com
- ns3.digitalocean.com
- It may take some time to change nameservers.

## Step 3 - Install Nginx
Now your domain is pointing to the server it's time to install and configure Nginx.

## Installing Nginx
On your terminal write the following command:
```bash
sudo apt-get install nginx
```
It will install Nginx along with other dependencies.

## Configure Firewall
Before we can test Nginx, we need to reconfigure our firewall software to allow access to the service.

We can list the applications configurations that ufw knows how to work with by typing:

``` bash 
sudo ufw app list
```

You should see the following output:

``` bash 
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```
Now we will enable Nginx HTTP by typing:
``` bash
sudo ufw allow 'Nginx HTTP'
```
and we can see the changes by typing:
``` bash
sudo ufw status
```
Now we'll test Ngnix if it's working fine.

## Testing Web Server:
We can test our server by typing:
``` bash
systemctl status nginx
output should looks like this:
● nginx.service - A high performance web server and a reverse proxy server
    Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
    Active: active (running) since Mon 2016-04-18 16:14:00 EDT; 4min 2s ago
  Main PID: 12857 (nginx)
    CGroup: /system.slice/nginx.service
      ├─12857 nginx: master process /usr/sbin/nginx -g daemon on; master_process on
      └─12858 nginx: worker process
```
Now enter your ip_address into the browser and you should see the Nginx landing page.

![Output](https://res.cloudinary.com/practicaldev/image/fetch/s--Rtspk3WK--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/dx5c9s84ukgg1uicd9aq.png)

## Nginx Configuration
Open the default config file with nano or your favorite text editor:
```bash
sudo nano /etc/nginx/sites-available/default
```
Find the server_name line and replace the underscore with your domain name:
```bash
. . .

server_name example.com www.example.com;

. . .
```
Save the file and exit the editor and verify any error by typing:
```bash
sudo nginx -t
```
and then reload server by typing:
```bash
sudo systemctl reload nginx
```
Now allow access to HTTP Firewall by typing the following command:
```bash
sudo ufw allow 'Nginx Full'
```
Step 4 - SSL Configuration Using Let's Encrypt and Certbot
Let's Encrypt is a Certificate Authority (CA) that provides an easy way to obtain and install free SSL certificates, thereby enabling encrypted HTTPS on web servers. It simplifies the process by providing a software client, Certbot, that attempts to automate most (if not all) of the required steps. Currently, the entire process of obtaining and installing a certificate is fully automated on both Apache and Nginx.

## Install Certbot
First we will add the repo. to the server:
```bash
sudo add-apt-repository ppa:certbot/certbot
```
Press ENTER

Now install Certbot by typing:
```bash
sudo apt install python-certbot-nginx
```
Get SSL Certificates From Certbot
To get SSL certificates for your example.com and www.example.com URLs, use this command
```bash
sudo certbot --nginx -d example.com -d www.example.com
```
After that, Certbot will ask how you'd like to configure your HTTPS settings.

```bash
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cance
```
Select ENTER. Now your website server on HTTPS.

Now enter your domain and test it.

## Step 5 - Deploying React APP
First of all, create a folder on your website name, in my case, it's example.com in /var/www/.
```bash
sudo mkdir -p /var/www/example.com/html
```
Now go to /var/www/example.com/html by using
```bash
cd /var/www/example.com/html
```
and create index.html file by typing:
```bash
cat > index.html
```
and open it by using the following command:
```bash
nano index.html
```
Inside the file, create a basic HTML file.
```bash
<html>
  <head>
    <title>Hello World!!!</title>
  </head>
  <body>
    <h1>Success! The example.com server block is working!</h1>
  </body>
</html>
```
Save and close the file.

## Re-configuring Nginx
Now that you have the content created in the new /var/www/example.com/html directory, you need to tell Nginx to serve that directory instead of the default /var/www/html it currently is.

By using the following command add root to the file and tell Nginx the path

open the file by using:
```bash
sudo nano /etc/nginx/sites-available/default
```
and add a path to it:
```bash
root /var/www/example.com/html;
```
check any syntax error by typing:
```bash
sudo nginx -t
```
and restart Nginx
```bash
sudo systemctl restart nginx
```
Now enter your domain name and test your site.

## Deploying React App

Now open your app in the terminal and run the following command into your App's terminal:
```bash
scp -r ./build/* user@server_ip_address:/var/www/example.com/html
```
Enter the password and you are good to go.

Now open package.json file in your React App and in "scrips" section add the following code:
```bash
 "deploy-production": "react-scripts build && scp -r ./build/* user@server_ip_address:/var/www/example.com/html"
```
Write your ip_address and your website name instead of server_ip_address and example.com.

Now run the following command:
```bash
npm run deploy-production
```
Now write your domain name into the browser. If you didn't make any mistakes, your React website is deployed.

# Deploy NodeJS app on DigitalOcean using reverse proxy server blocks

We have our firewall "ufw" configured and Nginx is installed and configured, our 70% of work is done already. It will not take much time.

## Installing Node
write the following commands on the terminal:

```bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```
```bash
sudo apt install node.js
```
```bash
node --version
```

## Clone your project from GitHub
copy the link from your GitHub repo. and run the following command
```bash
git clone yourrepolink.git
```
## installing dependencies
```bash
cd yourproject
```
```bash
npm install
```
```bash
npm start (or whatever your start command)
```
 stop app
 ```bash
ctrl+C
```
## Installing PM2 to keep your app running
```bash
sudo npm i pm2 -g
```
```bash
pm2 start app.js (app.js is the file name) only start will make it
```
## aap aisa bhi kr sakte hai
```bash
pm2 start "npm run dev" --name myAppName
```
also using this command you can run next.js app without making build folder

## To make sure app starts when reboot
```bash
pm2 startup ubuntu
```
Write reboot and login to your server again by writing
```bash
ssh-copy-id bob@server_ip_address
```

Now in /etc/nginx/sites-available/default add another server block and add server_name and write your subdomain. In my case, it would be
nodejs.example.com.
```bash
    server_name nodejs.example.com
```
under the server_name add the following location part:
```bash
    location / {
        proxy_pass http://localhost:5000; #whatever port your app runs on
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
```
check any syntax error by typing:
```bash
sudo nginx -t
```
and restart the server by using the following command:
```bash
sudo service nginx restart
```
Now in DigitalOcean in the "Add a Domain" section, open CNAME and any subdomain name in my case it is node.js so you can see nodejs.example.com under HOSTNAME and select the same droplet.

## Now a React App and Node.js apps are hosted on a single DigitalOcean droplet.
I hope this article was very helpful. If you have any questions, let me know in the comment section.

I am a beginner so any suggestions from the seniors will be appreciated.
# Subdomain config & Setup

## login server

```bash
ssh root@server_ip_address
```
 ` $ Enter Password`

Now we need to remove default nginx file from sites-available
- Go into `sites-available` folder

```bash
cd /etc/nginx/sites-available
```
- Now run to remove file
```bash
rm -rf default
```
- Now back to nginx root folder

```bash
cd /etc/nginx/
```

## We will config our all domain and Subdomain in conf.d directory
We need make a conf.d directory
```bash
mkdir conf.d
```
Now we have to need edit our  `nginx.conf`  file

```bash
nano nginx.conf
```
And Just paste below snippet
```bash
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        # server_tokens off;

         server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}

```
Now go into the `conf.d` directory which we was created already
```bash
cd conf.d/
```
## Here we will add our all Domain and Subdomain
make a file as you want to call it. In my case i'm creating `domain.conf`
```bash
nano domain.conf
```
Inside `domain.conf` file we will add below snippet
```bash
server {
    listen      80;
    listen      [::]:80;

    server_name yourdomainname.com;

        access_log  /var/log/nginx/subdomain-mytym.log;
location / {
        proxy_http_version 1.1;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_pass http://localhost:8080;
        add_header Real_IP $remote_addr;
   }
    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/yourdomainname.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/yourdomainname.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server  {

#server_name yourdomainname.com www.yourdomainname.com;
    listen 80;
    return 301 https://$host$request_uri;
}
```
Now save and exit from the file using `ctrl+x` and `y` press `enter`

## Subdomain Config
Now make another file same `conf.d` directory. In my case `subdomain.conf`
```bash
nano subdomain.conf
```
Inside `subdomain.conf` file 
```bash
server {
   server_name *.yourdomainname;

        access_log  /var/log/nginx/subdomain-mytym.log;
location / {
        proxy_http_version 1.1;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_pass http://localhost:3001;
        add_header Real_IP $remote_addr;
   }
#  return 301 https://$server_name$request_uri;

  listen 443 ssl http2 default_server;
  listen [::]:443 ssl http2 default_server;

 # server_name *.yourdomainname;
  ssl_certificate /etc/letsencrypt/live/yourdomainname-0002/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/yourdomainname-0002/privkey.pem;
  include /etc/letsencrypt/options-ssl-nginx.conf;
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

#    include snippets/self-signed.conf;
#    include snippets/ssl-params.conf;

 root /root/myty-link/start.sh;
      index index.html index.htm index.nginx-debian.html;

}
```
Now test `nginx` server
```bash
nginx -t
```
You will see 
```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

## I think our nginx configuration might be completed. Now we need to start our server
Go to root folder
```bash 
cd ~
```
## If you have `next.js` app you can easily run app without making `build folder`.
Go to your app directory and run 
```bash
pm2 start "npm run dev" --name myAppName
```
If you want to start a specific file like `app.js` or `index.js` run.
```bash
pm2 start app.js
```
Now we need to save `pm2 servers`.
```bash
pm2 save
```
If you want to console of the your running app
```bash 
pm2 log
```
Restart pm2 server if you want
```bash
pm2 restart all --watch
```
Monit pm2 server
```bash 
pm2 monit
```
