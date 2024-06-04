# Automating Loadbalancer Configuration With Shell scripting

Introduction

Automate the Deployment of Webservers
In this course i will be automating the entire process of my previous project. I will do that by writing a shell script that when ran, all that i did manually in previous project to be done automatically.

As a DevOps Engineers automation is at the heart of the work we do. Automation helps us speed the development of services and reduce the chance of making errors in our day activity.

What is automation?
Automation refers to the practice of using automated tools, processes, and workflows to streamline and accelerate the development, deployment, and operations of software applications. Automation plays a central role in DevOps practices, enabling teams to achieve faster delivery cycles, higher quality, and greater efficiency throughout the software development lifecycle.

Importance of Automation
Speed and Efficiency: Automation eliminates manual intervention in the CI/CD process, enabling rapid execution of tasks such as building, testing, and deploying software changes. Automated processes can run concurrently, in parallel, or sequentially, allowing teams to achieve faster delivery cycles and shorter time-to-market for new features and updates.

Consistency and Reliability: Automated CI/CD pipelines ensure that software changes are built, tested, and deployed consistently and reliably across different environments. Automation eliminates human errors and inconsistencies in the deployment process, leading to more predictable and stable releases.

Scalability: Automation enables CI/CD pipelines to scale up or down to handle varying workloads, changes in demand, or increases in development activity. Automated processes can be easily replicated, configured, and orchestrated to accommodate changes in project size, team size, or deployment complexity.

Quality Assurance: Automation facilitates the execution of comprehensive and repeatable tests as part of the CI/CD pipeline, including unit tests, integration tests, functional tests, performance tests, and security tests. Automated testing helps identify defects, regressions, and quality issues early in the development process, ensuring that software changes meet quality standards before deployment.

Deploying And Configuring Webservers and Load Balancer
Step 1: Launch 3 EC2 Instances We will provision 3 EC2 instance, two will of these will be our webserver and one will be a load balaner

i. Launch 2 EC2 instances and name each "WS_1" and "WS_2" (Webservers)

ii. Launch another EC2 instance and name it "load balancer"

Step 2: Open New Security Group For Both Webservers and load balancer i. For the webservers and load balancer, go to the security groups

ii Edit inbound rules on open port 8000 for our both WS_1 and WS_2 and port 80 for our load balancer

iii. Allow traffic from anywhere on the open ports

iv. ssh into the three instances

Step 3: Automating Webservers Configurartion With Shell Script

i. Paste the shell script below in a file on webserver 1 and 2 instances to configure each webservers

#!/bin/bash

####################################################################################################################
##### This automates the installation and configuring of apache webserver to listen on port 8000
##### Usage: Call the script and pass in the Public_IP of your EC2 instance as the first argument as shown below:
######## ./install_configure_apache.sh 127.0.0.1
####################################################################################################################

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure

PUBLIC_IP=$1   # place the public ip address of each webserver

[ -z "${PUBLIC_IP}" ] && echo "Please pass the public IP of your EC2 instance as an argument to the script" && exit 1

sudo apt update -y &&  sudo apt install apache2 -y

sudo systemctl status apache2

if [[ $? -eq 0 ]]; then
 sudo chmod 777 /etc/apache2/ports.conf
 echo "Listen 8000" >> /etc/apache2/ports.conf
 sudo chmod 777 -R /etc/apache2/

 sudo sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8000>/' /etc/apache2/sites-available/000-default.conf

fi
sudo chmod 777 -R /var/www/
echo "<!DOCTYPE html>
   <html>
   <head>
       <title>My EC2 Instance</title>
   </head>
   <body>
       <h1>Welcome to my EC2 instance</h1>
       <p>Public IP: "${PUBLIC_IP}"</p>
   </body>
   </html>" > /var/www/html/index.html

sudo systemctl restart apache2
alt text

Explanation of the shell script above:

#!/bin/bash : Indicating that it should be executed using the Bash shell. Comments: The script contains comments that provide explanations and usage instructions.

set -x(debug mode): command is a shell command used to enable the debugging mode.After running this command, the shell will start displaying each command along with its expanded and executed form before running it.

