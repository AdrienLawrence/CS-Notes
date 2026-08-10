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

# Global Infrastructure and Accounts

- An AWS account is a security, billing, and resource isolation boundary

- AWS Organizations groups accounts and applies governance through organizational units and Service Control Policies

- Use separate accounts for production, development, security/logging, and shared infrastructure when scale and risk justify it

- A region contains multiple Availability Zones. Many resources are regional while some services/configuration are global

- An Availability Zone is one or more isolated data centers connected with low-latency networking

- Deploying across zones improves availability but the application and data tier must both support failover

- Every AWS API request is signed by an identity and evaluated against policies, resource policies, organization controls, permission boundaries, and explicit denies

# IAM

- IAM principals include users, roles, and service identities

- Prefer roles and temporary credentials over long-lived access keys

- A policy contains effects, actions, resources, and optional conditions

    ```JSON
    {
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Action": ["s3:GetObject"],
            "Resource": "arn:aws:s3:::example-private-bucket/users/${aws:userid}/*"
        }]
    }
    ```

- An explicit deny overrides an allow

- Identity policies attach to principals. Resource policies attach to resources such as S3 buckets, KMS keys, queues, and Lambda functions

- Trust policies define who may assume a role

- Apply least privilege, MFA, short sessions, access review, and root-account protection

- IAM Access Analyzer and CloudTrail help review exposure and actual API behavior

- Do not place AWS access keys in code, container images, public environment variables, or CI secrets when workload identity/OIDC roles are available

# VPC Networking

- A VPC is a regional virtual network with one or more IPv4/IPv6 ranges

- A subnet belongs to one Availability Zone

- Route tables decide where traffic is sent

- An Internet Gateway enables public internet routing for public addresses

- A NAT Gateway allows private-subnet outbound IPv4 access without allowing unsolicited inbound connections

- Security Groups are stateful resource firewalls. Network ACLs are stateless subnet controls

- VPC endpoints connect privately to supported AWS services without public routing

- Application Load Balancer handles HTTP routing. Network Load Balancer handles high-performance Layer 4 traffic

- Route 53 provides DNS, routing policies, domains, and health checks

- CloudFront is a CDN for cached/accelerated content and may use S3, load balancers, or other origins

- AWS WAF filters web requests. Shield provides DDoS protections at different service levels

# Compute

- EC2 gives virtual machines with chosen CPU, memory, disk, network, and operating system

- Auto Scaling Groups maintain desired EC2 capacity and replace unhealthy instances

- User data can bootstrap instances, but immutable images and configuration management improve repeatability

- Lambda runs event-driven functions with configured memory, timeout, runtime, concurrency, and IAM role

- Lambda concurrency can overload databases or downstream APIs. Configure reserved concurrency and connection strategies

- Cold starts occur when a new execution environment initializes. Package size, runtime work, networking, and provisioned concurrency affect latency

- ECS orchestrates containers. Fargate supplies managed serverless container capacity. EKS provides managed Kubernetes control planes

- Elastic Beanstalk and App Runner provide higher-level application platforms for selected workloads

# Storage

- S3 stores objects in buckets and supports versioning, lifecycle rules, replication, encryption, event notifications, and multiple storage classes

- S3 keys are object names, not true filesystem directories

- Block Public Access and least-privilege bucket policies protect accidental public exposure

- Presigned URLs grant temporary scoped object operations without exposing AWS credentials

- Versioning helps recover overwritten/deleted objects. Lifecycle rules transition or expire versions

- EBS provides block volumes for EC2 and snapshots for backup/replication workflows

- EFS provides managed shared NFS file storage

- Instance store is fast ephemeral disk tied to the instance lifecycle

# Databases and Caches

- RDS manages relational engines such as PostgreSQL and MySQL with backups, patching options, monitoring, and high-availability configurations

- Multi-AZ improves availability. Read replicas primarily scale reads and support other topology goals

- Aurora is an AWS relational database compatible with PostgreSQL/MySQL protocols and uses a distributed storage design

- DynamoDB is a managed key-value/document database built around partition keys and optional sort keys

- DynamoDB access patterns should be designed before keys and secondary indexes

- Conditional writes support optimistic concurrency and idempotency

- ElastiCache provides managed Redis or Memcached-compatible caching options

- OpenSearch Service provides managed search/log analytics and related search features

