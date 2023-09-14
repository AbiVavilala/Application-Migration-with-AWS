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
![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/source-env.png)
- The Onprem application has a webserver and database running on server. I will replatform database with AWS Database Migration Service.

   
### Migration Strategy
in this migration, My strategy is to rehost Wordpress application on EC2 instance and Migrate onprem Mysql database to AWS managed Mysql RDS. this is lift and shift strategy.
-  I will rehost the webserver with AWS Application Migration Service. I will migrate Source database using AWS Database Migration service.
-  I will follow AWS Well Architected Framework to improve Operation Excellence, Security, Performance Efficency and Cost Optimization for My application in AWS.

### 2. AWS Account Setup

- Configure AWS Identity and Access Management (IAM) roles and permissions for IAM user needed for this migration.
  
- AWS IAM user has been created with Administrator access permission added. Logged  into AWS using IAM User.
![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/aws-sinign.PNG)


 ### 3. Cloud Formation 

I Uploaded AWS cloud formation template which will create a source environment and target environment as required for this project. 

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/cloudformation1.PNG)

click on next

I entered the details of the stack

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/cloudformation2.PNG)

once stack is submitted the required resource will be created. you can check the output of stack to see the output. both source and target environment is craeted.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/cloudformation3.PNG)
 
###  Target Environment

- The following target Amazon Virtual private cloud (VPC) is deployed during the environment preparation.
![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/target-vpc.png)

The VPC consists of 6 subnets (x2 public, x2 private for webservers and x2 private for database) across two availability zones. nat gateway is deployed in two public subnets in AZs and internet gateway is deployed using this cloud formation template.


### 4. Database Migration
In th
### Set Up Networking

In this workshoop, will migrate Source database to Target database using DMS replication instance. DMS replication instance will need to connect to source database over public internet, while to the target database over private network.

![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/Set-Up-Networking.png)
 
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

 