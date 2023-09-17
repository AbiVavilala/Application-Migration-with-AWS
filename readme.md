# Application Migration with AWS

This README.md file provides comprehensive documentation for the process of migrating an application to the Amazon Web Services (AWS) cloud platform. I followed the steps outlined below for Migrating Onprem Wordpress Application to AWS.

## Table of Contents

- [Introduction](#introduction)
 - [Migration Steps](#migration-steps)
   - [1. Assessment and Planning](#1-assessment-and-planning)
   - [2. AWS Account Setup](#2-AWS-Account-Setup)
   - [3. Cloud Formation template](#3-CloudFormation-template)
   - [4. Database Migration](#4-Database-Migration)
   - [5. Server Migration](#5-Server-Migration)
   - [6. Testing and Validation](#6-testing-and-validation)
- [Monitoring and Optimization](#monitoring-and-optimization)
- [Security Considerations](#security-considerations)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)
- [License](#license)

## Introduction

This document outlines the steps and best practices for migrating your application to AWS. AWS provides a range of services and tools that can help you seamlessly migrate your application to the cloud. By following this guide, you will ensure a smooth transition while taking advantage of AWS's scalability, reliability, and cost-effectiveness.


## Migration Steps

### 1. Assessment and Planning

### Source Environment
- Current Environment consist of a three tier e-commerce application; a webserver running ubuntu with apache, PHP, Wordpress/ WooCommerce and a database server running Ubuntu with MySQL version 5.7.
![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/source-env.png)
- The Onprem application has a webserver and database running on server. I will replatform database with AWS Database Migration Service.

   
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

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/installagent3.PNG)


Replication starts 

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/sourceserver.PNG)


### 6. Testing and Validation

- Conduct thorough testing of your migrated application.
- Perform load testing to ensure scalability.
- Monitor application performance and optimize as needed.

## Monitoring and Optimization

- Implement AWS CloudWatch for monitoring and alerting.
- Continuously optimize your AWS resources for cost efficiency.
- Implement auto-scaling and redundancy for improved reliability.

## Security Considerations

- Follow AWS best practices for security (e.g., IAM, Security Groups, Network ACLs).
- Use AWS Identity and Access Management (IAM) to manage access and permissions.
- Enable encryption for data in transit and at rest.

## Troubleshooting

- Monitor AWS CloudWatch logs for error tracking.
- Utilize AWS support resources and forums for assistance.
- Maintain backup and recovery strategies.

## Resources

- [AWS Migration Hub](https://aws.amazon.com/migration/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Documentation](https://docs.aws.amazon.com/)

 