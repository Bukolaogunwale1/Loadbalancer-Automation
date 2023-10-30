# Loadbalancer-Automation

Loadbalancer-Automation

## Automating LoadBalancer configuration with Shell Scripting

> This course is an introduction to automation. This projects demonstrates how to automate the setup and maintenance of loadbalancer using a freestyle job, enhancing efficiency and and reducing manual effort.

## Automate the deployment of webservers

## Deploying and configuring servers' websites

**Step 1:** 

> Provision an EC2 instance

![image](https://github.com/Bukolaogunwale1/Loadbalancer-Automation/assets/122865359/26f8ac04-6f18-4d2f-adae-01b1a41f50fc)

> Open port 8000 to allow traffic

![image](https://github.com/Bukolaogunwale1/Loadbalancer-Automation/assets/122865359/4b400f8e-a644-43b5-b311-0d5064b7f627)

> Connect to the server uisng ssh

<img width="954" alt="ssh commad" src="https://github.com/Bukolaogunwale1/Loadbalancer-Automation/assets/122865359/8b0e994e-8d90-4af1-b747-d1db93f0f2d3">

```
sudo vi install.sh
```

paste this

```
#!/bin/bash

####################################################################################################################
##### This automates the installation and configuring of apache webserver to listen on port 8000
##### Usage: Call the script and pass in the Public_IP of your EC2 instance as the first argument as shown below:
######## ./install_configure_apache.sh 127.0.0.1
####################################################################################################################

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure

PUBLIC_IP=$1

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
```

To close this file type escape key the shift:wqa! then enter.

<img width="948" alt="vi" src="https://github.com/Bukolaogunwale1/Loadbalancer-Automation/assets/122865359/b499590e-7376-4fa8-b70d-ffcfeb0d622f">

> Change permmision on the file to make it executable

 ```
 sudo chmod +x install.sh
 ```

Run shell script using:

**./install.sh PUBLIC_IP**

```
./install.sh 18.116.15.155
```

![image](https://github.com/Bukolaogunwale1/Loadbalancer-Automation/assets/122865359/1b4780fd-d9a3-4187-835d-68777c05b0cd)

## Deployment of Nginx as a Load blalancer using shell script

### Deploying and Configuring Nginx Load Balancer

#### Setup and run shell script

> Open a file nginx.sh

```
sudo vi nginx.sh
```

> Paste code in script file

```
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
```

![image](https://github.com/Bukolaogunwale1/Loadbalancer-Automation/assets/122865359/68993c03-ca32-4520-aec8-3ca7b3fe86bd)

**Steps to run the shell script**
open a file nginx.sh

```
sudo vi nginx.sh
```

paste the script

```
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
```

to save

```
type esc the shift + :wqa!
```
To change file permission . to make it executable

```
sudo chmod +x nginx.sh
```

Run   **./nginx.sh PUBLIC_IP Webserver-1 Webserver-2**

```
./nginx.sh 3.14.134.153 13.58.64.167
```

For web 1

![image](https://github.com/Bukolaogunwale1/Loadbalancer-Automation/assets/122865359/fafd9c0c-e9d3-46d0-96f7-2ef1e2a143be)

for web 2

<img width="960" alt="welcome to y ec2 for 1" src="https://github.com/Bukolaogunwale1/Loadbalancer-Automation/assets/122865359/de61420f-5c3b-497e-90cf-7aa383d53ea6">













    