- Redshift is a columnar analytics warehouse

# Messaging and Events

- SQS is a durable message queue. Standard queues prioritize scale and at-least-once delivery; consumers must be idempotent

- FIFO queues provide ordered message groups and deduplication behavior with different throughput constraints

- SNS publishes messages to multiple subscribers

- EventBridge routes events between AWS services, applications, SaaS sources, and schedules

- Step Functions coordinates multi-step state machines with retries, choices, parallel work, and execution history

- Kinesis provides streaming data ingestion and processing

- Dead-letter queues isolate messages which repeatedly fail

- Visibility timeout prevents another SQS consumer from processing a message temporarily; extend it or finish before it expires

# Security Services

- KMS manages encryption keys and key policies

- Secrets Manager stores and rotates secrets. Systems Manager Parameter Store stores hierarchical configuration and optional protected values

- CloudTrail records AWS API activity

- GuardDuty detects suspicious behavior from AWS telemetry

- Security Hub aggregates security findings and standards

- Inspector scans supported workloads and images for vulnerabilities

- Macie discovers sensitive data patterns in S3

- Use defense in depth: IAM, network control, encryption, secret management, logging, patching, backup, and monitoring

# Observability and Operations

- CloudWatch collects metrics, logs, alarms, dashboards, and events

- X-Ray and OpenTelemetry integrations provide distributed tracing

- CloudTrail answers who called which AWS API and from where

- AWS Config records resource configuration history and evaluates rules

- Systems Manager supports inventory, patching, remote commands, automation, and Session Manager access without opening SSH ports

- Tag resources consistently for owner, application, environment, confidentiality, and cost

- Health checks should distinguish process liveness, traffic readiness, and downstream degradation

# Infrastructure and Deployment

- CloudFormation declares AWS resources in templates

- AWS CDK defines CloudFormation using programming languages and constructs

- Terraform/OpenTofu can manage AWS along with other providers

- A deployment pipeline should build an immutable artifact once, test it, and promote it through environments

- Deployment patterns include rolling, blue-green, canary, and feature flags

- CodeBuild builds/tests, CodeDeploy coordinates deployments, and CodePipeline connects stages. GitHub Actions can also assume AWS roles through OIDC

- Database migrations require a separate compatibility and rollback plan from application rollback

# Reliability and Cost

- The AWS Well-Architected pillars include operational excellence, security, reliability, performance efficiency, cost optimization, and sustainability

- Design around failure domains and service quotas

- Use retries with exponential backoff and jitter for transient AWS errors, while keeping mutations idempotent

- Set timeouts and alarms, test failover, and restore backups

- Cost Explorer, Budgets, Cost and Usage Reports, and tagging provide cost visibility

- Savings Plans/Reserved capacity fit predictable compute. Spot capacity fits interruptible workloads

- Data transfer, NAT processing, logs, idle databases, overprovisioned compute, and retained snapshots can become major costs

- Check current regional pricing and quotas before architecture decisions because they change

# Common Architecture

- Static web application:

    Route 53 -> CloudFront -> S3

- Highly available web API:

    Route 53 -> WAF -> Application Load Balancer -> ECS/EC2 across zones -> RDS Multi-AZ

- Serverless API:

    API Gateway -> Lambda -> DynamoDB/RDS -> CloudWatch

- Event processing:

    Producer -> EventBridge/SNS/SQS -> Lambda/ECS consumer -> Dead-letter queue

- Private workloads should use private subnets/endpoints where practical, while public entry points are narrowly controlled

# Quick Reference

- Virtual machine -> EC2

- Event function -> Lambda

- Managed containers -> ECS/Fargate

- Kubernetes -> EKS

- Object storage/CDN -> S3 and CloudFront

- Block disk -> EBS

- Shared filesystem -> EFS

- Relational database -> RDS/Aurora

- Key-value/document database -> DynamoDB

- Cache -> ElastiCache

- Queue -> SQS

- Pub/sub -> SNS

- Event routing -> EventBridge

- Workflow state machine -> Step Functions

- Identity/permissions -> IAM

- Encryption keys -> KMS

- Secrets -> Secrets Manager

- Logs/metrics/alarms -> CloudWatch

- API audit -> CloudTrail

- Infrastructure as code -> CloudFormation/CDK
