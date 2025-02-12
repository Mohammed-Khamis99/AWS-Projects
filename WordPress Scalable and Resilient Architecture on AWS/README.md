# WordPress Scalable and Resilient Architecture on AWS

This project demonstrates the evolution of a WordPress web application from a single instance deployment to a scalable, highly available architecture on AWS. The final architecture leverages multiple AWS services to ensure resilience, auto-scaling, and separation of concerns across tiers.

# Architecture Diagram

![Image](https://github.com/user-attachments/assets/9728a460-552c-4e09-a110-6eeb457d3d09)

## 📋 Project Overview
**Evolution Journey:**
1. **Phase 1:** Manual single EC2 instance with Apache+WordPress and MySQL (monolithic)
2. **Phase 2:** Separated web tier (EC2) and database tier (RDS)
3. **Phase 3:** Introduced EFS for shared storage, Auto Scaling Groups, and Load Balancer
4. **Final Architecture:** Multi-AZ deployment with 3-tier VPC architecture

## 🏗️ Architecture Components
**AWS Network Infrastructure:**
- **VPC** with 3 Availability Zones
- **9 Subnets** (3 Public, 6 Private) across three tiers:
  - **Web Tier:** Public subnets for EC2 instances + Application Load Balancer
  - **App Tier:** Private subnets with EFS storage
  - **Database Tier:** Private subnets with RDS MySQL (Multi-AZ)

**Core Services:**
- **EC2 Auto Scaling Group:** Web servers running WordPress/PHP/Apache
- **Application Load Balancer:** Distributes traffic across web tier instances
- **Amazon EFS:** Central storage for WordPress files (wp-content)
- **RDS MySQL:** Managed relational database with Multi-AZ deployment

## 🔑 Key Features
✅ **High Availability**
- Multi-AZ deployment for all critical components
- RDS MySQL with standby replica in different AZ
- EFS storage replicated across 3 AZs

✅ **Auto Scaling**
- Web tier scales between 2-3 EC2 instances based on CPU utilization
- Automatic replacement of unhealthy instances

✅ **Security**
- Security groups with least-privilege access
- Web tier in public subnets with restricted SSH access
- App and Database tiers in private subnets
- Encrypted EFS file system and RDS storage

✅ **Resilient Storage**
- EFS provides shared file storage for WordPress
- Automatic backups with RDS snapshots
- Version-controlled WordPress core/plugins

## 🚀 Deployment Steps
**Infrastructure Setup**
### STEP 1 
•	Setup the environment which WordPress will run from.
•	Configure SSM Parameters which the manual and automatic stages of this project will use.
•	Perform a manual install of wordpress and a database on the same EC2 instance.

*Create an EC2 Instance to run wordpress* 

![Image](https://github.com/user-attachments/assets/c8b9ddb8-e5a9-460d-b6c4-9f2e59348dc8)

*Create SSM Parameter Store values for wordpress*
Storing configuration information within the SSM Parameter store scales much better than attempting to script them in some way.
1.	Create Parameter - DBUser (the login for the specific wordpress DB).
2.	Create Parameter - DBName (the name of the wordpress database).
3.	Create Parameter - DBEndpoint (the endpoint for the wordpress DB .. ).
4.	Create Parameter - DBPassword (the password for the DBUser).
5.	Create Parameter - DBRootPassword (the password for the database root user, used for self-managed admin). 

![Image](https://github.com/user-attachments/assets/bf9606a5-bfc5-44ad-a93f-33fa1fd38dfb)

*Connect to the instance and install a database and wordpress*
•	Bring in the parameter values from SSM
Run the commands below to bring the parameter store values into ENV variables to make the manual build easier.

```
DBPassword=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBPassword --with-decryption --query Parameters[0].Value)
DBPassword=`echo $DBPassword | sed -e 's/^"//' -e 's/"$//'`

DBRootPassword=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBRootPassword --with-decryption --query Parameters[0].Value)
DBRootPassword=`echo $DBRootPassword | sed -e 's/^"//' -e 's/"$//'`

DBUser=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBUser --query Parameters[0].Value)
DBUser=`echo $DBUser | sed -e 's/^"//' -e 's/"$//'`

DBName=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBName --query Parameters[0].Value)
DBName=`echo $DBName | sed -e 's/^"//' -e 's/"$//'`

DBEndpoint=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBEndpoint --query Parameters[0].Value)
DBEndpoint=`echo $DBEndpoint | sed -e 's/^"//' -e 's/"$//'`
```

•	Install updates

```
sudo dnf -y update
```

•	Install Pre-Reqs and Web Server

```
sudo dnf install wget php-mysqlnd httpd php-fpm php-mysqli mariadb105-server php-json php php-devel stress -y
```

•	Set DB and HTTP Server to running and start by default

```
sudo systemctl enable httpd
sudo systemctl enable mariadb
sudo systemctl start httpd
sudo systemctl start mariadb
```

•	Set the MariaDB Root Password

```
sudo mysqladmin -u root password $DBRootPassword
```

•	Download and extract Wordpress

```
sudo wget http://wordpress.org/latest.tar.gz -P /var/www/html
cd /var/www/html
sudo tar -zxvf latest.tar.gz
sudo cp -rvf wordpress/* .
sudo rm -R wordpress
sudo rm latest.tar.gz
```

•	Configure the wordpress wp-config.php file

```
sudo cp ./wp-config-sample.php ./wp-config.php
sudo sed -i "s/'database_name_here'/'$DBName'/g" wp-config.php
sudo sed -i "s/'username_here'/'$DBUser'/g" wp-config.php
sudo sed -i "s/'password_here'/'$DBPassword'/g" wp-config.php
```

•	Fix Permissions on the filesystem

```
sudo usermod -a -G apache ec2-user   
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www
sudo find /var/www -type d -exec chmod 2775 {} \;
sudo find /var/www -type f -exec chmod 0664 {} \;
```

•	Create Wordpress User, set its password, create the database and configure permissions

```
sudo echo "CREATE DATABASE $DBName;" >> /tmp/db.setup
sudo echo "CREATE USER '$DBUser'@'localhost' IDENTIFIED BY '$DBPassword';" >> /tmp/db.setup
sudo echo "GRANT ALL ON $DBName.* TO '$DBUser'@'localhost';" >> /tmp/db.setup
sudo echo "FLUSH PRIVILEGES;" >> /tmp/db.setup
sudo mysql -u root --password=$DBRootPassword < /tmp/db.setup
sudo rm /tmp/db.setup
```

![Image](https://github.com/user-attachments/assets/890b508a-327c-4244-8cf5-4a05be8e748a)

![Image](https://github.com/user-attachments/assets/27777628-f351-45ac-84b8-ed37c85c817b)

![Image](https://github.com/user-attachments/assets/87e03170-7f5d-4c63-8395-229fd5e9a5d0)

•	Test Wordpress is installed

![Image](https://github.com/user-attachments/assets/b8657a0e-12c0-44e9-a0d7-42d6e4d7f11c)

### STEP 2 
*Create the Launch Template*
*Add Userdata*
At this point we need to add the configuration which will build the instance Enter the user data below into the User Data box

```
#!/bin/bash -xe

DBPassword=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBPassword --with-decryption --query Parameters[0].Value)
DBPassword=`echo $DBPassword | sed -e 's/^"//' -e 's/"$//'`

DBRootPassword=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBRootPassword --with-decryption --query Parameters[0].Value)
DBRootPassword=`echo $DBRootPassword | sed -e 's/^"//' -e 's/"$//'`

DBUser=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBUser --query Parameters[0].Value)
DBUser=`echo $DBUser | sed -e 's/^"//' -e 's/"$//'`

DBName=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBName --query Parameters[0].Value)
DBName=`echo $DBName | sed -e 's/^"//' -e 's/"$//'`

DBEndpoint=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBEndpoint --query Parameters[0].Value)
DBEndpoint=`echo $DBEndpoint | sed -e 's/^"//' -e 's/"$//'`

dnf -y update

dnf install wget php-mysqlnd httpd php-fpm php-mysqli mariadb105-server php-json php php-devel stress -y

systemctl enable httpd
systemctl enable mariadb
systemctl start httpd
systemctl start mariadb

mysqladmin -u root password $DBRootPassword

wget http://wordpress.org/latest.tar.gz -P /var/www/html
cd /var/www/html
tar -zxvf latest.tar.gz
cp -rvf wordpress/* .
rm -R wordpress
rm latest.tar.gz

sudo cp ./wp-config-sample.php ./wp-config.php
sed -i "s/'database_name_here'/'$DBName'/g" wp-config.php
sed -i "s/'username_here'/'$DBUser'/g" wp-config.php
sed -i "s/'password_here'/'$DBPassword'/g" wp-config.php
sed -i "s/'localhost'/'$DBEndpoint'/g" wp-config.php

usermod -a -G apache ec2-user   
chown -R ec2-user:apache /var/www
chmod 2775 /var/www
find /var/www -type d -exec chmod 2775 {} \;
find /var/www -type f -exec chmod 0664 {} \;

echo "CREATE DATABASE $DBName;" >> /tmp/db.setup
echo "CREATE USER '$DBUser'@'localhost' IDENTIFIED BY '$DBPassword';" >> /tmp/db.setup
echo "GRANT ALL ON $DBName.* TO '$DBUser'@'localhost';" >> /tmp/db.setup
echo "FLUSH PRIVILEGES;" >> /tmp/db.setup
mysql -u root --password=$DBRootPassword < /tmp/db.setup
rm /tmp/db.setup
```
•	Launch an instance using it

![Image](https://github.com/user-attachments/assets/b8ab993d-7e33-4c2b-b8b9-1f938e953b12)

### STEP 3:

•	Create RDS Subnet Group

![Image](https://github.com/user-attachments/assets/012d803d-bc46-4dc5-9139-baed94f5487f)

•	Create RDS Instance

![Image](https://github.com/user-attachments/assets/1ee02624-2595-4b5a-ab01-85c8f7a54d4f)

Migrate WordPress data from MariaDB to RDS
•	Populate Environment Variables
You're going to do an export of the SQL database running on the local ec2 instance
First run these commands to populate variables with the data from Parameter store, it avoids having to keep locating passwords

```
DBPassword=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBPassword --with-decryption --query Parameters[0].Value)
DBPassword=`echo $DBPassword | sed -e 's/^"//' -e 's/"$//'`

DBRootPassword=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBRootPassword --with-decryption --query Parameters[0].Value)
DBRootPassword=`echo $DBRootPassword | sed -e 's/^"//' -e 's/"$//'`

DBUser=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBUser --query Parameters[0].Value)
DBUser=`echo $DBUser | sed -e 's/^"//' -e 's/"$//'`

DBName=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBName --query Parameters[0].Value)
DBName=`echo $DBName | sed -e 's/^"//' -e 's/"$//'`

DBEndpoint=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBEndpoint --query Parameters[0].Value)
DBEndpoint=`echo $DBEndpoint | sed -e 's/^"//' -e 's/"$//'`
```
•	Take a Backup of the local DB
•	Restore that Backup into RDS
Update the DbEndpoint environment variable with
```
DBEndpoint=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/DBEndpoint --query Parameters[0].Value)
DBEndpoint=`echo $DBEndpoint | sed -e 's/^"//' -e 's/"$//'`
```
Restore the database export into RDS using
```
mysql -h $DBEndpoint -u $DBUser -p$DBPassword $DBName < a4lWordPress.sql
```
•	Change the WordPress config file to use RDS
this command will substitute localhost in the config file for the contents of $DBEndpoint which is the RDS instance.
```
sudo sed -i "s/'localhost'/'$DBEndpoint'/g" /var/www/html/wp-config.php
```
•	Stop the MariaDB Service
```
sudo systemctl disable mariadb
sudo systemctl stop mariadb
```
•	Update the LT so it doesnt install
Locate and remove the following lines
```

systemctl enable mariadb
systemctl start mariadb
mysqladmin -u root password $DBRootPassword


echo "CREATE DATABASE $DBName;" >> /tmp/db.setup
echo "CREATE USER '$DBUser'@'localhost' IDENTIFIED BY '$DBPassword';" >> /tmp/db.setup
echo "GRANT ALL ON $DBName.* TO '$DBUser'@'localhost';" >> /tmp/db.setup
echo "FLUSH PRIVILEGES;" >> /tmp/db.setup
mysql -u root --password=$DBRootPassword < /tmp/db.setup
rm /tmp/db.setup
```
Click Create Template Version
Click View Launch Template
Select the template again (dont click) Click Actions and select Set Default Version
Under Template version select 2
Click Set as default version
### STEP 4:
•	Create EFS File System
•	Add an fsid to parameter store
•	Connect the file system to the EC2 instance & copy data
First we need to install the amazon EFS utilities to allow the instance to connect to EFS. EFS is based on NFS which is standard but the EFS tooling makes things easier.
```
sudo dnf -y install amazon-efs-utils
```
next you need to migrate the existing media content from wp-content into EFS, and this is a multi step process.
First, copy the content to a temporary location and make a new empty folder.
```
cd /var/www/html
sudo mv wp-content/ /tmp
sudo mkdir wp-content
```
then get the efs file system ID from parameter store
```
EFSFSID=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/EFSFSID --query Parameters[0].Value)
EFSFSID=`echo $EFSFSID | sed -e 's/^"//' -e 's/"$//'`
```
Next .. add a line to /etc/fstab to configure the EFS file system to mount as /var/www/html/wp-content/
```
echo -e "$EFSFSID:/ /var/www/html/wp-content efs _netdev,tls,iam 0 0" >> /etc/fstab
```
```
mount -a -t efs defaults
```
now we need to copy the origin content data back in and fix permissions
```
mv /tmp/wp-content/* /var/www/html/wp-content/
```
```
chown -R ec2-user:apache /var/www/
```
•	Test that the wordpress app can load the media
run the following command to reboot the EC2 wordpress instance
```
reboot
```
Once it restarts, ensure that you can still load the wordpress blog which is now loading the media from EFS.
•	Update the launch template with the config to automate the EFS part

### STEP 5:

•	Create the load balancer
•	Create a new Parameter store value with the ELB DNS name
•	Update the Launch template to wordpress is updated with the ELB DNS as its home
After #!/bin/bash -xe position cursor at the end & press enter twice to add new lines paste in this

```
ALBDNSNAME=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/ALBDNSNAME --query Parameters[0].Value)
ALBDNSNAME=`echo $ALBDNSNAME | sed -e 's/^"//' -e 's/"$//'`
```

Move all the way to the bottom of the User Data and paste in this block

```
cat >> /home/ec2-user/update_wp_ip.sh<< 'EOF'
#!/bin/bash
source <(php -r 'require("/var/www/html/wp-config.php"); echo("DB_NAME=".DB_NAME."; DB_USER=".DB_USER."; DB_PASSWORD=".DB_PASSWORD."; DB_HOST=".DB_HOST); ')
SQL_COMMAND="mysql -u $DB_USER -h $DB_HOST -p$DB_PASSWORD $DB_NAME -e"
OLD_URL=$(mysql -u $DB_USER -h $DB_HOST -p$DB_PASSWORD $DB_NAME -e 'select option_value from wp_options where option_name = "siteurl";' | grep http)

ALBDNSNAME=$(aws ssm get-parameters --region us-east-1 --names /A4L/Wordpress/ALBDNSNAME --query Parameters[0].Value)
ALBDNSNAME=`echo $ALBDNSNAME | sed -e 's/^"//' -e 's/"$//'`

$SQL_COMMAND "UPDATE wp_options SET option_value = replace(option_value, '$OLD_URL', 'http://$ALBDNSNAME') WHERE option_name = 'home' OR option_name = 'siteurl';"
$SQL_COMMAND "UPDATE wp_posts SET guid = replace(guid, '$OLD_URL','http://$ALBDNSNAME');"
$SQL_COMMAND "UPDATE wp_posts SET post_content = replace(post_content, '$OLD_URL', 'http://$ALBDNSNAME');"
$SQL_COMMAND "UPDATE wp_postmeta SET meta_value = replace(meta_value,'$OLD_URL','http://$ALBDNSNAME');"
EOF

chmod 755 /home/ec2-user/update_wp_ip.sh
echo "/home/ec2-user/update_wp_ip.sh" >> /etc/rc.local
/home/ec2-user/update_wp_ip.sh
```
Scroll down and click Create template version
Click View Launch Template
Select the template again (dont click) Click Actions and select Set Default Version
Under Template version select 4
Click Set as default version
•	Create an auto scaling group (no scaling yet)
Integrate ASG and ALB
•	Add scaling
We're going to add two policies, scale in and scale out.
•	SCALEOUT when CPU usage on average is above 40%
•	SCALEIN when CPU usage on average ie below 40%
•	ADJUST ASG Values
Set Desired 1, Minimum 1 and Maximum 3
Click Update.



















