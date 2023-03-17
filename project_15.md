# Project-15-AWS-Cloud-Solution-For-2-Company

Project 15 AWS Cloud Solution For 2 Company Websites Using A Reverse Proxy Technology

## Starting Off the AWS Cloud Project

1- Properly configure your AWS account and Organization Unit

- Create an AWS Master account. (Also known as Root Account)
  
- Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this)
  
  ![sub-account](./images-project15/sub-account.PNG)

- Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there)
  ![organisation-unit](./images-project15/organisation-unit.PNG)

- Move the DevOps account into the Dev OU.
  
  ![moved-devops-dev](./images-project15/moved-deops-to-dev.PNG)

- Login to the newly created AWS account using the new email address.
  
  Note: ** select `"Forgot Password"` to setup the new user password**

  ![login](./images-project15/sub-account-login.PNG)

2- Create a free domain name for your fictitious company at Freenom domain registrar here.

3- Create a hosted zone in AWS, and map it to your free domain from Freenom. Watch how to do that here


**Set Up a Virtual Private Network (VPC)**

Always make reference to the architectural diagram and ensure that your configuration is aligned with it.

![schematic](./images-project15/Schematic.PNG)

1. Create a `VPC`

![vpc](./images-project15/vpc.PNG)

Enable DNS hostnames

![vpc2](./images-project15/vpc2.PNG)

2. Create an `Internet Gateway`
Creating an internet Gateway and attache it to the VPC:

![igw](./images-project15/igw.PNG)

![igw](./images-project15/igw2.PNG)

   
3. Create `subnets` as shown in the architecture
   
   ![subnets](./images-project15/subnets.PNG)

4. Create a route table and associate it with `public subnets`
   
   ![public-route-table](./images-project15/Public-route-table.PNG)

   ![associate-public-subnets](./images-project15/associate-public-subnets.PNG)
   
5. Create a route table and associate it with `private subnets`
   
   ![private-route-table](./images-project15/private-route-table.PNG)

   ![associate-private-subnets](./images-project15/associate-private-subnets.PNG)


6. `Edit a route` in `public route table`, and associate it with the `Internet Gateway`. (This is what allows a public subnet to be accisble from the Internet)


![Edit-public-rtable](./images-project15/Edit-public-rtable.PNG)


7. Create `3 Elastic IPs`

8. Create a `Nat Gateway` and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)

create a Nat Gateway in public subnet

![Nat-gateway](./images-project15/Nat-gateway.PNG)

Route private subnets route table to the `Nat Gateway`

![edit-private-rtable](./images-project15/Edit-private-rtable.PNG)

9.  Create a Security Group for:

- `Nginx Servers`: Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.

![Nginx-reverse-proxy](./images-project15/Nginx-reverse-proxy-sg.PNG)
  
- `Bastion Servers`: Access to the Bastion servers should be allowed only from workstations that need to `SSH` into the bastion servers. Hence, you can use your workstation `public IP address`. To get this information, simply go to your terminal and type `curl www.canhazip.com`
  
  ![bastion](./images-project15/bastion-ssh.PNG)
  
- `Application Load Balancer`: `ALB` will be available from the Internet
  
  ![Ext-ALB](./images-project15/External-ALB.PNG)


- `Internal Application Load Balancer`: Int-ALB will get trafic from `Nginx reverse proxy`
  
  ![int-ALB](./images-project15/Internal-ALB.PNG)
  
- `Webservers`: Access to Webservers should only be allowed from the `Nginx servers`. 
  
  ![webserver](./images-project15/webserver.PNG)

- `Data Layer`: Access to the Data layer, which is comprised of Amazon `Relational Database Service (RDS)` and Amazon `Elastic File System (EFS)` must be carefully desinged – only `webservers` should be able to connect to `RDS`, while `Nginx` and `Webservers` will have access to `EFS Mountpoint`.


![data-layer](./images-project15/data-layer.PNG)


**Proceed With Compute Resources**
set up and configuration of compute resources inside the VPC. The recources related to compute are:

- EC2 Instances
- Launch Templates
- Target Groups
- Autoscaling Groups
- TLS Certificates
- Application Load Balancers (ALB)

## **Set Up Compute Resources for Nginx**

Provision EC2 Instances for Nginx

1- Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) in any 2 Availability Zones (AZ) in any AWS Region (it is recommended to use the Region that is closest to your customers). Use EC2 instance of T2 family (e.g. t2.micro or similar)

![ec2](./images-project15/ec2.PNG)

2- Ensure that it has the following software installed:

- python
- ntp
- net-tools
- vim
- wget
- telnet
- epel-release
- htop
  
3- `Create an AMI` out of the EC2 instance

Prepare Launch Template For Nginx (One Per Subnet)

1- Make use of the AMI to set up a launch template

2- Ensure the Instances are launched into a public subnet

3- Assign appropriate security group

4- Configure Userdata to update yum package repository and install nginx

## **Nginx Launch Template**

![Nginx-template1](./images-project15/Nginx-template1.PNG)

![Nginx-template2](./images-project15/Nginx-template2.PNG)

![Nginx-template3](./images-project15/Nginx-template3.PNG)

![Nginx-template4](./images-project15/Nginx-template4.PNG)

![Nginx-template5](./images-project15/Nginx-template5.PNG)

![Nginx-template6](./images-project15/Nginx-template6.PNG)


## **Configure Target Groups**

1- Select Instances as the target type

2- Ensure the protocol HTTPS on secure TLS port 443

3- Ensure that the health check path is /healthstatus

4- Register Nginx Instances as targets

5- Ensure that health check passes for the target group

 ## **Configure Autoscaling For Nginx**

1- Select the right launch template

2- Select the VPC

3- Select both public subnets

4- Enable Application Load Balancer for the AutoScalingGroup (ASG)

5- Select the target group you created before

6- Ensure that you have health checks for both EC2 and ALB

7- The desired capacity is 2

8- Minimum capacity is 2

9- Maximum capacity is 4

10- Set scale out if CPU utilization reaches 90%

11- Ensure there is an SNS topic to send scaling notifications

## **Set Up Compute Resources for Bastion**

 Provision the EC2 Instances for Bastion

1- Create an EC2 Instance based on CentOS `Amazon Machine Image (AMI)` per each Availability Zone in the same Region and same AZ where you created Nginx server

!

2- Ensure that it has the following software installed

- python
- ntp
- net-tools
- vim
- wget
- telnet
- epel-release
- htop
  ## **Bastion ami installation**

  ---
  ```
  yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm 
  
  yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm 
  
  yum install wget vim python3 telnet htop git mysql net-tools chrony -y
  
  systemctl start chronyd

  systemctl enable chronyd
  ```


3- `Associate an Elastic IP` with each of the `Bastion EC2 Instances`

## **Create a Bastion AMI**


![bastion-ami](./images-project15/bastion-ami.PNG)

## **Prepare `Launch Template` For `Bastion` (One per subnet)**

1- Make use of the `AMI` to set up a launch template

2- Ensure the Instances are launched into a public subnet

3- Assign appropriate security group

4- Configure Userdata to update yum package repository and install Ansible and git

## **Bastion Launch Template**

![bastion-template1](./images-project15/bastion-template1.PNG)

![bastion-template2](./images-project15/bastion-template2.PNG)

![bastion-template3](./images-project15/bastion-template3.PNG)

![bastion-template4](./images-project15/bastion-template4.PNG)

![bastion-template5](./images-project15/bastion-template5.PNG)

![bastion-template6](./images-project15/bastion-template6.PNG)


## **Configure Target Groups**

1- Select Instances as the target type

2- Ensure the protocol is TCP on port 22

3- Register Bastion Instances as targets

4- Ensure that health check passes for the target group
## **Configure Autoscaling For Bastion**

1- Select the right launch template

2- Select the VPC

3- Select both public subnets

4- Enable Application Load Balancer for the AutoScalingGroup (ASG)

5- Select the target group you created before

6- Ensure that you have health checks for both EC2 and ALB

7- The desired capacity is 2

8- Minimum capacity is 2

9- Maximum capacity is 4

10- Set scale out if CPU utilization reaches 90%

11- Ensure there is an SNS topic to send scaling notifications

## **`Nginx` ami installation**


```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd

```

## **configure selinux policies for the webservers and nginx servers**

```
setsebool -P httpd_can_network_connect=1

setsebool -P httpd_can_network_connect_db=1

setsebool -P httpd_execmem=1

setsebool -P httpd_use_nfs 1

```

## **This section will instll amazon efs utils for mounting the target on the Elastic file system**

