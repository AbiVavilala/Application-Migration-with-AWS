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

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/replication-instance.png)

give a name for subnet group and select Target VPC and add Two public subnets for Replication instance to launch so that they can communicate with source server on public network. 

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/replication-instance1.png)

In this Step, we will create DMS replication instance. 

Inside AWS Console, go to Services and Database Migration Service. Click on Replication instances and then on the Create replication instance button.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/replication-instance2.png)


On the Create replication instance screen I configured a new replication instance with values 

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/replication-instance3.png)

I selected the values as needed for my migration.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/replication-instance4.png)


Click on Create instance

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/images/replication-instance5.png)



### 5. Application Deployment

- Set up EC2 instances or choose a serverless approach (e.g., AWS Lambda).
- Install and configure your application and its dependencies.
- Ensure proper scaling and load balancing for high availability.

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

 