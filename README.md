Project Overview
This project demonstrates the deployment of a web server on an Amazon EC2 instance using NGINX and the integration of AWS CloudWatch for log collection. The goal was to host a static website and configure log monitoring for both access and error logs.

---

 Prerequisites
1. An AWS account with IAM permissions to create EC2 instances and access CloudWatch.
2. SSH client to connect to the EC2 instance.
3. Basic knowledge of Linux commands.

---

 Steps to Reproduce

 1. Launch an EC2 Instance
1. Create an Amazon EC2 instance.
2. Use the following user data script during instance creation:
   ```bash
   ! /bin/bash
   sudo su
   yum update -y
   amazon-linux-extras install nginx1.12
   chkconfig nginx on
   cd /usr/share/nginx/html
   chmod o+w /usr/share/nginx/html
   rm index.html
   wget https://www.free-css.com/assets/files/free-css-templates/download/page288/startup.zip
   unzip /usr/share/nginx/html/startup.zip
   sudo cp -R startup-website-template/ /usr/share/nginx/html/
   service nginx start
   ```
3. Ensure the security group allows HTTP (port 80) and SSH (port 22) access.

 2. Connect to the Instance
1. SSH into the instance using the key pair specified during creation.
   ```bash
   ssh -i <your-key.pem> ec2-user@<your-ec2-public-ip>
   ```

 3. Install and Configure AWS Logs
1. Install `awslogs` on the EC2 instance.
   ```bash
   sudo yum install -y awslogs
   ```
2. Edit the `awslogs.conf` configuration file.
   ```bash
   sudo nano /etc/awslogs/awslogs.conf
   ```
3. Add the following sections to monitor NGINX logs:
   ```
   [/var/log/nginx/access.log]
   datetime_format = %b %d %H:%M:%S
   file = /var/log/nginx/access.log
   buffer_duration = 5000
   log_stream_name = {instance_id}
   initial_position = start_of_file
   log_group_name = AccessLog

   [/var/log/nginx/error.log]
   datetime_format = %b %d %H:%M:%S
   file = /var/log/nginx/error.log
   buffer_duration = 5000
   log_stream_name = {instance_id}
   initial_position = start_of_file
   log_group_name = ErrorLog
   ```
4. Restart the AWS logs service:
   ```bash
   sudo systemctl start awslogs
   sudo systemctl enable awslogs
   ```

 4. Access the Website
1. Copy the public IP address of the EC2 instance.
2. Open a web browser and navigate to the public IP address.
3. The static website will load successfully.

 5. Verify Log Collection
1. Navigate to the AWS CloudWatch console.
2. Check the following log groups:
   - `/var/log/messages`
   - `AccessLog`
   - `ErrorLog`
3. Verify that the NGINX access and error logs are being collected.

---

 Key Components
- Amazon EC2: Virtual machine hosting the website and logs.
- NGINX: Web server serving the static site.
- AWS CloudWatch: Monitoring and log management service.
- Static Website Template: Downloaded from [Free CSS](https://www.free-css.com).

---

 Results
1. The website is accessible via the EC2 instance public IP address.
2. CloudWatch successfully collects NGINX access and error logs.
3. The project is a functional example of deploying a simple web server with integrated logging.

---

 Improvements
1. Secure the NGINX server using HTTPS with a certificate (e.g., Let's Encrypt).
2. Automate the setup further using AWS CloudFormation or Terraform.
3. Implement IAM roles and policies to restrict permissions as needed.

---

 Troubleshooting
1. Website not loading:
   - Check that NGINX is running:
     ```bash
     sudo systemctl status nginx
     ```
   - Verify the security group settings allow inbound traffic on port 80.

2. Logs not appearing in CloudWatch:
   - Ensure the AWS logs service is running:
     ```bash
     sudo systemctl status awslogs
     ```
   - Verify the IAM role attached to the instance has the `CloudWatchAgentServerPolicy` policy.

---

 Author
This project was designed and implemented as part of a hands-on learning experience in deploying cloud-based infrastructure.