```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm 

yum install -y  ./build/amazon-efs-utils*rpm

```

## **seting up self-signed certificate for the `nginx` instance**

```
sudo mkdir /etc/ssl/private

sudo chmod 700 /etc/ssl/private

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ACS.key -out /etc/ssl/certs/ACS.crt

sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

```

## **Create Nginx AMI**

![nginx-ami](./images-project15/nginx-ami.PNG)

## **Create a `Target group` for `Nginx`**

![nginx-target](./images-project15/nginx-target.PNG)


## **`webserver` ami installation**

```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd

```
## **configure selinux policies for the webservers and nginx servers**

```
setsebool -P httpd_can_network_connect=1

setsebool -P httpd_can_network_connect_db=1

setsebool -P httpd_execmem=1

setsebool -P httpd_use_nfs 1
```

## **this section will instll amazon efs utils for mounting the target on the Elastic file system**


```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm 

yum install -y  ./build/amazon-efs-utils*rpm

```
## **seting up self-signed certificate for the apache webserver instance**

```
yum install -y mod_ssl

openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/ACS.key -x509 -days 365 -out /etc/pki/tls/certs/ACS.crt

vi /etc/httpd/conf.d/ssl.conf
```
![apache-certification-config](./images-project15/apache-certificat-config.PNG)

## **Create webserver ami**

![webserver-ami](./images-project15/webserver-ami.PNG)


## **Create a `Target group` for `worldpress`**

![worldpress-target](./images-project15/worldpress-target.PNG)

## **Create a `Target group` for `tooling`**

![tooling-target](./images-project15/tooling-target.PNG)





Provision the EC2 Instances for Webservers
Now, you will need to create 2 separate launch templates for both the WordPress and Tooling websites

1- Create an EC2 Instance (Centos) each for WordPress and Tooling websites per Availability Zone (in the same Region).

2- Ensure that it has the following software installed

- python
- ntp
- net-tools
- vim
- wget
- telnet
- epel-release
- htop
- php
  
3- Create an AMI out of the EC2 instance

Prepare Launch Template For Webservers (One per subnet)

1- Make use of the AMI to set up a launch template

2- Ensure the Instances are launched into a public subnet

3- Assign appropriate security group

4- Configure Userdata to update yum package repository and install wordpress (Only required 

on the WordPress launch template)



## **TLS Certificates From Amazon Certificate Manager (ACM)**

You will need `TLS certificates` to handle secured connectivity to your `Application Load Balancers (ALB)`.

1- Navigate to `AWS ACM`

2- Request a public wildcard certificate for the domain name you registered in Freenom

3- Use DNS to validate the domain name

4- Tag the resource



## **CONFIGURE APPLICATION LOAD BALANCER (ALB)

`Application Load Balancer` To Route Traffic To `NGINX`

Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, we will benefit from intelligent routing of requests from the ALB to Nginx servers across the 2 Availability Zones. We will also be able to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate `certificates` for every request.

1- Create an Internet facing ALB

2- Ensure that it listens on HTTPS protocol (TCP port 443)

3- Ensure the ALB is created within the appropriate VPC | AZ | Subnets

4- Choose the Certificate from ACM

5- Select Security Group

6- Select Nginx Instances as the target group

![Ext-ALB](./images-project15/Ext-ALB.PNG)

![Ext-ALB2](./images-project15/Ext-ALB2.PNG)

![ALB](./images-project15/ALB-created.PNG)



Application Load Balancer To Route Traffic To Web Servers

Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scalled out or in. Nginx will not know about the new IP addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic.

To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private subnet, and we do not want direct access to them.

1- Create an Internal ALB

2- Ensure that it listens on HTTPS protocol (TCP port 443)

3- Ensure the ALB is created within the appropriate VPC | AZ | Subnets

4- Choose the Certificate from ACM

5- Select Security Group

6- Select webserver Instances as the target group

7- Ensure that health check passes for the target group

NOTE: This process must be repeated for both WordPress and Tooling websites.

![Int-ALB1](./images-project15/Int-ALB1.PNG)

![Int-ALB2](./images-project15/Int-ALB2.PNG)

![Int-ALB3](./images-project15/Int-ALB3.PNG)

![Int-ALB](./images-project15/Int-ALB.PNG)
**Setup EFS**

