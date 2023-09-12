# Application Migration with AWS

This README.md file provides comprehensive documentation for the process of migrating an application to the Amazon Web Services (AWS) cloud platform. I followed the steps outlined below for Migrating Onprem Wordpress Application to AWS.

## Table of Contents

- [Introduction](#introduction)
 - [Migration Steps](#migration-steps)
   - [1. Assessment and Planning](#1-assessment-and-planning)
   - [2. Target Environment](#2-Target-Environment)
   - [3. AWS Account Setup](#3-AWS-Account-Setup)
   - [4. Cloud Formation template](#4-CloudFormation-template)
   - [5. Database Migration](#5-Database-Migration)
   - [6. Server Migration](#6-Server-Migration)
   - [7. Testing and Validation](#6-testing-and-validation)
- [Monitoring and Optimization](#monitoring-and-optimization)
- [Security Considerations](#security-considerations)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)
- [License](#license)

## Introduction

This document outlines the steps and best practices for migrating your application to AWS. AWS provides a range of services and tools that can help you seamlessly migrate your application to the cloud. By following this guide, you will ensure a smooth transition while taking advantage of AWS's scalability, reliability, and cost-effectiveness.


## Migration Steps

### 1. Assessment and Plannin

- Current Environment consist of a three tier e-commerce application; a webserver running ubuntu with apache, PHP, Wordpress/ WooCommerce and a database server running Ubuntu with MySQL version 5.7.
![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/source-env.png)
- The Onprem application has a webserver and database running on server. I will replatform database with AWS Database Migration Service.
-  I will rehost the webserver with AWS Application Migration Service and modernize webserver to containers running on Amazon Elastic Container Service. 
-  I will follow AWS Well Architected Framework to improve Operation Excellence, Security, Performance Efficency and Cost Optimization for My application in AWS.

### 2. Target Environment

The following target Amazon Virtual private cloud (VPC) is deployed during the environment preparation.
![](https://github.com/AbiVavilala/Application-Migration-with-AWS/blob/master/target-vpc.png)

The VPC consists of 6 subnets (x2 public, x2 private for webservers and x2 private for database) across two availability zones.


### 3. AWS Account Setup

- Configure AWS Identity and Access Management (IAM) roles and permissions.
  
  AwS IAM user has been created with Administrator access permission added. Logged  into AWS using IAM User


 ### 4. Cloud Formation 

I downloaded AWS cloud formation template which will create a source environment and target environment. 
 

### 4. Data Migration

- Choose an appropriate data migration strategy (e.g., AWS Database Migration Service, AWS Snowball, etc.).
- Migrate your application data and databases to AWS.
- Verify data integrity and consistency.

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

## License

This project is licensed under the [MIT License](LICENSE).
