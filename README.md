# Project Name: MERN Application Deployment

## Table of Contents
1. [About MERN App](#about-mern-app)
2. [Initial Infrastructure Setup](#initial-infrastructure-setup)
3. [Application Deployment with Docker](#application-deployment-with-docker)
4. [Security and Monitoring Setup](#security-and-monitoring-setup)
5. [Monitoring Demo](#monitoring-demo)

---
## About MERN App

- This is a simple MERN Todo notes App, in which we can save our tasks and can delete them if we don't need them.
- These tasks are stored in monogdb database  

### Screenshots

#### Frontend
![App Screenshot](https://drive.google.com/uc?id=1lR-RKZFiH-fkITn6pL-X2JH_fybm6ZfH "Frontend")

#### Database 
![App Screenshot](https://drive.google.com/uc?id=17xwt9ZuSx0v4-iHgAkWh4318ZBsP5Pbw "Database")
## Initial Infrastructure Setup

### Steps
1. **Connect to the server using SSH:**
    ```bash
    ssh root@<Public-IP>
2. **Update the package list:**
    ```bash
    apt update
    apt upgrade
3. **Install Docker and Docker Compose:**
    ```bash
    apt install docker.io docker-compose
4. **Create a new user and provide sudo privileges:**
    ```bash
    adduser asad
    usermod -aG sudo asad
    su asad
    sudo usermod -aG docker $USER 
5. **Configure firewall**
    ```bash
    sudo ufw allow OpenSSH
    # We will open other ports as we need them 
    sudo ufw enable
    sudo service sshd restart

## Application Deployment with Docker

### Steps
1. **Clone the MERN sample project from GitHub:**
    ```bash
    git clone https://github.com/AsadSubhan/MERN-TodoApp.git
2. **Create Dockerfile for React and Nodejs.** (React Dockerfile is present in /frontend dir and Nodejs Dockerfile is present in /backend dir)
3. **Create docker-compose.yaml file** with 3 services for react running on 3000, nodejs running on 5000, mongodb running on 27017. Then, run this docker compose file:
    ```bash 
    docker-compose up -d
4. **Configure Nginx to serve the React application:**
    ```bash
    sudo apt install nginx
    sudo vim /etc/nginx/sites-available/react-app
    # Nginx config file is present in Config.txt

    sudo ln -s /etc/nginx/sites-available/react-app /etc/nginx/sites-enabled/
    cd /etc/nginx/sites-available
    sudo rm -rf default
    cd /etc/nginx/sites-enabled
    sudo rm -rf default

    # Allowing port for Nginx in firewall
    sudo ufw allow 'Nginx Full'

## Security and Monitoring Setup

### Steps
1. **Set up SSL/TLS certificates for secure communication(Self-Signed Certificate):**

    ```bash
    sudo mkdir -p /etc/nginx/ssl
    sudo openssl req -x509 -nodes -newkey rsa:2048 -keyout /etc/nginx/ssl/self-signed.key -out /etc/nginx/ssl/self-signed.crt -days 365

    #It will ask for details like country, state, city, domain name etc. Currently we don't have domain name so we will leave it blank, and in Nginx config file we will use public IP address.

    sudo systemctl restart nginx
2. **Implement server monitoring with Prometheus and Grafana:**
- #### Install and Setup Prometheus

  ``Prometheus is an open-source monitoring solution for collecting and aggregating metrics as time series data``

    ```bash
    # Download Prometheus
    wget https://github.com/prometheus/prometheus/releases/download/v2.47.2/prometheus-2.47.2.linux-amd64.tar.gz
    # Unzip file
    tar xzf prometheus-2.47.2.linux-amd64.tar.gz

    # Create Prometheus service 
    mv prometheus-2.47.2.linux-amd64 /etc/prometheus
    sudo vim /etc/systemd/system/prometheus.service
    # Prometheus systemd file is present in Config.txt
    
    # Configure alerts for key metrics in Prometheus
    cd /etc/prometheus/
    sudo vim alert.rules.yml  
    # alert.rules.yml file is present in Config.txt

    sudo systemctl daemon-reload
    sudo systemctl restart prometheus
    sudo systemctl enable prometheus

    # Allowing port for Prometheus in firewall (Best security practice is to not open this port for every IP Address i.e 0.0.0.0, instead we should open it for specific IP)
    sudo ufw allow 9090/tcp
    
- #### Install and Setup Node-Exporter

  ``Node Exporter is a Prometheus exporter specifically designed to collect metrics from Linux and Unix-like systems. It provides detailed information about the system’s hardware and operating system.``

    ```bash
    # Download Node-Exporter
    wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
    # Unzip file
    tar xzf node_exporter-1.6.1.linux-amd64.tar.gz

    # Create Node-Exporter service
    sudo mv node_exporter-1.6.1.linux-amd64 /etc/node_exporter
    sudo vim /etc/systemd/system/node_exporter.service
    # Node-Exporter systemd file is present in Config.txt

    sudo systemctl daemon-reload
    sudo systemctl restart node_exporter
    sudo systemctl enable node_exporter

    # Connecting Prometheus and Node-Exporter
    sudo rm -rf /etc/prometheus/prometheus.yml
    sudo vim /etc/prometheus/prometheus.yml
    # Prometheus scrape file is present in Config.txt

    sudo systemctl restart prometheus


- #### Install and Setup Grafana

  ``Grafana is a tool which transforms metrics into visualization and dashboards.``
  

  ```bash
  # Installing utilities and downloading Grafana
  sudo apt-get install -y adduser libfontconfig1 musl
  wget https://dl.grafana.com/enterprise/release/grafana-enterprise_10.2.0_amd64.deb
  sudo dpkg -i grafana-enterprise_10.2.0_amd64.deb

  sudo systemctl stop grafana-server
  sudo vim /etc/grafana/grafana.ini
  # Change Grafana default port from 3000 to 4000, because React is running on 3000 port(http_port = 4000)

  sudo systemctl restart grafana-server
  sudo systemctl enable grafana-server

  # Allowing port for Grafana in firewall (Best security practice is to not open this port for every IP Address i.e 0.0.0.0, instead we should open it for specific IP)
    sudo ufw allow 4000/tcp

## Monitoring Demo

- We have Prometheus accessible on <PublicIP>:9090, where we have configured alerts for metrics like High CPU Utilization and Low Disk Space
- Grafana is accesible on <PublicIP>:4000, by default username and password both are "admin", when we login first it prompts us to change password. Then we can connect it with prometheus for metrics collection for visualization purpose.

### Screenshots

#### Prometheus Alerts

![App Screenshot](https://drive.google.com/uc?id=1VEC1r7_aXENh9dkKjOoan2hgZn_ajmVA "Prometheus Alerts")

#### Grafana Visualization dashboard

![App Screenshot](https://drive.google.com/uc?id=1YpJ2KScjDHWQWOtNVgqWRUiCWZFzidSX "Grafana Dashboard")


## Authors

- [@AsadSubhan](https://www.linkedin.com/in/asadsubhan9/)

