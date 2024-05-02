# Implememting Load Balancers With Nginx
## Introduction

**What is Load Balancer?**

A **load balancer** is a network device or software application that efficiently distributes incoming network traffic across multiple servers. The primary purpose of a load balancer is to enhance the availability and reliability of applications by ensuring that no single server is overwhelmed with too much traffic, thus distributing the load across multiple servers.

![load balancer](<Images/load balancer.png>)

## Setting up a Basic Load Balancer

### Provisioning EC2 Instance

+ Launch 2 **EC2** instances and name them **"webserver_1"** and **"webserver_2"**.

+ Launch another **EC2** instance and name it **"load balancer"**.

+ Under security, click on security groups and edit inbound rules. Open port 8000 for both webservers and port 80 for the load balancer. Allow traffic from anywhere on the open ports.

+ Connect 2 terminals to webserver_1 and webserver_2

+ Install Apache on both webservers 

`sudo apt update -y &&  sudo apt install apache2 -y`

+ Verify that Apache has been successfully installed

`sudo systemctl status apache2`

+ Configure Apache to Port 8000

`sudo nano /etc/apache2/ports.conf`

`sudo nano /etc/apache2/sites-available/000-default.conf`

+ Reload Apache

`sudo systemctl reload apache2`

+ Open a new index.html file

`sudo nano index.html
ii. switch to nano editor and past the html file

iii. Change file ownership of index.html file

sudo chown www-data:www-data ./index.html
iv. Overriding the default html file of Apache Webserver

sudo cp -f ./index.html /var/www/html/index.html
v. Restart the webserver to load the new configuration

sudo systemctl restart apache2
Note: This step should be done for both webserver1 and webserver 2

Step 6: Install and Configure Nginx As A Load Balancer For Both WebServers

In the step, we will configure nginx as a load balancer for webserver 1 and 2

On our load balancer instance;

i. Update package lists and instal nginx

sudo apt update -y && sudo apt install nginx -y
ii. Verify that Nginx is successfully installed

sudo systemctl status nginx

iii. Edit Nginx load balancer configuration file

sudo nano /etc/nginx/conf.d/loadbalancer.conf
iv. Paste the configuration file below to configure nginx to act like a load balancer. A screenshot of an example config file is shown below: Make sure you edit the file and provide necessary information like your server IP address etc.

upstream backend_servers {

    # your are to replace the public IP and Port to that of your webservers
    server 3.128.168.233:8000; # public IP and port for webserser 1
    server 18.118.144.126:8000; # public IP and port for webserver 2

}

server {
    listen 80;
    server_name 3.143.235.90; # provide your load balancers public IP address

    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
upstream backend_servers defines a group of backend servers. The server lines inside the upstream block list the addresses and ports of your backend servers. proxy_pass inside the location block sets up the load balancing, passing the requests to the backend servers. The proxy_set_header lines pass necessary headers to the backend servers to correctly handle the requests

v. Test if nginx configuration is correct

sudo nginx -t

alt text

vi. Restart nginx

sudo systemctl restart nginx
vii. Paste load balancer public Ip address on your web browser to see the content of web server 1 and 2

