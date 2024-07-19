![Alt text](/Dynamic_Web_Site_on_AWS.png)

# Dynamic Website Hosting on AWS

This project demonstrates how to host a dynamic website on AWS using various AWS services to ensure reliability, fault tolerance, and security. The project includes a reference architecture diagram and deployment scripts available in the [GitHub repository](https://github.com/galkini/Dynamic_Web_Site_on_AWS).

## Architecture Overview

1. **VPC Configuration**:
    - A Virtual Private Cloud (VPC) with both public and private subnets that spanned two availability zones was configured to provide isolation and segmentation.

2. **Internet Gateway**:
    - An Internet Gateway was deployed to enable connectivity between the VPC instances and the wider Internet.

3. **Security Groups**:
    - Established Security Groups to serve as a network firewall mechanism.

4. **Availability Zones**:
    - Leveraged two Availability Zones to increase system reliability and fault tolerance.

5. **Public Subnets**:
    - Used Public Subnets for infrastructure components like the NAT Gateway and Application Load Balancer.

6. **EC2 Instance Connect Endpoint**:
    - Implemented EC2 Instance Connect Endpoint for secure connections to assets within both public and private subnets.

7. **Private Subnets**:
    - Placed web servers (EC2 instances) within Private Subnets for enhanced security.

8. **NAT Gateway**:
    - Allowed instances in both the private Application and Data subnets to access the Internet via the NAT Gateway.

9. **EC2 Instances**:
    - Hosted the dynamic website on EC2 Instances.

10. **Application Load Balancer**:
    - Used an Application Load Balancer and a target group for evenly distributing web traffic to an Auto Scaling Group of EC2 instances across multiple Availability Zones.

11. **Auto Scaling Group**:
    - Utilized an Auto Scaling Group to automatically manage EC2 instances, ensuring website availability, scalability, fault tolerance, and elasticity.

12. **Certificate Manager**:
    - Secured application communications using a Certificate Manager.

13. **SNS (Simple Notification Service)**:
    - Configured Simple Notification Service to alert about activities within the Auto Scaling Group.

14. **Route 53**:
    - Registered the domain name and set up a DNS record using Route 53.

15. **S3 Storage**:
    - Used S3 to store application codes.

## Deployment Instructions

### Prerequisites

- An AWS account with necessary permissions to create VPCs, subnets, EC2 instances, security groups, etc.
- A GitHub repository containing the deployment scripts and reference diagram.
- A registered domain name (optional but recommended).

### Database Migration to RDS Script

```bash
#!/bin/bash

S3_URI=s3://galkini-sql-files/V1__shopwise.sql
RDS_ENDPOINT=dev-rds-db.cn6omgyi84a0.us-east-1.rds.amazonaws.com
RDS_DB_NAME=applicationdb
RDS_DB_USERNAME=galkini
RDS_DB_PASSWORD=galkini123

# Update all packages
sudo yum update -y

# Download and extract Flyway
sudo wget -qO- https://download.red-gate.com/maven/release/com/redgate/flyway/flyway-commandline/10.9.1/flyway-commandline-10.9.1-linux-x64.tar.gz | tar -xvz 

# Create a symbolic link to make Flyway accessible globally
sudo ln -s $(pwd)/flyway-10.9.1/flyway /usr/local/bin

# Create the SQL directory for migrations
sudo mkdir sql

# Download the migration SQL script from AWS S3
sudo aws s3 cp "$S3_URI" sql/

# Run Flyway migration
flyway -url=jdbc:mysql://"$RDS_ENDPOINT":3306/"$RDS_DB_NAME" \
  -user="$RDS_DB_USERNAME" \
  -password="$RDS_DB_PASSWORD" \
  -locations=filesystem:sql \
  migrate
```

### Web Server Installation and Configuration Script

```bash
#!/bin/bash

# Update all the packages on the server to their latest versions
sudo yum update -y

# Install the Apache web server, enable it to start on boot, and start it immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP along with several necessary extensions for the application to run
sudo dnf install -y \
php \
php-pdo \
php-openssl \
php-mbstring \
php-exif \
php-fileinfo \
php-xml \
php-ctype \
php-json \
php-tokenizer \
php-curl \
php-cli \
php-fpm \
php-mysqlnd \
php-bcmath \
php-gd \
php-cgi \
php-gettext \
php-intl \
php-zip

# Install MySQL version 8
# Install the MySQL Community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
# Install the MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 
# Start and enable the MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Enable the 'mod_rewrite' module in Apache
sudo sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf

# Environment Variable
S3_BUCKET_NAME=dynamic-web-app-files

# Download the contents of the specified S3 bucket to the '/var/www/html' directory
sudo aws s3 sync s3://"$S3_BUCKET_NAME" /var/www/html

# Change the current working directory to '/var/www/html'
cd /var/www/html

# Extract the contents of the application code zip file that was previously downloaded from the S3 bucket
sudo unzip shopwise.zip

# Recursively copy all files, including hidden ones, from the 'shopwise' directory to the '/var/www/html/'
sudo cp -R shopwise/. /var/www/html/

# Permanently delete the 'shopwise' directory and the 'shopwise.zip' file
sudo rm -rf shopwise shopwise.zip

# Set permissions for the '/var/www/html' directory and the 'storage/' directory
sudo chmod -R 777 /var/www/html
sudo chmod -R 777 storage/

# Edit the .env file to add your database credentials
sudo vi .env

# Restart the Apache server
sudo service httpd restart
```

### Repository

Find all configuration files, diagrams, and scripts in the [GitHub repository](https://github.com/galkini/Dynamic_Web_Site_on_AWS).

## Additional Configuration

1. **Domain Name and SSL**:
    - Use AWS Certificate Manager to create and manage certificates for SSL/TLS.
    - Integrate the certificate with the Application Load Balancer.
    - Configure Route 53 to route the domain name to the Load Balancer.

2. **Notifications and Monitoring**:
    - Set up Amazon SNS for notifications about auto-scaling activities.
    - Use Amazon CloudWatch for monitoring instance health and performance metrics.

## Conclusion

By following the steps outlined above, you can successfully host a dynamic website on AWS with a robust and scalable architecture. This setup leverages various AWS services to ensure high availability, security, and efficient management of resources.
