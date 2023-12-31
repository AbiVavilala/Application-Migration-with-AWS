# Application Migration with AWS

## Table of Contents

- [Introduction](#introduction)
 - [Migration Steps](#migration-steps)
   - [1. Assessment and Planning](#1-assessment-and-planning)
   - [2. AWS Account Setup](#2-AWS-Account-Setup)
   - [3. Cloud Formation template](#3-CloudFormation-template)
   - [4. Database Migration](#4-Database-Migration)
   - [5. Server Migration](#5-Server-Migration)
- [Resources](#resources)
- [License](#license)

## Introduction
I Migrated Onprem Wordpress Application to AWS. I have documented my work. This project  provides comprehensive documentation for the process of migrating an application to the Amazon Web Services (AWS) cloud platform. I followed the steps outlined below for Migrating Onprem Wordpress Application to AWS.

This document outlines the steps and best practices for migrating your application to AWS. AWS provides a range of services and tools that can help you seamlessly migrate your application to the cloud. By following this guide, you will ensure a smooth transition while taking advantage of AWS's scalability, reliability, and cost-effectiveness.


## Migration Steps

### 1. Assessment and Planning

### Source Environment
- Current Environment consist of a three tier e-commerce application (Unicorn Store Application); a webserver running ubuntu with apache, PHP, Wordpress/ WooCommerce and a database server running Ubuntu with MySQL version 5.7.
![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/source-env.png)
- The Onprem application has a webserver and database running on server. I will replatform database with AWS Database Migration Service. I will Rehost (lift-and-shift) my webserver to AWS using AWS's leading migration service Application Migration Service (AWS MGN).

   
### Migration Strategy
in this migration, My strategy is to rehost Wordpress application on EC2 instance and Migrate onprem Mysql database to AWS managed Mysql RDS. this is lift and shift strategy.
-  I will rehost the webserver with AWS Application Migration Service. I will migrate Source database using AWS Database Migration service.
-  I will follow AWS Well Architected Framework to improve Operation Excellence, Security, Performance Efficency and Cost Optimization for My application in AWS.

### 2. AWS Account Setup

- Configure AWS Identity and Access Management (IAM) roles and permissions for IAM user needed for this migration.
  
- AWS IAM user has been created with Administrator access permission added. Logged  into AWS using IAM User.
![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/aws-sinign.PNG)


 ## 3. Cloud Formation to create source and target environment

I Uploaded AWS cloud formation template which will create a source environment and target environment as required for this project. 

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/cloudformation1.PNG)

click on next

I entered the details of the stack

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/cloudformation2.PNG)

once stack is submitted the required resource will be created. you can check the output of stack to see the output. both source and target environment is craeted.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/cloudformation3.PNG)
 
###  Target Environment

- The following target Amazon Virtual private cloud (VPC) is deployed during the environment preparation.
![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/target-vpc.png)

The VPC consists of 6 subnets (x2 public, x2 private for webservers and x2 private for database) across two availability zones. nat gateway is deployed in two public subnets in AZs and internet gateway is deployed using this cloud formation template.


## 4. Database Migration

I will migrate Source Datbase hosted on EC2 instance to AWS managed MySQL RDS
![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourcedb.PNG)

### Set Up Networking

I will migrate Source database to Target database using DMS replication instance. DMS replication instance will need to connect to source database over public internet, while to the target database over private network. I will setup required networking before creating replication instance and Target database

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/Set-Up-Networking.png)

I created a security group for  allowing outbound traffic from DMS Replication instace

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/RI-SG1.PNG)


I created a security group to allow inbound traffic from DMS Replication instance Security Group (RI-SG). Create a rule to allow inbound traffic from DMS Replication instance Security group.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/DB-SG.PNG)

 

I will create a subnet group where target RDS will be deployed. I will install RDS in private subnet in Target VPC

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/subnetgroup1.PNG)


I selected databases from the menu and on the left click create database. select Mysql as engine options and Engine version MySQL 5.7.41

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/targetdb1.PNG)

For this project I selected Free tier as I want to keep cost low.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/targetdb2.PNG)

In settings section I configured DB Instance identifier, then Master username and password for new database

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/targetdb3.PNG)

In storage section, I selected db.t3.micro from the Burstable DB instance vclass and allocated 20 GB of storage for this database.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/targetdb4.PNG)

In connectivity section, I selected target VPC, in VPC security group select DB-SG. make sure subnet group we craeted is auto populated.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/targetdb5.PNG)
![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/targetdb6.PNG)

In Database authentication select password authentication
![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/targetdb7.PNG)