Amazon Elastic File System (Amazon EFS) provides a simple, scalable, fully managed elastic Network File System (NFS) for use with AWS Cloud services and on-premises resources. In this project, we will utulize EFS service and mount filesystems on both Nginx and Webservers to store data.

1- Create an EFS filesystem



2- Create an `EFS` mount target per AZ in the `VPC`, associate it with both subnets dedicated for data layer

- select the subnets of the webeservers to be mounted

  
![EFS-mount-targets](./images-project15/EFS-mount-targets.PNG)

3- Associate the Security groups created earlier for data layer.

- select the datalayer security group

![EFS-mount-targets](./images-project15/EFS-mount-targets.PNG)

4- Create an EFS access point. (Give it a name and leave all other settings as default)

![access-point](./images-project15/access-point.PNG)

create 2 access points:
- 1 for tooling server
- 1 for worldpress server
- POSIX user ID : 0 for root
- add permissions to the files
  
  ![worldpress-access-point](./images-project15/worldpress-access-point.PNG)

  ![tooling-access-point](./images-project15/tooling-access-point.PNG)


**Setup RDS**

*Pre-requisite:* Create a `KMS key from Key Management Service (KMS)` to be used to encrypt the database instance.

![RDS-KMS](./images-project15/RDS-KMS.PNG)

![RDS-KMS](./images-project15/RDS-KMS2.PNG)

`Amazon Relational Database Service (Amazon RDS)` is a managed distributed relational database service by Amazon Web Services. This web service running in the cloud designed to simplify setup, operations, maintenans & scaling of relational databases. Without RDS, Database Administrators (DBA) have more work to do, due to RDS, some DBAs have become jobless

To ensure that yout databases are highly available and also have failover support in case one availability zone fails, we will configure a `multi-AZ` set up of `RDS MySQL database` instance. In our case, since we are only using `2 AZs`, we can only failover to one, but the same concept applies to 3 Availability Zones. We will not consider possible failure of the whole Region, but for this AWS also has a solution – this is a more advanced concept that will be discussed in following projects.

To `configure RDS`, follow steps below:

1- Create a subnet group and add 2 private subnets (data Layer)

- select the avalability zone of the private subnets created
  
- select the subnets of the RDS locations

![RDS-subnet-group](./images-project15/RDS-subnet-group.PNG)

2- Create an RDS Instance for mysql 8.*.*

![create-database](./images-project15/create-database1.PNG)

![cteate-database2](./images-project15/create-database2.PNG)

![create-database3](./images-project15/create-database3.PNG)

![create-database](./images-project15/create-database4.PNG)



3- To satisfy our architectural diagram, you will need to select either Dev/Test or 
Production Sample Template. But to minimize AWS cost, you can select the Do not create a standby instance option under Availability & durability sample template (The production template will enable Multi-AZ deployment)

4- Configure other settings accordingly (For test purposes, most of the default settings are good to go). In the real world, you will need to size the database appropriately. You will need to get some information about the usage. If it is a highly transactional database that grows at 10GB weekly, you must bear that in mind while configuring the initial storage allocation, storage autoscaling, and maximum storage threshold.


5- Configure VPC and security (ensure the database is not available from the Internet)

6- Configure backups and retention

7- Encrypt the database using the KMS key created earlier

![create-database](./images-project15/create-database.PNG)

8- Enable CloudWatch monitoring and export Error and Slow Query logs (for production, also include Audit)

`Note This service is an expensinve one`. 

Ensure to review the monthly cost before creating. (DO NOT LEAVE ANY SERVICE RUNNING FOR LONG)

## **Configuring DNS with Route53**

Earlier in this project you `registered a free domain` with Freenom and `configured a hosted zone in Route53`. But that is not all that needs to be done as far as DNS configuration is concerned.

You need to ensure that the `main domain for the WordPress website can be reached`, and the `subdomain for Tooling website` can also be reached using a browser.

Create other records such as `CNAME, alias and A records`.

NOTE: You can use either `CNAME` or `alias records` to achieve the same thing. But `alias record has better functionality` because it is a faster to resolve DNS record, and can coexist with other records on that name. Read here to get to know more about the differences.

1- Create an `alias record` for the root domain and direct its traffic to the `ALB DNS name`.

2- Create an alias record for `tooling.<yourdomain>.com` and direct its traffic to the ALB DNS name.