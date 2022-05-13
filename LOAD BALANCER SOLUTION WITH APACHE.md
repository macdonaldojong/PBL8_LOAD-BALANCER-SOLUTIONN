### Load Balancer Solution With Nginx and SSL/TLS

This project consists of two parts:

Configure Nginx as a Load Balancer
Register a new domain name and configure secured connection using SSL/TLS certificates

#### Part 1 - Configure Nginx As A Load Balancer

1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 - this port is used for secured HTTPS connections)

2. Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses

3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

###### Install Nginx

```
sudo apt update
sudo apt install nginx
```

Configure /etc/hosts on the Nginx Load Balancer server to add the web servers:
```
sudo nano /etc/hosts
```
Next step is to configure the Load Balancer to point traffic to the DNS names of the two webservers:

```
sudo nano /etc/nginx/nginx.conf

#insert following configuration into http section

upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;
```
Restart Nginx and check status:
```
sudo systemctl restart nginx
sudo systemctl status nginx
```
![Project 10a](https://user-images.githubusercontent.com/41236641/130810763-f72d0102-2ba4-4da0-9ac4-28d118436f97.JPG)

#### Register a new domain name and configure secured connection using SSL/TLS certificates

To obtain a valid SSL certificate, a domain name needs to be registered. The following steps were taken:
![Project 10](https://user-images.githubusercontent.com/41236641/130810465-8d0e7de7-4f85-4532-9710-0ea8edc8a98a.JPG)
- Registered a domain name with freenorm
- Assigned an Elastic IP to the Nginx LB server and associated the domain name with this Elastic IP 
![project10elasticipaddress](https://user-images.githubusercontent.com/41236641/130812590-5daf75b4-3fa0-4e56-b822-8c31dd068a40.JPG)
- Updated a record in the domain name registrar to point to the Nginx LB using Elastic IP address
![image](https://user-images.githubusercontent.com/41236641/130813130-dea354af-8770-43b9-a60e-9e421fd53357.png)

###### Configure Nginx to recognize your new domain name

Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com.
Check that the tooling application can be reached from the browser using new domain name using HTTP protocol - http://<your-domain-name.com>

###### Install certbot and request for an SSL/TLS certificate

Before installing certbot, ensure that the snapd service is active and running

```
sudo apt update 
sudo apt install snapd 
sudo systemctl status snapd
```
Install certbot
```
sudo snap install --classic certbot
```
Request the SSL certificate. Choose the domain that the certificate is to be issued for, domain name will be looked up from nginx.conf.

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```
![Project 10e](https://user-images.githubusercontent.com/41236641/130810958-7955d75f-f80c-4d77-aa06-941efcd736da.JPG)

The web solution should now be securely accessible at https://toolingcd.ga
![Project 10d](https://user-images.githubusercontent.com/41236641/130813799-55809a48-e96e-41bc-82e8-b867eca8eded.JPG)

Access the website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in the browser’s search string. Click on the padlock icon to find the details of the certificate issued for the website.
###### Set up periodical renewal of your SSL/TLS certificate

By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently. Test the renewal command in dry-run mode:

```
sudo certbot renew --dry-run
```
###### Setting up a cronjob for automatic renewal of the SSL certificate

Best pracice is to have a scheduled job that to run renew command periodically. Configure a cronjob to run the command twice a day.
To do so, edit the crontab file with the following command:

```
crontab -e
```

Choose nano as the editor and add the following line:
```
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```
