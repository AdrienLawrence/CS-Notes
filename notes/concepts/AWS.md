# Overview

- AWS (Amazon Web Services) is a cloud platform that provides computing resources over the internet

- AWS is used to scale on-demand, is paid per usage, and has data centers all over the world

- AWS uses cloud computing to outsource your scaling needs to a company with existing infrastructure ready to be used instead of you having to build your own infrastructure, avoiding potential overhead cost for users

- AWS offers 200+ services that fall into 6 main categories: 

1. Processing: 

    EC2 (Elastic Compute Cloud): Virtual Servers
    
    Lambda: Serverless functions

2. Storage: 

    S3 (Simple Storage Service): Object Storage

    EBS (Elastic Block Store): Disk Drives for EC2

3. Databases:

    RDS (Relational Database Service): Managed SQL databases

    DynamoDB: Managed NoSQL databases

4. Networking:

    VPC (Virtual Private Cloud): Your private network in AWS

    Route 53: DNS Service

5. Security

    IAM (Identity and Access Management): User permissions

6. Deployment

    CloudFormation: Infrastructure as code

Example with netflix: 

1. User requests movie -> Route 53 directs to nearest server
2. Application running -> EC2 Instances
3. User Data -> RDS database
4. Video file storage -> S3 buckets
5. User access -> IAM policies
6. Video caching -> CloudFront (CDN)
7. Peak Hour servers -> Auto Scaling
8. Video transcoding -> Lambda

Netflix doesn't own any hardware. AWS rents everything by the usage