set -e(exit on error): When you run set -e in a shell session or script, it instructs the shell to exit immediately if any command exits with a non-zero status (indicating failure).After running this command, the shell will terminate the script or session if any command within the script exits with a non-zero status (i.e., indicates failure).

set -o pipefail: When pipefail is enabled (set -o pipefail), the exit status of a pipeline is determined by the exit status of the last command that exits with a non-zero status within the pipeline. If any command in the pipeline fails (exits with a non-zero status), the overall exit status of the pipeline will be non-zero, even if other commands in the pipeline succeed.

PUBLIC_IP=$1: The line PUBLIC_IP=$1 is a shell script command that assigns the value of the first command-line argument (denoted by $1) to a variable named PUBLIC_IP. This is a common practice in shell scripting to capture input parameters passed to a script when it is executed.

Argument Check: The script checks if the PUBLIC_IP variable is empty and prints an error message if it is, along with usage instructions. If the IP is missing, the script exits with an error code.

Update and Install Apache: The script updates the package list with sudo apt update -y and installs the Apache web server with sudo apt install apache2 -y.

Check Apache Status: It checks the status of the Apache service using sudo systemctl status apache2. If the service is running successfully (return code 0), the script proceeds with the configuration.

Configuration Changes: The script makes the following configuration changes to Apache:

It grants full read and write permissions to /etc/apache2/ports.conf to allow editing the configuration file.

It appends Listen 8000 to the end of /etc/apache2/ports.conf, which instructs Apache to listen on port 8000.

It grants full read and write permissions recursively to the /etc/apache2/ directory.

It replaces the <VirtualHost *:80> line with <VirtualHost *:8000> in the Apache default site configuration file (/etc/apache2/sites-available/000-default.conf), ensuring Apache listens on port 8000.

Set Up Web Page: The script grants full read and write permissions recursively to the /var/www/ directory and creates an HTML file (index.html) in the default web directory (/var/www/html). This HTML file displays a simple web page with the instance's public IP address.

Restart Apache: Finally, the script restarts the Apache service with sudo systemctl restart apache2 to apply the configuration changes.

alt text

Step 4: Automating Load Balancers Configurartion With Shell Script On our load balancer instance;

i. Open a file and paste the shell script below

#!/bin/bash

######################################################################################################################
##### This automates the configuration of Nginx to act as a load balancer
##### Usage: The script is called with 3 command line arguments. The public IP of the EC2 instance where Nginx is installed
##### the webserver urls for which the load balancer distributes traffic. An example of how to call the script is shown below:
##### ./configure_nginx_loadbalancer.sh PUBLIC_IP Webserver-1 Webserver-2
#####  ./configure_nginx_loadbalancer.sh 127.0.0.1 192.2.4.6:8000  192.32.5.8:8000
############################################################################################################# 

PUBLIC_IP=$1
firstWebserver=$2
secondWebserver=$3

[ -z "${PUBLIC_IP}" ] && echo "Please pass the Public IP of your EC2 instance as the argument to the script" && exit 1

[ -z "${firstWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the second argument to the script" && exit 1

[ -z "${secondWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the third argument to the script" && exit 1

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure


sudo apt update -y && sudo apt install nginx -y
sudo systemctl status nginx

if [[ $? -eq 0 ]]; then
sudo touch /etc/nginx/conf.d/loadbalancer.conf

sudo chmod 777 /etc/nginx/conf.d/loadbalancer.conf
sudo chmod 777 -R /etc/nginx/


 echo " upstream backend_servers {

        # your are to replace the public IP and Port to that of your webservers
        server  "${firstWebserver}"; # public IP and port for webserser 1
        server "${secondWebserver}"; # public IP and port for webserver 2

        }

       server {
        listen 80;
        server_name "${PUBLIC_IP}";

        location / {
            proxy_pass http://backend_servers;   
        }
 } " > /etc/nginx/conf.d/loadbalancer.conf
fi

sudo nginx -t

sudo systemctl restart nginx
alt text

alt text

alt text

Thank you!