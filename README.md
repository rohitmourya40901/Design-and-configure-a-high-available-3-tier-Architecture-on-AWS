# Design-and-configure-a-high-available-3-tier-Architecture-on-AWS

This guide provides step-by-step instructions for setting up a Virtual Private Cloud (VPC) on Amazon Web Services (AWS) along with creating EC2 instances, security groups, and connecting to different instances securely.

## Table of Contents
1. [Create VPC](#create-vpc)
2. [Create Bastion Host](#create-bastion-host)
3. [Create Web Server](#create-web-server)
4. [Create App Server](#create-app-server)
5. [Create DB Instance](#create-db-instance)
6. [Upload SSH Keys to Bastion Host](#upload-ssh-keys-to-bastion-host)
7. [Test Key Upload](#test-to-make-sure-key-was-uploaded-into-bastion-host)
8. [Connect to App Server](#connect-to-app-server)

---

## Create VPC
1. Go to AWS Management Console and navigate to VPC Dashboard.
2. Click on "Create VPC" and configure the following settings:
   - Name: MyVPC
   - IPv4 CIDR Block: 10.0.0.0/16
3. Create 4 subnets:
   - 1 public subnet (e.g., 10.0.1.0/24)
   - 3 private subnets (e.g., 10.0.2.0/24, 10.0.3.0/24, 10.0.4.0/24)
4. Enable public IP addresses for instances in the public subnet.
5. Ensure high availability by using 2 availability zones.
6. Allocate an Elastic IP.
7. Create a NAT Gateway.
8. Create an Internet Gateway and attach it to your VPC.
9. Configure route tables for public and private subnets, attaching the Internet Gateway to the public subnet's route table and the NAT Gateway to the private subnet's route table.
10. Create security groups for Bastion Host, Web Server, App Server, and Database.

## Create Bastion Host
1. Launch an EC2 instance using the Amazon Linux 2 AMI and t2.micro instance type.
2. Use your VPC and public subnet for the instance.
3. Configure security group for Bastion Host allowing SSH access from your IP address.
4. Connect to Bastion Host using SSH.

## Create Web Server
1. Launch another EC2 instance with the Amazon Linux 2 AMI and t2.micro instance type.
2. Use your VPC and public subnet for the instance.
3. In user data, add the following script:
   ```bash
   #!/bin/bash
   sudo yum update -y
   sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
   sudo yum install -y httpd
   sudo systemctl start httpd
   sudo systemctl enable httpd
   ```
4. Configure security group for Web Server allowing HTTP/HTTPS traffic.

## Create App Server
1. Launch an EC2 instance using the Amazon Linux 2 AMI and t2.micro instance type.
2. Use your VPC and public subnet for the instance.
3. In user data, add the following script:
   ```bash
   #!/bin/bash
   sudo yum install -y mariadb-server
   sudo service mariadb start
   ```
4. Configure security group for App Server allowing necessary inbound/outbound traffic.

## Create DB Instance
1. Create a subnet group for your database instance.
2. Launch a MariaDB database instance:
   - Standard create
   - MariaDB engine
   - Free Tier
   - Disable automated backups
   - Disable encryption
   - Set user as root with password Re:Start!9
   - Initial database name: mydb

## Upload SSH Keys to Bastion Host
### Windows Users
1. Open Command Prompt.
2. Use PSCP to upload SSH keys:
   ```bash
   Pscp -scp -P 22 -i '.\Downloads\labsuser.ppk' -l user '.\Downloads\labsuser.pem' ec2-user@bastion-host-public-ip:/home/ec2-user
   ```
   Replace labsuser.ppk, labsuser.pem, and bastion-host-public-ip with your actual key names and Bastion Host's public IP.

### Mac and Linux Users
1. Open Terminal.
2. Change permissions for SSH key:
   ```bash
   chmod 400 labuser.pem
   ```
3. Use SCP to upload SSH keys:
   ```bash
   scp -i '.\Downloads\labsuser.pem' -l user '.\Downloads\labsuser.pem' ec2-user@bastion-host-public-ip:/home/ec2-user
   ```
   Replace labsuser.pem and bastion-host-public-ip with your actual key name and Bastion Host's public IP.

## Test Key Upload
1. SSH into Bastion Host.
2. Check if SSH key is uploaded:
   ```bash
   ls
   ```
   labsuser.pem should be listed.

## Connect to App Server
1. From Bastion Host's SSH session, change file permissions for SSH key:
   ```bash
   chmod 400 labsuser.pem
   ```
2. SSH into App Server:
   ```bash
   ssh -i my-key-pair.pem ec2-user@app-server-private-ip
   ```
   Replace my-key-pair with your key name and app-server-private-ip with App Server's private IP.
3. Test connectivity:
   - Ping Web Server: `ping web-server-private-ip`
   - Connect to Database: `mysql --user=root --password='Re:Start!9' --host=database-server-endpoint`
   Replace web-server-private-ip and database-server-endpoint with actual IPs/URLs.

## Conclusion
Follow these steps carefully to set up your AWS environment with VPC, EC2 instances, security groups, and connectivity between instances. Adjust configurations and settings as per your requirements and best practices.