In additional configuration select Enable encryption and untick Enable automated backups to reduce cost.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/targetdb8.PNG)
 
Click on Craete a database.
![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/10.png)


### Create replication Instance

One of the pre-requisites for using of AWS DMS is having configured a subnet group, which is a collection of public subnets that will be used by the DMS Replication Instance.

Go to AWS Console > Services > Migration & Transfer > Database Migration Service > Subnet groups and click on Create subnet group button.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/replication-instance.PNG)

give a name for subnet group and select Target VPC and add Two public subnets for Replication instance to launch so that they can communicate with source server on public network. 

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/replication-instance1.PNG)

In this Step, we will create DMS replication instance. 

Inside AWS Console, go to Services and Database Migration Service. Click on Replication instances and then on the Create replication instance button.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/replication-instance2.PNG)


On the Create replication instance screen I configured a new replication instance with values 

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/replication-instance3.PNG)

I selected the values as needed for my migration.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/replication-instance4.PNG)


Click on Create instance

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/replication-instance5.PNG)

I made changes to Source database security group and allowed all traffic from internet for Repliation Instance to reach Source database.

### Configure Source Database

For AWS DMS continuous replication from MySQL database, you'll need to enable the binary log and make configuration changes on the source database. Please see the configuration changes made on source database

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/Inkedconfigure-source-database.jpg)

I tested the changes to make sure update in /etc/mysql/my.cnf took effect, by running the following commans

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/configure-source-database1.PNG)

### Create Source and target endpoint

In AWS console search for WS Database Migration Service, click on Endpoints and Create endpoint button.

Create Target Endpoint

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/target-endpoint.PNG)

Will test target Endpoint connect my selecting Target VPC and replication instance we just created. please see image below.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/target-endpoint1.PNG)

Create Source Endpoint and run connection test

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/source%20endpoint%20connection%20test.PNG)

### Create and run replication

In AWS DMS console, go to Database migration tasks and click the Create Task button.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/databasereplicationtask.PNG)

In Create database migration task screen fill with the required information

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/databasereplicationtask1.PNG)

In Task Setting Wizard enter the following values

Parameter	Value
- Target table preparation mode: Do nothing
- Validation:Turn on
- Task Logs:Turn on

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/databasereplicationtask2.PNG)

In the Table mappings panel select Wizard mode, press the Add new selection rule button and select wordpress-db in the Schema drop-down.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/databasereplicationtask4.PNG)

In the Migration task startup configuration, select Automatically on create then Scroll to the bottom of the screen and click the Create task button to create the task and start the data replication.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/databasereplicationtask5.PNG)

Monitor the task until the status is changed to Load complete, replication ongoing and the progress is 100%

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/replicationtask.PNG)


## 5. Server Migration

 After succesfully migrating database to AWS. I am migrating Onprem webserver to AWS. This is straight lift and shift migration. in this I am only rehosting the application

### Re-host with Application Migration Service

- Go to AWS Application Migration Service console .

- Click on Get started button, and then Set up service. This will create the necessary IAM resources and also the default templates.

- At the MGN console, go to Replication template menu at Settings section and edit the default template to change the staging area's subnet.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/AWSMGN.PNG)

Click on Set up service

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/AWSMGN1.PNG)

Template gets created and click on edit template

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/AWSMGN3.PNG)

select public subnet and save template

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/AWSMGN4.PNG)


### Install Agent

AWS Application Migration Service replicates data to AWS using agent that must be installed on the source server.

For the Application Migration Service agent to replicate data, you need to have an AWS IAM User with proper privileges created in your target AWS account. I created them in the cloud formation stack. 

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/accesskey.PNG)

Connect to the source server using ssh key.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/installagent1.PNG)

I ran the following commands

cd ~
wget -O ./aws-replication-installer-init.py https://aws-application-migration-service-us-west-2.s3.amazonaws.com/latest/linux/aws-replication-installer-init.py
sudo python3 aws-replication-installer-init.py


Provide AWS region and AWS credentials

When prompted provide AWS Region (us-west-2), then AWS Access Key ID and AWS Secret Access Key from cloudformation template.
Provide AWS region and AWS credentials

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/installagent.png)

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/installagent3.png)


Replication starts 

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver.PNG)


Click on Source Server name and it will take yo to following page. on this page I monitor replication progress.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver1.PNG)


In Server Info we can find retrived information about source server. We can view disk settings, launch settings.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver2.PNG)

In Lifecycle, clikc on launch settings and click on Edit.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver3.PNG)

On launch settings page change instance type right sizing to off and then click on Save.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver4.PNG)

