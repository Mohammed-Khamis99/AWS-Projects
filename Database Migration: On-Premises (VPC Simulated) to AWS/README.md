## Database Migration: On-Premises (VPC Simulated) to AWS
This project demonstrates a database migration strategy from a simulated on-premises environment to AWS. The "on-premises" setup is achieved by deploying resources within a Virtual Private Cloud (VPC) that mimics an isolated network, while connectivity to AWS is established using a VPC Peering connection.

## Introduction:

Migrating databases from on-premises data centers to the cloud. This project provides a practical example of how to approach such a migration by:

- Simulating an on-premises environment using an isolated AWS VPC.
- Establishing secure network connectivity between the simulated on-premises environment and the AWS cloud using VPC Peering.
- Outlining the steps for database deployment in both environments and the migration process.

## Architecture:

![Image](https://github.com/user-attachments/assets/fc871865-b642-41be-89d8-55879a9e9624)

The core architecture involves two main VPCs:

1.	"On-Premises" (Source) VPC: This VPC contains the source database server and simulates the local data center. It have its own subnets, routing tables, and security groups, completely isolated from the "AWS Target" VPC initially.
2.	"AWS Target" (Target) VPC: This VPC will host the target database (e.g., Amazon RDS, EC2 instance with a database) and any applications that will consume data from it.
3.	VPC Peering Connection: A secure and direct network connection between the "On-Premises" VPC and the "AWS Target" VPC, allowing resources in both VPCs to communicate using private IP addresses.

## Here's a high-level diagram of the architecture:

![Image](https://github.com/user-attachments/assets/4cf96ea3-3aba-4a4e-af59-fb2b3830e957)

### Features:

- Simulated On-Premises Environment: Utilizes an isolated AWS VPC to mimic a local data center.
- Secure Connectivity: Establishes private network communication between VPCs using VPC Peering.
- Database Migration Workflow: Demonstrates the essential steps for migrating a database.
- Scalable and Flexible: The architecture can be adapted for various database types and migration tools.

### Setup and Deployment:

This section outlines the manual steps for setting up the environment. For an automated approach, used AWS CloudFormation

![Image](https://github.com/user-attachments/assets/0676dd42-574e-466f-b04d-03db6b71023f)

1.	 VPC Peering Connection
-	Request Peering Connection:

- Click on Peering Connections under Virtual Private Cloud.
- Click Create Peering Connection.
- for VPC (Requester) choose onpremVPC.
- for VPC (Accepter) choose awsVPC.
- Scroll down and click Create Peering Connection.
- then click Actions and then Accept Request.
- Click Accept Request. 

![Image](https://github.com/user-attachments/assets/30a8aa7a-756e-462b-9d6d-2b6b29c6341d)

-	Accept Peering Connection:

- The request will show as "Pending Acceptance".
- Select the peering connection and click "Actions" -> "Accept Request".
-	Update Route Tables:
-	For on-prem-rt:
-	Select on-prem-rt.
-	Go to "Routes" tab and click "Edit routes".
-	Add a route:
-	Destination: 10.16.0.0/16 (CIDR of aws-target-vpc)
-	Target: Select the peering connection pcx-xxxxxx
- For aws-target-rt:
- Select aws-target-rt.
- Go to "Routes" tab and click "Edit routes".
- Add a route:
- Destination: 192.168.10.0/24 (CIDR of on-prem-vpc)
- Target: Select the peering connection pcx-xxxxxx

2.	CREATE THE RDS INSTANCE:

Deploy your source database within the on-prem-private-subnet-a.
-	Move to the RDS Console, click Subnet Groups, click Create DB Subnet Group.
-	For Name call it A4LDBSNGROUP enter the same for description in the VPC dropdown, choose awsVPC. 
-	Under availability zones, choose us-east-1a and us-east-1b
for subnets check the box next to 10.16.32.0/20 which is privateA and 10.16.96.0/20 which is private.
-	Scroll down and click Create, click on Databases, click create Database.
-	Choose Standard Create, choose MariaDB Choose Free Tier for Templates.
-	For DB instance identifier enter a4lwordpress, for Master username choose a4lwordpress.
-	For Masterpassword enter the DBPassword parameter for cloudformation which you noted down in stage of this demo enter that same password in the Confirm password box.
-	Scroll down to Connectivity, for Virtual private cloud (VPC) choose awsVPC
make sure Subnet Groups is set toe a4ldbsngroup, for public access choose No
for VPC security groups select Choose Existing and choose ***-awsSecurityGroupDB-*** (*** aren't important).
-	Remove the Default security group by clicking the X, scroll down and expand Additional configuration.
-	Under Initial database name enter a4lwordpress, scroll down and click Create Database.

![Image](https://github.com/user-attachments/assets/c8dc3dfb-a56c-4ea0-98b5-8a90dd2f2fd7)

3.	CREATE THE EC2 INSTANCE

![Image](https://github.com/user-attachments/assets/3ff4e8d8-d5b8-43f0-a95f-7557d124b286)

4.	INSTALL WORDPRESS Requirements
Select Session Manager and click Connect
When connected type sudo bash to run a privileged bash shell then update the instance with a yum -y update and wait for it to complete.
Then install the apache web server with yum -y install httpd mariadb (the mariadb part is for the mysql tools) Then install php with amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2 then make sure apache is running and set to run at startup with
///
systemctl enable httpd
systemctl start httpd

![Image](https://github.com/user-attachments/assets/5c15d40a-8090-478c-a024-2eb6eb718f75)

5.	MIGRATE WORDPRESS Content over
You're going to edit the SSH config on this machine to allow password authentication on a temporary basis.
You will use this to copy the wordpress data across to the awsCatWeb machine from the on-premises CatWeb Machine.
run a nano /etc/ssh/sshd_config
locate PasswordAuthentication no and change to PasswordAuthentication yes , then ctrl+o to save and ctrl+x to exit.
then set a password on the ec2-user user
run a passwd ec2-user and enter the DBPassword you noted down at the start of the demo.
this is only temporary.. we're using the same password throughout the demo to make things easier and less prone to mistakes
restart SSHD to make those changes with service sshd restart or systemctl restart sshd.

![Image](https://github.com/user-attachments/assets/e144455f-4d4f-4709-92d8-33b37310fd04)

6.	CREATE THE DMS SUBNET GROUP
Click Create Subnet Group
For Name and Description use A4LDMSSNGROUP for VPC choose awsVPC
for Add Subnets choose aws-privateA and aws-privateB
Click Create subnet group.
7.	CREATE THE DMS REPLICATION INSTANCE.

![Image](https://github.com/user-attachments/assets/7c135b92-ddcc-44d9-93cb-e5378259c3b0)

8.	CREATE THE DMS SOURCE ENDPOINT.

![Image](https://github.com/user-attachments/assets/160745b6-8b07-4142-a7de-ecb09811329a)

9.	CREATE THE DMS DESTINATION ENDPOINT (RDS).

![Image](https://github.com/user-attachments/assets/4e94d95f-1c36-4bdb-946d-8060786f0618)

10.	TEST THE ENDPOINTS.

![Image](https://github.com/user-attachments/assets/4dd41f50-ea61-41a3-be33-09867f4cc7ae)

11.	Migrate:

for Task identifier enter A4LONPREMTOAWSWORDPRESS for Replication instance pick the replication instance you just created
for Source database endpoint pick catdbonpremises
for Target database endpoint pick a4lwordpress
for Migration type pick migrate existing data you could pick and replicate changes here if this were a high volume production DB
for Table mappings pick Wizard
Click Add new selection rule
in Schema box select Enter a Schema
in Schema Name type a4lwordpress
Scroll down and click Create Task

This starts the replication task and does a full load from catdbonpremises to the RDS Instance.
It will create the task
then start the task
then it will be in the Running State until it moves into Load complete
At this point the data has been migrated into the RDS instance

Cutover the application instance

![Image](https://github.com/user-attachments/assets/efad4a69-e929-4523-b17f-f6b551bd0084)

![Image](https://github.com/user-attachments/assets/2dc64dc1-9ad3-46d3-9396-a72dc24402d5)

![Image](https://github.com/user-attachments/assets/56a3b645-fe83-4dcc-9d22-0ce77b87427b)
