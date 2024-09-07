# WordPress Website on AWS - DevOps Project
## Overview
This project demonstrates how to host a WordPress website on AWS, using a combination of AWS services to ensure scalability, high availability, and security. The infrastructure is deployed within an Amazon Web Services (AWS) Virtual Private Cloud (VPC), leveraging multiple Availability Zones (AZs) for fault tolerance and auto-scaling for handling traffic spikes. 

The repository includes scripts and a reference diagram that can help others replicate the deployment.

## Architecture
![Alt text].(html.png).

### Key Components:
1. **VPC**: A custom VPC configured with public and private subnets across two different availability zones to ensure high availability and fault tolerance.
2. **Internet Gateway**: Facilitates internet access for instances within the VPC.
3. **Security Groups**: Acts as a network firewall to control traffic to and from the instances.
4. **Public Subnets**: Used for components like the NAT Gateway and Application Load Balancer.
5. **Private Subnets**: Hosts the EC2 instances running the WordPress website, providing an extra layer of security.
6. **EC2 Instance Connect Endpoint**: Ensures secure connections to EC2 instances within both public and private subnets.
7. **NAT Gateway**: Allows instances within private subnets to access the internet while remaining secure.
8. **EC2 Instances**: Used to host the WordPress application.
9. **Application Load Balancer**: Distributes incoming traffic evenly across EC2 instances in different Availability Zones.
10. **Auto Scaling Group**: Automatically adjusts the number of EC2 instances based on traffic load, ensuring high availability and scalability.
11. **GitHub**: Stores website files for version control and collaboration.
12. **AWS Certificate Manager**: Secures the website using SSL/TLS certificates.
13. **Amazon SNS**: Sends notifications regarding Auto Scaling Group activities.
14. **Route 53**: Manages DNS and domain name resolution for the website.
15. **Amazon EFS**: Provides shared storage for the web servers.
16. **Amazon RDS**: Hosts the WordPress database, ensuring a reliable and fault-tolerant data layer.

## Prerequisites
Before deploying this project, you will need:
- **AWS Account**: Ensure you have an AWS account with proper permissions.
- **Domain Name**: A registered domain name for your website.
- **GitHub Repository**: Ensure your WordPress files are stored in a GitHub repository.
- **AWS CLI**: Installed and configured on your local machine for managing AWS resources.
- **Text Editor**: For editing configuration files (e.g., `wp-config.php`).

## Deployment Steps

### 1. Set up VPC and Subnets
- Configure a VPC with both public and private subnets across two Availability Zones.
- Use the provided reference diagram for guidance.

### 2. Deploy EC2 Instances
- Use the provided script to launch EC2 instances in private subnets.
- Follow the script to install and configure Apache, PHP, MySQL, and WordPress.

### 3. Set up EFS (Elastic File System)
- Mount an EFS file system to the web server directory for shared file storage.

### 4. Configure Application Load Balancer
- Set up an Application Load Balancer to distribute traffic across the EC2 instances in different Availability Zones.

### 5. Enable Auto Scaling
- Configure an Auto Scaling Group to automatically scale EC2 instances based on traffic.

### 6. Install WordPress
- Download and install WordPress using the provided script, and configure it by editing the `wp-config.php` file.

### 7. Set up SSL
- Use AWS Certificate Manager to secure the website with SSL/TLS.

### 8. Route 53 DNS Configuration
- Configure Route 53 to handle DNS resolution for your domain name.

## Deployment Script

The following script is used to deploy a WordPress website on an EC2 instance:

```bash
sudo su

# Update the software packages on the EC2 instance 
sudo yum update -y

# Create an HTML directory 
sudo mkdir -p /var/www/html

# Set the environment variable for the EFS DNS name
EFS_DNS_NAME=fs-07c0c6689dc95b7b4.efs.us-east-1.amazonaws.com

# Mount the EFS to the HTML directory 
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install Apache web server, enable it to start on boot, and start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP 8 and necessary extensions for WordPress
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# Install MySQL 8 community repository and server
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 

# Start and enable the MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set file permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html 

# Download WordPress files
wget http://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# Create the wp-config.php file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# Edit the wp-config.php file
sudo vi /var/www/html/wp-config.php

# Restart the web server
sudo service httpd restart
```

## Repository Structure

```plaintext
├── /scripts                 # Automation scripts for deployment
├── /diagrams                # Architecture diagrams for the project
└── README.md                # Project documentation
```

## AWS Services Used
- Amazon VPC
- Amazon EC2
- Amazon RDS
- Amazon EFS
- Application Load Balancer
- Auto Scaling Group
- AWS Certificate Manager
- Amazon Route 53
- Amazon SNS
- EC2 Instance Connect Endpoint

## Conclusion
This project demonstrates a fully functional WordPress website deployed on AWS with high availability, scalability, and fault tolerance. The included scripts and architecture allow for easy replication and automation.

---