Modify EC2 launch Template. click on Modify EC2 launch template section and confirm by clicking on Modify.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver5.PNG)

Scroll to the instance type and change to T3.micro.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver6.PNG)

Scroll down to network settings and select TargetVPC-public-a as the Subnet - this is where we want our webserver to run after the migration.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver7.PNG)

In network settings, create security group. create new SG and allow all HTTP and HTTPS traffic.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver7.PNG)

Then expand the Advanced network configuration section, and for the first Network interface, set Auto-assign public IP to Enable to make sure webserver will be accessible over public IP.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver8.PNG)

In the Resource tags section change the Name tag value to Webserver, like on the screenshot below.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver9.PNG)

Finally in the Advanced details section, select the following role as IAM Instance profile: AWSApplicationMigrationLaunchInstanceWithSsmRole.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver10.PNG)

Scroll down and click on create template version.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver11.PNG)

update default EC2 version in Launch template. In the lversion tab slect the latest version and set it as default version. This template will be used to launch our new Migrated Web Server.
![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver12.PNG)

You will see following message

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver14.PNG)


### Launch test instance

Now my source server is ready for test. 

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver1.PNG)

I will launch a test instance by clicking on launch test instance by clikcing on test and cutover button

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/Inkedfinalizecutover.jpg)

Wait for the Completed Status (it should take ~10-15 minutes).

![]( https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/testing_job_details.png)

Once testing is completed look for the events

you will see succesfully launched test/cutover ec2 instance. This process validates conversion of boot volume and target server network configuration.

In souce server,  click on your server Hostname, then Test and Cutover -> Mark as "ready for cutover".

![]( https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/ready_for_cutover.png)

Continue on the popup to confirm that the testing went succesful and testing instance can be terminated.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/ready_for_cutover_pop-up.png)


### launch cutover instance

Select hostname of the Source server and click on Launch cutover instances.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/launch_cutover_instance.png)


you will get following popup. please click on launch

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/launchcutoverinstance.PNG)

This will trigger the cutover job (it will take ~10-15 minutes), you can find its details after clicking on Cutover Job ID in the Lifecycle section.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/launchcutoverinstance1.PNG)

 
Wait for the Launch status to change from Waiting to Launched / First boot: Started. At this stage Webserver will be visible and Running in the EC2 Console.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/launchec2.PNG)

On source server click on finalize cutover. this will stop replication and terminate source server assoiciated with migration.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/finalizecutover.PNG)

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/finalize_cutover_pop-up.png)



### Configure application.

When the Cutover is finished and Application Migration Service has created a running instance of the Webserver in my AWS account, it's time to update the web application configuration to use replicated AWS RDS database (created in the Database Migration step).

Will update Webserver security group to accept traffic from anywhere to Port 80, 443 and 22.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/application%20config.PNG)

 Click Edit inbound rules. Make sure traffic from anywhere to port 80 (HTTP) is allowed. Add rule to allow SSH from anywhere to port 22 (SSH). Finally click Save rules.

 Go to Networking tab and make a note of Public DNS (IPv4) address and Private IP

 Connect via SSH to the Webserver created by the Application Migration Service

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/applicationconfiguration2.png)

Modified wordpress configuration and add the following two lines, replacing TARGET_WEBSERVER_PUBLIC_DNS with your Target Webserver EC2 Public DNS (IPv4), to make sure links in your wordpress site point to the new webserver.

    define('WP_SITEURL', 'http://ec2-54-244-168-116.us-west-2.compute.amazonaws.com');
    define('WP_HOME',    'http://ec2-54-244-168-116.us-west-2.compute.amazonaws.com');

Update the RDS instance VPC security group to allow inbound traffic from Webserver

a. Go to AWS Console > Services > EC2 > Security Groups and select your RDS VPC security group (DB-SG)
b. Go to the Inbound tab and click the Edit button
c. Add inbound rule that allows traffic from the Webserver (using its Private IP or the security group it belongs to) on port 3306 (MySQL port)


![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/applicationconfig1.PNG)

Validate the migration

Open the Webserver Public DNS (IPv4) name in your web browser, will see a unicorn store.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/applicationconfig5.png)


This is how I performed life and shift migration of my wordpress unicorn store application.
 

## Resources

- [AWS Migration Hub](https://aws.amazon.com/migration/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Documentation](https://docs.aws.amazon.com/)
- [Application Migration with AWS](https://catalog.us-east-1.prod.workshops.aws/workshops/c6bdf8dc-d2b2-4dbd-b673-90836e954745/en-US)
 