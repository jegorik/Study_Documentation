# AWS Solutions Architect Associate (SAA-C03) Exam Notes

## Table of Contents

1. [Quick Review Checklist](#quick-review-checklist)

2. [About the Exam](#1-about-the-exam)

3. [Study Strategy & Resources](#2-study-strategy--resources)

4. [AWS Core Concepts](#3-aws-core-concepts)

5. [Compute Services](#4-compute-services)

6. [Storage Services](#5-storage-services)

7. [Networking & Content Delivery](#6-networking--content-delivery)

8. [Database Services](#7-database-services)

9. [Security, Identity & Compliance](#8-security-identity--compliance)

10. [Monitoring & Management](#9-monitoring--management)

11. [Application Integration](#10-application-integration)

12. [Migration & Transfer](#11-migration--transfer)

13. [Analytics & Machine Learning](#12-analytics--machine-learning)

14. [Exam Tips & Scenarios](#13-exam-tips--scenarios)

15. [Practical Scenario Walkthroughs](#13-exam-tips--scenarios)

16. [Common Misconceptions](#13-exam-tips--scenarios)

17. [Service Comparison & Decision Trees](#13-exam-tips--scenarios)

18. [Mnemonic Devices](#mnemonic-devices)

19. [Glossary](#14-glossary)

## AWS Service Quick Index

Jump directly to specific AWS service documentation:

### Compute

- [EC2](#amazon-ec2) - Elastic Compute Cloud
- [Lambda](#aws-lambda-expanded) - Serverless Functions
- [ECS/ECR/EKS](#container-services) - Container Services
- [Fargate](#container-services) - Serverless Container Compute
- [Auto Scaling](#auto-scaling-groups) - Dynamic Resource Scaling
- [Elastic Beanstalk](#elastic-beanstalk) - Application Deployment Service

### Storage

- [S3](#amazon-s3-expanded) - Simple Storage Service
- [EBS](#amazon-ebs) - Elastic Block Store
- [EFS](#amazon-efs) - Elastic File System
- [Storage Gateway](#storage-gateway) - Hybrid Storage
- [FSx](#fsx) - Managed File Systems

### Networking

- [VPC](#amazon-vpc-expanded) - Virtual Private Cloud
- [Route 53](#route-53) - DNS Service
- [CloudFront](#cloudfront) - CDN
- [Direct Connect](#direct-connect) - Dedicated Network Connection
- [API Gateway](#6-networking--content-delivery) - API Management
- [Global Accelerator](#global-accelerator) - Network Performance
- [Elastic Load Balancing](#elastic-load-balancing-elb) - Load Distribution Service
- [PrivateLink](#6-networking--content-delivery) - Private Endpoint Service
- [Transit Gateway](#6-networking--content-delivery) - Network Transit Hub
- [Gateway Load Balancer](#6-networking--content-delivery) - Appliance Load Balancer

### Database

- [RDS](#amazon-rds) - Relational Database Service
- [RDS Proxy](#specialized-database-services) - Database Connection Pooling
- [RDS Custom](#specialized-database-services) - Customizable Database
- [DynamoDB](#dynamodb-expanded) - NoSQL Database
- [DAX](#dynamodb-expanded) - DynamoDB Accelerator
- [Aurora](#aurora) - Managed MySQL/PostgreSQL
- [ElastiCache](#specialized-database-services) - In-Memory Cache
- [Neptune](#specialized-database-services) - Graph Database
- [DocumentDB](#specialized-database-services) - Document Database
- [Redshift](#redshift) - Data Warehouse
- [QLDB](#specialized-database-services) - Ledger Database
- [Timestream](#specialized-database-services) - Time Series Database

### Security & Identity

- [IAM](#iam-identity-and-access-management) - Identity & Access Management
- [Cognito](#cognito) - User Identity & Data Sync
- [KMS](#kms-key-management-service) - Key Management
- [CloudHSM](#cloudhsm) - Hardware Security Module
- [Secrets Manager](#8-security-identity--compliance) - Secrets Management
- [ACM](#acm-aws-certificate-manager) - Certificate Manager
- [WAF](#8-security-identity--compliance) - Web Application Firewall
- [Shield](#8-security-identity--compliance) - DDoS Protection
- [GuardDuty](#8-security-identity--compliance) - Threat Detection
- [Macie](#8-security-identity--compliance) - Data Security & Privacy

### Management & Monitoring

- [CloudWatch](#cloudwatch) - Monitoring & Observability
- [CloudTrail](#cloudtrail) - API Activity Tracking
- [AWS Config](#aws-config) - Resource Configuration
- [Systems Manager](#9-monitoring--management) - Operations Management
- [Trusted Advisor](#9-monitoring--management) - Optimization Recommendations
- [Control Tower](#control-tower) - Account Management
- [X-Ray](#x-ray) - Application Tracing
- [Organizations](#aws-organizations--scps) - Multi-account Management

### Integration

- [SQS](#amazon-sqs-simple-queue-service) - Simple Queue Service
- [SNS](#amazon-sns-simple-notification-service) - Simple Notification Service
- [EventBridge](#amazon-eventbridge-cloudwatch-events) - Event Bus
- [Step Functions](#step-functions) - Workflow Orchestration
- [MQ](#10-application-integration) - Managed Message Broker
- [AppFlow](#10-application-integration) - SaaS Integration Service

### Migration

- [DMS](#aws-dms-database-migration-service) - Database Migration Service
- [Snowball](#aws-snowball) - Large-scale Data Transfer
- [Transfer Family](#aws-transfer-family) - Managed File Transfer
- [DataSync](#11-migration--transfer) - Data Transfer Service

### Analytics

- [Athena](#amazon-athena) - Serverless Query Service
- [Glue](#aws-glue) - ETL Service
- [Kinesis](#amazon-kinesis) - Real-time Data Streaming
- [Kinesis Data Firehose](#12-analytics--machine-learning) - Data Delivery Stream
- [Kinesis Data Analytics](#12-analytics--machine-learning) - Real-time Analytics
- [EMR](#amazon-emr) - Managed Hadoop Framework
- [QuickSight](#amazon-quicksight) - Business Intelligence
- [SageMaker](#amazon-sagemaker) - Machine Learning Platform

---

## Quick Review Checklist

> Use this section for last-minute review before your exam. It contains the most critical concepts to remember.

### Key Services by Domain

- **Design Resilient Architectures**: S3, EBS, EFS, EC2 Auto Scaling, Multi-AZ, Read Replicas
- **Design High-Performance Architectures**: ElastiCache, RDS, Aurora, DynamoDB, Route 53
- **Design Secure Applications**: IAM, Security Groups, NACLs, KMS, CloudHSM, Shield, WAF
- **Design Cost-Optimized Architectures**: Reserved Instances, Spot Instances, S3 Storage Classes

### Critical Service Limits

- **S3**: 5TB max object size, unlimited storage
- **Lambda**: 15-minute max execution, 10GB memory
- **EC2**: 40Gbps network bandwidth (max), varies by instance type
- **RDS**: 64TB max storage (MySQL, PostgreSQL), 16TB (SQL Server)
- **DynamoDB**: No practical limits for tables or items, 400KB max item size

### Key AWS Well-Architected Framework Pillars

- **Operational Excellence**: Use Infrastructure as Code, automate deployments
- **Security**: Apply principle of least privilege, encrypt data in transit and at rest
- **Reliability**: Test recovery procedures, auto-scale horizontally to increase availability
- **Performance Efficiency**: Use serverless architectures, experiment with new technologies
- **Cost Optimization**: Match supply with demand, use managed services to reduce TCO

### Most Commonly Tested Scenarios

- High availability database deployment (Multi-AZ RDS)
- Building a serverless web application (API Gateway, Lambda, DynamoDB)
- Setting up a fault-tolerant web architecture (ELB, Auto Scaling, Multi-AZ)
- Securing a VPC with multiple public and private subnets
- Optimizing storage costs with lifecycle policies and storage classes
- Designing hybrid architectures with Direct Connect and VPN connections

---

## 1. About the Exam

- Format, duration, passing score, domains breakdown.
- [Official AWS Exam Guide](https://aws.amazon.com/certification/certified-solutions-architect-associate/)

## 2. Study Strategy & Resources

- Adrian Cantrill’s [SAA-C03 Course](https://learn.cantrill.io/p/aws-certified-solutions-architect-associate-saa-c03)
- [Tutorials Dojo AWS Cheat Sheets](https://tutorialsdojo.com/aws-cheat-sheets/)
- [Alberto Lozano's Course Notes](https://github.com/alozano-77/AWS-SAA-C02-Course#16-elastic-cloud-compute-ec2)
- AWS Whitepapers, FAQs, Hands-on Labs

## 3. AWS Core Concepts

### Shared Responsibility Model

- AWS is responsible for security **of** the cloud (hardware, software, networking, facilities).
- Customer is responsible for security **in** the cloud (data, identity, applications, OS, network config).
- **Exam Tip:** Know which responsibilities belong to AWS vs. the customer for different services (e.g., EC2 vs S3).

### AWS Global Infrastructure

- **Regions:** Isolated geographic areas (e.g., us-east-1).
- **Availability Zones (AZs):** Isolated locations within a region.
- **Edge Locations:** Used for CDN (CloudFront), DNS (Route 53), and caching.
- **Exam Tip:** Some services are global (IAM, Route 53), some are regional (EC2, S3), some are AZ-scoped (EBS).

### Pricing Models

- **On-Demand:** Pay for compute/storage by the hour/second with no long-term commitment.
- **Reserved:** Commit to 1 or 3 years for significant discount.
- **Spot:** Bid for unused capacity at steep discounts (can be interrupted).
- **Savings Plans:** Flexible pricing for compute usage.

---

## 4. Compute Services

### Amazon EC2

[Back to Service Index](#aws-service-quick-index)

- **What:** Virtual machines (instances) in the cloud. Supports multiple OS, sizes, and purchasing options.
- **Key Concepts:**
  - Instance types (general, compute, memory, storage optimized)
  - AMI (Amazon Machine Image): Template for launching instances
  - Security Groups: Virtual firewalls for instances
  - EBS: Persistent block storage
  - Lifecycle: Running, stopped, terminated
- **Exam Tips:**
  - Only EBS is billed when instance is stopped
  - Security Groups are stateful, NACLs are stateless
  - Use IAM roles for EC2 to avoid hardcoding credentials

### AWS Lambda (Expanded)

[Back to Service Index](#aws-service-quick-index)

- **What:** Serverless compute service that lets you run code without provisioning or managing servers.
- **Key Concepts:**
  - Supported runtimes: Node.js, Python, Java, Go, Ruby, .NET
  - Event-driven execution model
  - Integration with other AWS services
  - Cold starts vs. warm starts
  - Concurrency limits and quotas
- **Use Cases:**
  - Serverless web applications and microservices
  - Real-time file processing and data transformations
  - Backend API services
  - Scheduled tasks and ETL jobs
- **Exam Tips:**
  - Maximum execution time: 15 minutes
  - Memory allocation range: 128MB to 10GB (memory also determines CPU allocation)
  - Temporary storage: 512MB in /tmp directory
  - Deployment package size: 50MB (zipped), 250MB (unzipped)
  - Environment variables limit: 4KB
  - Concurrency per region: 1000 (soft limit, can be increased)
  - Supports VPC integration, requires ENI (Elastic Network Interface)
  
### Container Services

[Back to Service Index](#aws-service-quick-index)

- **Amazon ECS (Elastic Container Service)**
  - Fully managed container orchestration service
  - Supports Docker containers
  - Can use Fargate (serverless) or EC2 launch types
  - Integrates with Load Balancers, IAM, CloudWatch, VPC

- **Amazon ECR (Elastic Container Registry)**
  - Fully managed Docker container registry
  - Integrates with ECS and EKS
  - Supports vulnerability scanning
  - Private repositories with fine-grained access control

- **Amazon EKS (Elastic Kubernetes Service)**
  - Managed Kubernetes service
  - Runs upstream Kubernetes
  - Integrates with AWS services (IAM, VPC, etc.)
  - Supports Fargate for serverless Kubernetes

- **Exam Tips:**
  - ECS vs EKS: ECS is AWS proprietary, EKS is managed Kubernetes
  - Fargate removes the need to manage underlying EC2 instances
  - ECS Task Roles vs. Instance Roles: Task roles provide permissions to specific containers

### Fargate

[Back to Service Index](#aws-service-quick-index)

- **What:** Serverless compute engine for containers that works with both Amazon ECS and Amazon EKS.
- **Key Concepts:**
  - No EC2 instances to manage - AWS handles the underlying infrastructure
  - Pay only for the resources allocated to containers while they're running
  - Tasks are injected into your VPC with their own Elastic Network Interface (ENI)
  - 20 GB of ephemeral storage included with each task
- **Exam Tips:**
  - Use for small/burst workloads or batch/periodic processing to optimize costs
  - Ideal when you want to focus on application design rather than infrastructure management
  - Best choice for overhead-conscious large workloads, while EC2 mode is better for price-conscious large workloads
  - Integrates with IAM for task-level security through Task Roles

### Auto Scaling Groups

[Back to Service Index](#aws-service-quick-index)

- **What:** Automatically adjusts the number of EC2 instances based on demand.
- **Key Concepts:**
  - Launch Templates/Configurations define the EC2 instance settings
  - Scaling Policies determine when to scale out (add instances) or in (remove instances)
  - Cooldown periods prevent rapid scaling fluctuations
  - Integration with ELB ensures traffic is properly distributed

- **Scaling Types:**
  - Simple scaling: based on a single metric (e.g., CPU utilization)
  - Step scaling: variable adjustment based on metric magnitude
  - Target tracking: maintains a specific metric value (e.g., 70% CPU utilization)
  - Scheduled scaling: predetermined scaling based on time
  - Predictive scaling: ML-powered scaling based on patterns and forecasts

- **Exam Tips:**
  - Multi-AZ deployment increases availability
  - Warm-up time allows instances to initialize before receiving traffic
  - Health checks determine instance health and replacement
  - Instance protection prevents specific instances from termination

### Elastic Beanstalk

[Back to Service Index](#aws-service-quick-index)

- **What:** A fully managed service that makes it easy to deploy and run applications in multiple languages without worrying about the underlying infrastructure.

- **Key Concepts:**
  - Platform as a Service (PaaS) - handles deployment, capacity provisioning, load balancing, auto-scaling, and application health monitoring
  - Supports multiple languages and platforms: Java, .NET, PHP, Node.js, Python, Ruby, Go, and Docker
  - Includes web server environments (Apache, Nginx, IIS) and application server environments
  - Maintains multiple versions of your application for easy rollbacks
  - Integrates with CloudWatch for monitoring and X-Ray for tracing

- **Exam Tips:**
  - Use for quickly deploying web applications without managing infrastructure
  - Elastic Beanstalk is free (you only pay for the underlying AWS resources)
  - Two environment types: Web Server Environment and Worker Environment
  - Supports deployment of application versions with zero downtime using rolling updates
  - You can customize the environment using configuration files (.ebextensions)
  - Perfect for developers who want to focus on writing code rather than managing infrastructure

---

## 5. Storage Services

### Amazon S3

- **What:** Object storage for files, backups, static websites, and more.
- **Key Concepts:**
  - Buckets (globally unique names)
  - Objects (key-value pairs, up to 5TB)
  - Storage classes: Standard, IA, One Zone-IA, Glacier, Deep Archive, Intelligent-Tiering
  - Versioning, lifecycle policies, replication
  - Encryption: SSE-S3, SSE-KMS, SSE-C, client-side
- **Exam Tips:**
  - S3 is private by default; use bucket policies for access
  - S3 cannot be mounted as a filesystem
  - Versioning cannot be disabled (only suspended)

### Amazon S3 (Expanded)

[Back to Service Index](#aws-service-quick-index)

- **What:** Scalable object storage service designed for 99.999999999% (11 9's) of durability.
- **Core Concepts:**
  - **Buckets**: Container for objects, globally unique name
  - **Objects**: Files and metadata stored in buckets (0 bytes to 5TB)
  - **Keys**: Unique identifier for an object within a bucket
  - **Regions**: Physical location where buckets are stored
  - **S3 URL Formats**:
    - Virtual-hosted style: `https://bucket-name.s3.region.amazonaws.com/key-name`
    - Path style: `https://s3.region.amazonaws.com/bucket-name/key-name`
  
- **Storage Classes**:
  - **S3 Standard**: Default, high availability, durability, and performance
  - **S3 Intelligent-Tiering**: Automatic cost optimization based on access patterns
  - **S3 Standard-IA** (Infrequent Access): Lower cost for less frequently accessed data
  - **S3 One Zone-IA**: Lower cost by storing in a single AZ (99.5% availability)
  - **S3 Glacier**: Low-cost archival storage with retrieval times from minutes to hours
  - **S3 Glacier Deep Archive**: Lowest-cost storage for long-term retention (retrieval time: hours)
  
- **Data Management Features**:
  - **Lifecycle Policies**: Automate transitions between storage classes and expiration
  - **Versioning**: Store multiple versions of objects for protection from overwrites and deletions
  - **Replication**:
    - **Cross-Region Replication (CRR)**: Replicate objects across regions
    - **Same-Region Replication (SRR)**: Replicate objects within a region
  - **Object Lock**: Prevent objects from being deleted or overwritten for a fixed time or indefinitely
  - **Inventory**: Report on objects and their metadata for auditing and analysis
  
- **Access Management**:
  - **Bucket Policies**: JSON policies attached to buckets for access control
  - **Access Control Lists (ACLs)**: Legacy method for controlling access
  - **Presigned URLs**: Temporary access to objects
  - **Access Points**: Named network endpoints with dedicated access policies
  - **Block Public Access**: Settings to prevent public access at the account or bucket level
  
- **Security Features**:
  - **Encryption**:
    - **SSE-S3**: Server-side encryption with Amazon S3-managed keys
    - **SSE-KMS**: Server-side encryption with AWS KMS-managed keys
    - **SSE-C**: Server-side encryption with customer-provided keys
    - **Client-side encryption**: Encrypt data before uploading
  - **VPC Endpoints**: Private access to S3 without traversing the internet
  - **S3 Object Lambda**: Transform data as it's retrieved from S3
  
- **Performance Optimization**:
  - **Multipart Upload**: Parallel uploads for large objects
  - **Transfer Acceleration**: Fast transfers over long distances using CloudFront
  - **S3 Select**: Retrieve only needed data from objects using SQL
  - **Partitioning Strategy**: Distribute workload by adding prefixes to keys
  
- **Pricing Components**:
  - Storage by GB per month (varies by storage class)
  - Requests (PUT, GET, DELETE, etc.)
  - Data transfer (out of AWS, between regions)
  - Management features (inventory, analytics)
  
- **Use Cases**:
  - Static website hosting
  - Backup and recovery
  - Data lakes and big data analytics
  - Media storage and distribution
  - Software delivery
  
- **Limitations**:
  - Maximum object size: 5TB
  - Maximum bucket size: Unlimited
  - Maximum bucket name length: 3-63 characters
  - Default bucket limit: 100 per account (can be increased)
  
- **Exam Tips:**
  - Understand the differences between storage classes and when to use each
  - Know the security features and encryption options
  - Remember that bucket names are globally unique and must follow DNS naming conventions
  - Lifecycle policies are key to cost optimization
  - Object locks and versioning are important for compliance and data protection

### Amazon EBS

[Back to Service Index](#aws-service-quick-index)

- **What:** Block storage for EC2 instances.
- **Key Concepts:**
  - Persistent, can be detached/reattached
  - Snapshots for backup
- **Exam Tips:**
  - Only EBS is billed when EC2 is stopped

### Amazon EFS

[Back to Service Index](#aws-service-quick-index)

- **What:** Managed NFS file system for Linux EC2 instances.
- **Key Concepts:**
  - Shared, scalable, pay-per-use

### Storage Gateway

[Back to Service Index](#aws-service-quick-index)

- **What:** Hybrid storage service that enables on-premises applications to use AWS cloud storage.

- **Types:**
  - **File Gateway:** SMB/NFS interface to store files as objects in S3
  - **Volume Gateway:** iSCSI block storage interface with volumes backed by S3
    - **Stored Volumes:** Complete data stored on-premises with async backup to S3
    - **Cached Volumes:** Primary data in S3 with frequently accessed data cached on-premises
  - **Tape Gateway:** VTL interface to store virtual tapes in S3 and Glacier

- **Exam Tips:**
  - Use for cloud migration, backup and archive, disaster recovery, hybrid storage
  - Provides local caching for low-latency access to frequently accessed data
  - Supports encryption, snapshot scheduling, bandwidth throttling

### FSx

[Back to Service Index](#aws-service-quick-index)

- **What:** A family of fully managed file storage services that provides industry-standard file systems.

- **FSx for Windows File Server:**
  - **What:** Native Windows file system service that provides SMB-based file storage.
  - **Key Concepts:**
    - Fully managed Windows file servers built on Windows Server
    - Integrates with Active Directory (AWS Managed AD or self-hosted)
    - Supports SMB protocol and NTFS file systems
    - Accessible from Windows, Linux, and macOS
    - Data is replicated within an Availability Zone and can be configured for Multi-AZ
  - **Exam Tips:**
    - Use for Windows-based applications requiring shared file storage
    - Ideal for migrating Windows applications to AWS
    - Supports high availability with automatic failover in Multi-AZ deployments

- **FSx for Lustre:**
  - **What:** High-performance file system optimized for compute-intensive workloads.
  - **Key Concepts:**
    - Designed for high-performance computing (HPC), machine learning, and media processing
    - Provides sub-millisecond latencies and throughput up to hundreds of GB/s
    - Integrates with S3, allowing you to process data directly from S3
    - Available in Scratch (temporary, highly performant) or Persistent (longer-term) deployment types
  - **Exam Tips:**
    - Choose when you need high-performance shared storage
    - POSIX-compliant file system that works with Linux
    - Data can be automatically imported/exported to/from S3
    - Not suitable for general-purpose file sharing like Windows applications

- **Exam Decision Guide:**
  - Windows workloads requiring SMB → FSx for Windows
  - High-performance computing, big data, machine learning → FSx for Lustre
  - Linux workloads requiring NFS → EFS

---

## 6. Networking & Content Delivery

### Amazon VPC

- **What:** Virtual network for AWS resources.
- **Key Concepts:**
  - Subnets (public/private), route tables, IGW, NAT, NACL, Security Groups
  - Default VPC is created per region
  - VPC peering, endpoints, Transit Gateway
- **Exam Tips:**
  - Security Groups are stateful, NACLs are stateless
  - Default VPC CIDR: 172.31.0.0/16

### Amazon VPC (Expanded)

[Back to Service Index](#aws-service-quick-index)

- **What:** Virtual Private Cloud (VPC) is a logically isolated section of the AWS cloud where you can launch AWS resources in a virtual network that you define.
- **Core Components:**
  - **Subnets**: Subdivisions of a VPC CIDR block where you place resources
    - **Public Subnets**: Have a route to the Internet Gateway
    - **Private Subnets**: No direct route to the internet
  - **Route Tables**: Control the traffic flow between subnets and gateways
  - **Internet Gateway (IGW)**: Allows communication between your VPC and the internet
  - **NAT Gateway/NAT Instance**: Enables private subnet resources to access the internet
  - **Security Groups**: Virtual firewalls for EC2 instances (stateful)
  - **Network ACLs**: Firewall for subnets (stateless)
  - **VPC Endpoints**: Private connections to AWS services without traversing the internet
    - **Gateway Endpoints**: For S3 and DynamoDB
    - **Interface Endpoints (Powered by PrivateLink)**: For other AWS services
  
- **Advanced Features**:
  - **VPC Peering**: Connect two VPCs to route traffic between them
  - **Transit Gateway**: Hub that connects VPCs and on-premises networks
  - **VPN Connections**:
    - **Site-to-Site VPN**: Connect on-premises network to VPC
    - **Client VPN**: Secure remote access for users
  - **Direct Connect**: Dedicated private connection from on-premises to AWS
  - **Egress-only Internet Gateway**: Allow outbound IPv6 traffic but prevent inbound
  - **Flow Logs**: Capture network traffic information for monitoring
  
- **Default VPC**:
  - Created in each region automatically
  - CIDR block: 172.31.0.0/16
  - Default subnet in each AZ
  - Internet gateway attached
  - Default security group, network ACL, and route table
  
- **IP Addressing**:
  - IPv4: Primary addressing system
  - IPv6: Optional, all addresses are public
  - Elastic IPs: Static public IPv4 addresses
  
- **Best Practices**:
  - Plan your CIDR block carefully to allow for future growth
  - Use private subnets for backend resources
  - Use security groups as your primary firewall mechanism
  - Use NACLs for an additional layer of security
  - Use flow logs for network monitoring and troubleshooting
  
- **Limitations**:
  - Maximum of 5 VPCs per region (default limit, can be increased)
  - Maximum of 200 subnets per VPC
  - CIDR block size between /16 (65,536 IPs) and /28 (16 IPs)
  
- **Exam Tips:**
  - Remember the differences between stateful security groups and stateless NACLs
  - Understand the routing principles and how different components interact
  - Know which services use gateway endpoints vs. interface endpoints
  - Transit Gateway simplifies complex networking with many VPCs
  - Understand that VPC peering doesn't support transitive routing

### Route 53

[Back to Service Index](#aws-service-quick-index)

- **What:** Managed DNS service.
- **Key Concepts:**
  - Hosted zones, record types (A, AAAA, CNAME, MX, TXT, NS)
  - Routing policies: Simple, weighted, latency, failover, geolocation
- **Exam Tips:**
  - S3 static website hosting requires Route 53 for custom domains

### CloudFront

[Back to Service Index](#aws-service-quick-index)

- **What:** Content Delivery Network (CDN) for caching and accelerating content.
- **Key Concepts:**
  - Edge locations, distributions, origins, behaviors
  - Supports static and dynamic content
  - Integrates with S3, EC2, ELB, Route 53
- **Exam Tips:**
  - Use CloudFront to reduce latency and offload traffic from origin
  - Can restrict access to S3 using signed URLs/cookies

### Direct Connect

[Back to Service Index](#aws-service-quick-index)

- **What:** Dedicated network connection from on-premises to AWS.

- **Key Concepts:**
  - Dedicated connection: 1Gbps, 10Gbps, or 100Gbps
  - Hosted connection: Various speeds through AWS partners
  - Virtual interfaces (VIFs): Private, public, or transit
  - Direct Connect gateways: Connect to multiple regions

- **Exam Tips:**
  - More reliable and consistent connection than VPN
  - Lower latency and higher bandwidth
  - Does not provide automatic failover to the internet
  - For HA, establish multiple Direct Connect connections
  - Can be combined with VPN for encrypted communication

### API Gateway

[Back to Service Index](#aws-service-quick-index)

- **What:** Fully managed service for creating, publishing, maintaining, monitoring, and securing APIs.

- **Key Features:**
  - RESTful and WebSocket API support
  - Versioning, staging, and canary releases
  - API keys and usage plans
  - Request/response transformations
  - Caching to improve performance

- **Exam Tips:**
  - Integrates with Lambda, HTTP endpoints, and other AWS services
  - Provides throttling, monitoring, and security features
  - Edge-optimized endpoint for global distribution via CloudFront
  - Regional endpoint for lower latency within the same region
  - Private endpoint for APIs accessible only within VPCs

### Global Accelerator

[Back to Service Index](#aws-service-quick-index)

- **What:** Service that improves availability and performance of applications with global users.

- **Key Features:**
  - Uses Anycast IP addresses to route traffic to closest edge location
  - Traffic remains on AWS global network for improved performance
  - Automatic failover across regions
  - Supports TCP and UDP protocols

- **Exam Tips:**
  - CloudFront vs. Global Accelerator: CloudFront caches content for HTTP/S, GA optimizes TCP/UDP
  - Improves latency and availability for non-HTTP applications
  - Use for gaming, IoT, voice, and media streaming applications
  - Provides static IP addresses that don't change

### PrivateLink

[Back to Service Index](#aws-service-quick-index)

- **What:** Securely access services over the AWS network via private endpoints.
- **Key Concepts:**
  - Interface VPC endpoints: Elastic network interfaces with private IPs
  - Service providers: AWS services, AWS Marketplace, custom services
  - Endpoint policies: Control access to services
- **Exam Tips:**
  - Use PrivateLink to access services securely without exposing traffic to the public internet
  - Integrates with IAM for fine-grained access control

### Transit Gateway

[Back to Service Index](#aws-service-quick-index)

- **What:** Network transit hub that interconnects VPCs and on-premises networks.
- **Key Concepts:**
  - Attachments: Connect VPCs, VPNs, Direct Connect
  - Route tables: Control traffic between attachments
  - Multicast support: Efficiently route traffic to multiple destinations
- **Exam Tips:**
  - Simplifies network architecture by consolidating connections
  - Reduces the number of peering connections needed
  - Use for complex networks with multiple VPCs and on-premises connections

### Gateway Load Balancer

[Back to Service Index](#aws-service-quick-index)

- **What:** Distributes traffic to multiple targets, such as EC2 instances, in one or more Availability Zones.
- **Key Concepts:**
  - Operates at the connection level (TCP/UDP)
  - Integrates with AWS services (EC2, ECS, EKS)
  - Health checks to ensure traffic is only sent to healthy targets
- **Exam Tips:**
  - Use Gateway Load Balancer to deploy, scale, and manage virtual appliances
  - Ideal for third-party security appliances, such as firewalls and intrusion detection systems

---

## 7. Database Services

### Amazon RDS

[Back to Service Index](#aws-service-quick-index)

- **What:** Managed relational database service (supports MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora).
- **Key Concepts:**
  - Automated backups, Multi-AZ deployments, Read Replicas
  - Storage scaling, maintenance windows
- **Exam Tips:**
  - Multi-AZ for high availability, Read Replicas for scaling reads
  - Automated backups are enabled by default

### DynamoDB

- **What:** Managed NoSQL database (key-value and document store).
- **Key Concepts:**
  - Tables, items, attributes, primary key (partition/sort key)
  - On-demand and provisioned capacity, DAX (caching), Streams
- **Exam Tips:**
  - Single-digit millisecond latency at any scale
  - Use DAX for caching, Streams for event-driven architectures

### DynamoDB (Expanded)

[Back to Service Index](#aws-service-quick-index)

- **What:** Fully managed, serverless, NoSQL database service that provides fast and predictable performance with seamless scalability.
- **Core Components:**
  - **Tables**: Collections of items (similar to rows in relational databases)
  - **Items**: Groups of attributes (similar to records)
  - **Attributes**: Fundamental data elements (similar to fields)
  - **Primary Key**: Uniquely identifies each item in a table
    - **Partition Key**: Determines the partition where the item is stored
    - **Sort Key** (optional): Enables sorting items with the same partition key
  
- **Advanced Features**:
  - **Secondary Indexes**:
    - **Global Secondary Index (GSI)**: Index with different partition and sort keys
    - **Local Secondary Index (LSI)**: Index with same partition key but different sort key
  - **Streams**: Captures item-level modifications in real-time
  - **Global Tables**: Multi-region, multi-master replication
  - **Transactions**: Atomic operations across multiple items/tables
  - **TTL (Time to Live)**: Automatically delete items after a specified timestamp
  - **DAX (DynamoDB Accelerator)**: In-memory cache for DynamoDB tables
  
- **Capacity Modes**:
  - **Provisioned Capacity**: Specify read/write capacity units
    - Read Capacity Units (RCUs): 1 RCU = 1 strongly consistent read or 2 eventually consistent reads per second for items up to 4KB
    - Write Capacity Units (WCUs): 1 WCU = 1 write per second for items up to 1KB
  - **On-Demand Capacity**: Pay-per-request pricing, no capacity planning needed
  
- **Consistency Models**:
  - **Eventually Consistent Reads**: Default, less expensive, might not reflect most recent write
  - **Strongly Consistent Reads**: Always reflects most recent write, uses more capacity
  
- **Performance Optimization**:
  - Effective partition key design to avoid hot partitions
  - Use sparse indexes to minimize storage and RCU/WCU consumption
  - Batch operations for efficient reads/writes
  - Consider compression for large attributes
  
- **Cost Optimization**:
  - Choose appropriate capacity mode
  - Use TTL to automatically remove unnecessary data
  - Monitor and adjust capacity settings
  - Use Auto Scaling to match capacity with demand
  
- **Limitations**:
  - Maximum item size: 400KB
  - Maximum partition key size: 2KB
  - Maximum sort key size: 1KB
  
- **Exam Tips:**
  - DynamoDB is ideal for applications that need consistent, single-digit millisecond latency at any scale
  - Understand when to use different consistency models and capacity modes
  - Know how to design effective partition keys to distribute data evenly
  - Remember that DynamoDB Streams can trigger Lambda functions for event-driven architectures
  - DAX provides microsecond response times for read-heavy workloads

### Aurora

[Back to Service Index](#aws-service-quick-index)

- **What:** AWS's high-performance, MySQL- and PostgreSQL-compatible relational database.
- **Key Concepts:**
  - Up to 15 read replicas, automatic failover, storage auto-scaling

### Redshift

[Back to Service Index](#aws-service-quick-index)

- **What:** A petabyte-scale data warehouse service designed for OLAP (Online Analytical Processing) workloads.

- **Key Concepts:**
  - Column-based database optimized for analytics and reporting
  - Leader node (handles query planning) and compute nodes (execute queries)
  - Single-AZ deployment within a VPC
  - Redshift Spectrum allows direct querying of data in S3
  - Federated query enables querying other databases
  - Enhanced VPC Routing gives more control over network traffic
  - Integrates with AWS tools like QuickSight

- **Exam Tips:**
  - Redshift is NOT multi-AZ by default (unlike RDS)
  - Security: VPC security, IAM permissions, KMS encryption at rest
  - Pay-as-you-use pricing model similar to RDS
  - OLAP (column-based) vs OLTP (row-based) - know the difference
  - SQL-like interface with JDBC/ODBC connectivity
  - Differentiate from other database services by its data warehousing purpose

### Specialized Database Services

- **Amazon ElastiCache**:
  - In-memory caching service that supports Redis and Memcached
  - Improves application performance by reducing database load
  - Redis features: Multi-AZ, replication, backup/restore, transactions
  - Memcached features: Multi-threaded, no persistence, no replication

- **Amazon RDS Proxy**:
  - Fully managed database proxy for RDS
  - Pools and shares database connections to improve scalability
  - Reduces failover time by up to 66%
  - Preserves application connections during failovers
  - Enforces IAM authentication and stores credentials in AWS Secrets Manager

- **RDS Custom**:

[Back to Service Index](#aws-service-quick-index)

- **What:** Managed RDS service that provides database customization access at the operating system and database levels.
- **Key Concepts:**
  - Combines automation of RDS with flexibility of self-managed databases
  - Access to underlying OS and database engine
  - Ability to install custom agents, patches, and drivers
  - Currently supports Oracle and Microsoft SQL Server
- **Exam Tips:**
  - Use when you need both RDS automation and deep customization capabilities
  - Ideal for legacy, custom, and packaged applications that require OS and database-level access
  - Maintains automated backups, point-in-time recovery, and monitoring
  - Requires more operational responsibility than standard RDS

- **Amazon DocumentDB**:
  - MongoDB-compatible document database service
  - Scales storage automatically up to 64TB
  - Up to 15 read replicas with millisecond replication
  - Fully managed with automated backups, patching, and recovery

- **Amazon Neptune**:
  - Fully managed graph database service
  - Supports property graph and RDF (Resource Description Framework)
  - Query languages: Gremlin, SPARQL, openCypher
  - Ideal for knowledge graphs, fraud detection, recommendation engines

- **Amazon QLDB (Quantum Ledger Database)**:
  - Fully managed ledger database with immutable, cryptographically verifiable transaction log
  - Centralized and trusted authority
  - SQL-like query language (PartiQL)
  - Use cases: financial transactions, supply chain, registration, medical records

- **Amazon Timestream**:
  - Fully managed, purpose-built time series database service for collecting, storing, and analyzing time-series data.
  - **Key Concepts:**
    - Automatically scales up/down to adjust capacity based on workload
    - Time-optimized storage tiers: recent data in memory, historical data in cost-optimized storage
    - Built-in time series analytics functions
    - Serverless architecture with pay-per-use pricing
  - **Exam Tips:**
    - Use for applications generating large volumes of timestamped data like IoT telemetry, DevOps, industrial equipment
    - 1,000 times faster and 1/10th the cost of relational databases for time series workloads
    - Integrates with AWS IoT, Lambda, and Grafana for visualization
    - Query data across both memory and storage tiers with a single query

---

## 8. Security, Identity & Compliance

### IAM (Identity and Access Management)

[Back to Service Index](#aws-service-quick-index)

- **What:** Manage users, groups, roles, and permissions for AWS resources.
- **Key Concepts:**
  - Users, groups, roles, policies (JSON), MFA, identity federation
  - Managed vs. inline policies, least privilege principle
- **Exam Tips:**
  - IAM is global and free
  - Use roles for EC2/Lambda, never hardcode credentials
  - Explicit deny always overrides allow

### KMS (Key Management Service)

[Back to Service Index](#aws-service-quick-index)

- **What:** Create and manage cryptographic keys for encryption.
- **Key Concepts:**
  - Customer managed keys (CMK), automatic key rotation, grants
- **Exam Tips:**
  - KMS keys are regional
  - Use KMS for S3, EBS, RDS encryption

### CloudHSM

[Back to Service Index](#aws-service-quick-index)

- **What:** Dedicated Hardware Security Module (HSM) in the AWS Cloud that provides secure cryptographic key storage and operations.
- **Key Concepts:**
  - True "single-tenant" hardware security modules
  - FIPS 140-2 Level 3 compliance (compared to KMS at Level 2 overall)
  - Fully customer-managed (AWS provisions the hardware, but customers manage the keys)
  - Supports industry-standard APIs: PKCS#11, Java Cryptography Extensions (JCE), Microsoft CryptoNG (CNG)
- **Exam Tips:**
  - Use when you need the highest level of security compliance or complete control over the key hierarchy
  - KMS can use CloudHSM as a custom key store
  - Ideal for applications that require dedicated hardware for regulatory compliance
  - More expensive and complex to manage than KMS

### AWS Organizations & SCPs

[Back to Service Index](#aws-service-quick-index)

- **What:** Manage multiple AWS accounts centrally.
- **Key Concepts:**
  - Organizational units (OUs), consolidated billing, Service Control Policies (SCPs)
- **Exam Tips:**
  - SCPs set permission boundaries for accounts, but do not grant permissions

### ACM (AWS Certificate Manager)

[Back to Service Index](#aws-service-quick-index)

- **What:** Service for provisioning, managing, and deploying SSL/TLS certificates for use with AWS services.
- **Key Concepts:**
  - Free public certificates for AWS resources
  - Automated certificate renewal
  - Integration with Elastic Load Balancing, CloudFront, API Gateway
  - Support for importing certificates from third-party issuers
- **Exam Tips:**
  - Simplifies certificate management by handling provisioning, deployment, and renewal
  - Certificates generated by ACM can only be used with AWS integrated services, not on EC2 directly
  - Private certificates require a private CA (Certificate Authority)
  - Supports both wildcard and multi-domain certificates

---

## 9. Monitoring & Management

### CloudWatch

[Back to Service Index](#aws-service-quick-index)

- **What:** Monitoring and observability service for AWS resources and applications.
- **Key Concepts:**
  - Metrics, logs, alarms, dashboards, events
  - Custom metrics, CloudWatch Agent for on-premises/EC2
- **Exam Tips:**
  - Use alarms for auto-scaling and notifications
  - CloudWatch Logs for log aggregation and analysis

### CloudTrail

[Back to Service Index](#aws-service-quick-index)

- **What:** Tracks API calls and user activity across AWS.
- **Key Concepts:**
  - Event history, trails, S3/CloudWatch Logs integration
- **Exam Tips:**
  - CloudTrail is enabled by default for 90 days of event history
  - Use for auditing and compliance

### AWS Config

[Back to Service Index](#aws-service-quick-index)

- **What:** Tracks resource configuration changes and compliance.
- **Key Concepts:**
  - Rules, configuration history, snapshots
- **Exam Tips:**
  - Use Config Rules to enforce compliance
  - Snapshot feature for historical configuration tracking

### Control Tower

[Back to Service Index](#aws-service-quick-index)

- **What:** Service that provides a simplified way to set up and govern a secure, multi-account AWS environment (landing zone).
- **Key Concepts:**
  - Automates the setup of a landing zone that follows AWS best practices
  - Implements guardrails (preventive and detective controls)
  - Account Factory for standardized account provisioning
  - Dashboard for visibility into policy compliance
- **Exam Tips:**
  - Use for creating and managing multi-account environments with compliance requirements
  - Built on top of AWS Organizations, CloudFormation, Config, and other services
  - Guardrails enforce governance across all accounts
  - Reduces time to set up multi-account architectures from weeks to days

### X-Ray

[Back to Service Index](#aws-service-quick-index)

- **What:** Service that collects data about your applications and provides tools to view, filter, and gain insights into request data for identifying issues and optimization opportunities.
- **Key Concepts:**
  - Distributed tracing system for microservice architectures
  - End-to-end view of requests as they travel through the application
  - Service map visualization of application components
  - Integrates with many AWS services, including Lambda, API Gateway, and ECS
- **Exam Tips:**
  - Use for debugging and analyzing microservices applications
  - Provides insight into performance bottlenecks and error conditions
  - Requires instrumentation of application code with the X-Ray SDK
  - Sampling rules determine which requests are traced and at what rate

---

## 10. Application Integration

### Amazon SQS (Simple Queue Service)

[Back to Service Index](#aws-service-quick-index)

- **What:** Fully managed message queuing for decoupling and scaling microservices, distributed systems, and serverless apps.
- **Key Concepts:**
  - Standard vs. FIFO queues, message retention, dead-letter queues
  - At-least-once delivery (Standard), exactly-once processing (FIFO)
- **Exam Tips:**
  - Use SQS to decouple application components
  - FIFO queues for order and deduplication

### Amazon SNS (Simple Notification Service)

[Back to Service Index](#aws-service-quick-index)

- **What:** Pub/sub messaging for application-to-application and application-to-person notifications.
- **Key Concepts:**
  - Topics, subscriptions (email, SMS, Lambda, SQS, HTTP/S)
- **Exam Tips:**
  - Use SNS for fan-out to multiple endpoints
  - Integrate SNS with SQS for message durability

### Amazon EventBridge (CloudWatch Events)

[Back to Service Index](#aws-service-quick-index)

- **What:** Serverless event bus for application integration using events from AWS services, SaaS, and custom sources.
- **Key Concepts:**
  - Event buses, rules, targets
- **Exam Tips:**
  - Use EventBridge for event-driven architectures and cross-service automation

### Step Functions

[Back to Service Index](#aws-service-quick-index)

- **What:** Serverless orchestration for workflows across AWS services.
- **Key Concepts:**
  - State machines, tasks, parallel execution, error handling

### Managed Message Broker (MQ)

[Back to Service Index](#aws-service-quick-index)

- **What:** Managed message broker service for Apache ActiveMQ and RabbitMQ that makes it easy to set up and operate message brokers in the cloud.
- **Key Concepts:**
  - Supports industry-standard APIs and protocols (JMS, AMQP, MQTT, OpenWire, STOMP)
  - Provides both queues and topics for messaging
  - Available in single-instance (development) or highly available (production) modes
  - Runs within a VPC for network isolation
- **Exam Tips:**
  - Use when migrating existing applications using message brokers to AWS with minimal code changes
  - Not a public service - requires private network connections for on-premises integration
  - No native AWS service integration like SNS and SQS have
  - Preferred for applications that use open standard messaging protocols and require compatibility with existing systems

### AppFlow

[Back to Service Index](#aws-service-quick-index)

- **What:** Fully managed integration service that enables you to securely transfer data between AWS services and SaaS applications.
- **Key Concepts:**
  - No-code data flow creation
  - Supports various data sources and destinations
  - Data transformation and filtering capabilities
- **Exam Tips:**
  - Use AppFlow for real-time and batch data processing
  - Integrates with services like Salesforce, ServiceNow, and Google Analytics

---

## 11. Migration & Transfer

### AWS DMS (Database Migration Service)

[Back to Service Index](#aws-service-quick-index)

- **What:** Migrate databases to AWS with minimal downtime.
- **Key Concepts:**
  - Supports homogeneous and heterogeneous migrations
- **Exam Tips:**
  - Use DMS for live migrations and continuous replication

### AWS Snowball

[Back to Service Index](#aws-service-quick-index)

- **What:** Physical device for large-scale data transfer to/from AWS.
- **Key Concepts:**
  - Snowball Edge for compute and storage at edge locations
- **Exam Tips:**
  - Use Snowball for petabyte-scale migrations

### AWS Transfer Family

[Back to Service Index](#aws-service-quick-index)

- **What:** Managed SFTP, FTPS, and FTP for file transfers directly into and out of S3.

## DataSync

[Back to Service Index](#aws-service-quick-index)

- **What:** A data transfer service that simplifies, automates, and accelerates moving data between on-premises storage systems and AWS storage services.

- **Key Concepts:**
  - Transfers data over Direct Connect, VPN, or public internet
  - Preserves file metadata (timestamps, ownership, permissions)
  - Built-in data validation to ensure data integrity
  - Components:
    - DataSync Agent: Software appliance installed on-premises
    - Locations: Source and destination endpoints (where data is copied from/to)
    - Tasks: The configuration defining what is being synced and how
  - Bandwidth throttling to control impact on networks

- **Supported Storage:**
  - **Source:** NFS, SMB, HDFS, self-managed object storage
  - **Destination:** S3, EFS, FSx for Windows, FSx for Lustre

- **Exam Tips:**
  - Use for migrating large datasets to AWS (one-time transfers)
  - Use for recurring transfers in hybrid environments
  - More feature-rich than AWS Transfer Family (which focuses on FTP/SFTP protocols)
  - Up to 10x faster than open-source tools
  - Tasks can be scheduled to run hourly, daily, or weekly
  - Pay only for data transferred (no minimum fees or upfront commitments)

---

## 12. Analytics & Machine Learning

### Amazon Athena

[Back to Service Index](#aws-service-quick-index)

- **What:** Serverless, interactive query service for analyzing data in S3 using SQL.
- **Key Concepts:**
  - Pay-per-query, integrates with Glue Data Catalog

### AWS Glue

[Back to Service Index](#aws-service-quick-index)

- **What:** Serverless data integration service for ETL (extract, transform, load).
- **Key Concepts:**
  - Crawlers, jobs, data catalog

### Amazon Kinesis

[Back to Service Index](#aws-service-quick-index)

- **What:** Real-time data streaming and analytics.
- **Key Concepts:**
  - Streams, Firehose, Analytics, Data Streams

### Kinesis Data Firehose

[Back to Service Index](#aws-service-quick-index)

- **What:** Fully managed service for delivering real-time streaming data to destinations such as S3, Redshift, Elasticsearch, and Splunk.
- **Key Concepts:**
  - Automatically scales to match throughput without management
  - Near real-time delivery (~60 seconds)
  - Supports data transformation with Lambda
  - Batch, compress, and encrypt data before delivery
- **Exam Tips:**
  - Use when you need to load streaming data into data stores and analytics tools
  - Can be integrated with Kinesis Data Streams for extended functionality
  - Requires no ongoing administration, capacity planning, or maintenance
  - Pay only for the volume of data processed through the service

### Amazon EMR

[Back to Service Index](#aws-service-quick-index)

- **What:** Managed Hadoop framework for big data processing.

### Amazon QuickSight

[Back to Service Index](#aws-service-quick-index)

- **What:** Scalable business intelligence (BI) service for data visualization.

### Amazon SageMaker

[Back to Service Index](#aws-service-quick-index)

- **What:** Fully managed service that enables data scientists and developers to build, train, and deploy machine learning models quickly.
- **Key Concepts:**
  - Integrated Jupyter notebooks for exploration and analysis
  - Built-in algorithms and support for custom frameworks
  - Managed infrastructure for training at any scale
  - Model deployment with automatic scaling
- **Exam Tips:**
  - Eliminates the heavy lifting of machine learning infrastructure management
  - Supports all major frameworks (TensorFlow, PyTorch, MXNet, etc.)
  - Provides built-in model monitoring and automatic scaling
  - Offers tools for the entire ML lifecycle from data labeling to deployment and monitoring

### Kinesis Data Analytics

[Back to Service Index](#aws-service-quick-index)

- **What:** Fully managed service that enables you to analyze streaming data in real time with SQL or Apache Flink.
- **Key Concepts:**
  - Process and analyze streaming data in real time
  - Use standard SQL queries or Apache Flink applications
  - Ingest data from Kinesis Data Streams or Kinesis Data Firehose
  - Output results to S3, Redshift, Elasticsearch, Kinesis Data Streams, or Lambda
- **Exam Tips:**
  - Use for real-time analytics on streaming data such as log analytics, time-series analytics, and IoT data processing
  - Supports time-based windowing operations for trend analysis
  - Automatically scales to handle any data throughput
  - Pay only for the resources consumed while processing your data

---

## 13. Exam Tips & Scenarios

- Review AWS Well-Architected Framework pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization
- Focus on high availability, fault tolerance, disaster recovery, and security best practices
- Practice scenario-based questions and understand trade-offs between services
- Know when to use which service (e.g., S3 vs. EBS vs. EFS, RDS vs. DynamoDB)
- Use IAM roles and policies for least privilege
- Understand VPC networking, security groups, and NACLs

---

## 14. Glossary

- **Region:** A geographical area with multiple, isolated locations known as Availability Zones
- **Availability Zone (AZ):** A data center or group of data centers within a region
- **Edge Location:** A site for content delivery (CloudFront, Route 53)
- **IAM:** Identity and Access Management
- **S3:** Simple Storage Service
- **EC2:** Elastic Compute Cloud
- **VPC:** Virtual Private Cloud
- **RDS:** Relational Database Service
- **DynamoDB:** Managed NoSQL database
- **CloudWatch:** Monitoring and observability service
- **CloudTrail:** API call logging service
- **KMS:** Key Management Service
- **SCP:** Service Control Policy

---

## Mnemonic Devices

> This section provides memory aids to help you remember complex AWS concepts, service features, and relationships.

### Well-Architected Framework Pillars: "CORPS"

- **C** - Cost Optimization
- **O** - Operational Excellence
- **R** - Reliability
- **P** - Performance Efficiency
- **S** - Security

### S3 Storage Classes: "SIGOGI"

- **S** - Standard (default, high availability)
- **I** - Intelligent-Tiering (automatic cost optimization)
- **G** - Glacier (long-term archival, minutes to hours retrieval)
- **O** - One Zone-IA (lower availability, cheaper)
- **G** - Glacier Deep Archive (lowest cost, longest retrieval)
- **I** - Infrequent Access (Standard-IA) (less frequent access, lower cost)

### EC2 Instance Families: "CMGTRXDIFP"

- **C** - Compute Optimized
- **M** - General Purpose
- **G** - Graphics Optimized
- **T** - Burstable Performance
- **R** - Memory Optimized
- **X** - Memory Optimized (Extreme)
- **D** - Dense Storage
- **I** - I/O Optimized
- **F** - FPGA Instances
- **P** - GPU Instances

### IAM Policy Evaluation: "DREAM"

- **D** - Deny Evaluation (explicit denies always win)
- **R** - Restriction Evaluation (Organizations SCPs)
- **E** - Explicit Allow/Deny (permission boundaries)
- **A** - Allow Evaluation (identity-based policies)
- **M** - More Allow Evaluation (resource-based policies)

### High Availability Services: "DREAD"

- **D** - DynamoDB (multi-region, global tables)
- **R** - RDS (multi-AZ deployments)
- **E** - ElastiCache (multi-AZ with automatic failover)
- **A** - Aurora (multi-AZ with automatic failover)
- **D** - Directory Service (multi-AZ resilience)

### Load Balancer Types: "ANGs"

- **A** - Application Load Balancer (HTTP/HTTPS, layer 7)
- **N** - Network Load Balancer (TCP/UDP, layer 4)
- **G** - Gateway Load Balancer (appliances, layer 3+)

### Security "PACTS"

- **P** - Principle of least Privilege
- **A** - Audit continuously
- **C** - Credentials should rotate regularly
- **T** - Test your security controls
- **S** - Shared Responsibility Model

### Disaster Recovery Options: "BCPW"

- **B** - Backup & Restore (highest RTO/RPO, lowest cost)
- **C** - Cold Standby (reduced RTO/RPO, moderate cost)
- **P** - Pilot Light (minimal resources running, moderate-high cost)
- **W** - Warm Standby (scaled-down version ready, high cost)
- Missing the "Hot Standby" or "Multi-site" which would be highest cost, lowest RTO/RPO

### Database Choices: "TANKER"

- **T** - Transactional (RDS, Aurora)
- **A** - Analytical (Redshift)
- **N** - NoSQL (DynamoDB)
- **K** - Key-value (ElastiCache)
- **E** - ElasticSearch (full-text search)
- **R** - Relational (RDS, Aurora)

### Common Port Numbers: "SSH HTP SMTP"

- **22** - SSH
- **80** - HTTP
- **443** - HTTPS
- **25** - SMTP
- **3306** - MySQL
- **5432** - PostgreSQL

### VPC Components: "SINK RIDGE"

- **S** - Subnets
- **I** - Internet Gateway
- **N** - NAT Gateway
- **K** - Key Pairs (for EC2 access)
- **R** - Route Tables
- **I** - IP Addressing
- **D** - DHCP Options
- **G** - Gateway Endpoints
- **E** - Egress-only Internet Gateway

### Cognito

[Back to Service Index](#aws-service-quick-index)

- **What:** Authentication, authorization, and user management service for web and mobile applications.

- **Key Concepts:**
  - User Pools: User directory service with sign-up/sign-in functionality
    - Provides JSON Web Tokens (JWT) upon authentication
    - Supports MFA and customizable web UI
    - Handles user profiles and directory management
  - Identity Pools: Provides temporary AWS credentials
    - Allows guest access (unauthenticated identities)
    - Supports federated identities (Google, Facebook, SAML, etc.)
    - Assumes IAM roles to access AWS resources
  - Can use both User Pools and Identity Pools together

- **Exam Tips:**
  - Understand the difference between User Pools (authentication) and Identity Pools (authorization)
  - User Pools authenticate users and return tokens; Identity Pools authorize access to AWS resources
  - Identity pools can work with social identity providers and Cognito User Pools
  - Cognito can handle guest users through unauthenticated identities
  - Temporary credentials from Identity Pools use IAM roles

## Detective

[Back to Service Index](#aws-service-quick-index)

- **What:** Security service that automatically collects log data from AWS resources and uses machine learning, statistical analysis, and graph theory to help identify security issues or suspicious activities.

- **Key Concepts:**
  - Analyzes trillions of events from multiple data sources like VPC Flow Logs, CloudTrail, and GuardDuty
  - Creates a unified, interactive view of resource behavior and interactions over time
  - Provides visualizations to identify patterns and anomalies
  - Helps with in-depth security investigations and root cause analysis
  - Complements GuardDuty by providing deep analysis capabilities

- **Exam Tips:**
  - Unlike GuardDuty, which focuses on real-time threat detection, Detective focuses on investigation and analysis
  - No need to set up or maintain complex data collection and analysis infrastructure
  - Works best when used with other AWS security services (GuardDuty, Security Hub, etc.)
  - Stores and analyzes up to a year of historical event data
  - Helps answer questions like "who accessed this resource?" and "what actions did an IAM role take?"
  - No agents required - uses existing AWS log sources

## Systems Manager

[Back to Service Index](#aws-service-quick-index)

- **What:** A suite of management tools that helps you automate operational tasks across your AWS resources.

- **Key Concepts:**
  - Automation of operational tasks
  - Management of AWS resources at scale
  - Secure access to instances without opening inbound ports
  - Several components including:
    - Parameter Store: Secure storage for configuration data and secrets
    - Run Command: Remote execution of commands or scripts
    - Patch Manager: Automated patching process
    - State Manager: Ensures resources maintain a defined state
    - Session Manager: Browser-based shell or CLI access to instances
    - Inventory: Collects metadata about instances and installed software

  - **Parameter Store:**
    - Storage for configuration data and secrets
    - Supports plain text (String, StringList) and encrypted values (SecureString)
    - Hierarchical storage with versioning
    - Integration with KMS for encryption
    - No additional cost for standard parameters

- **Exam Tips:**
  - Requires the SSM Agent to be installed on managed instances
  - Provides a unified interface for multiple AWS resource types
  - Use Parameter Store for storing configuration data instead of hardcoding values
  - Session Manager eliminates the need for bastion hosts or SSH key management
  - Integrates with CloudWatch for monitoring and IAM for access control
  - Supports both EC2 instances and on-premises servers (with hybrid activations)

## Trusted Advisor

[Back to Service Index](#aws-service-quick-index)

- **What:** An online tool that provides real-time guidance to help you provision your resources following AWS best practices.

- **Key Concepts:**
  - Analyzes your AWS environment and makes recommendations in five categories:
    - Cost Optimization: Identifying idle and underutilized resources
    - Performance: Suggestions to improve service performance
    - Security: Identifying security vulnerabilities and closing gaps
    - Fault Tolerance: Increasing resiliency by eliminating single points of failure
    - Service Limits: Notifying when you approach service quotas
  - Different tiers based on AWS Support plan:
    - Basic/Developer: Limited to core checks
    - Business/Enterprise: Full set of checks with programmatic access

- **Exam Tips:**
  - Business and Enterprise Support plans get access to all Trusted Advisor checks
  - Can integrate with CloudWatch Events for automated actions based on alerts
  - Use for identifying cost-saving opportunities, improving security posture, and ensuring optimal resource utilization
  - Complements other AWS services like Cost Explorer (cost) and Inspector (security)
  - Unlike manual auditing, provides continuous, automated guidance
  - Recommendations are based on AWS best practices and the experience of serving hundreds of thousands of AWS customers

## AWS Backup

[Back to Service Index](#aws-service-quick-index)

- **What:** A fully managed, centralized backup service that makes it easy to automate the backup of data across AWS services.

- **Key Concepts:**
  - Centralized management for backups across multiple AWS services
  - Policy-based backup solution with scheduling capabilities
  - Support for cross-region and cross-account backup
  - Backup plans define backup frequency, window, and retention periods
  - Backup vaults store and organize recovery points

- **Supported AWS Services:**
  - Amazon EC2, EBS, RDS, Aurora, DynamoDB
  - Amazon EFS, FSx for Windows File Server, FSx for Lustre
  - Amazon S3, VMware workloads, AWS Storage Gateway

- **Key Features:**
  - Point-in-time recovery for supported services
  - Lifecycle management to automatically transition recovery points to cold storage
  - AWS Organizations integration for multi-account management
  - Tag-based backup policies
  - Access control via IAM

- **Exam Tips:**
  - Use when you need to centralize and automate backup across multiple AWS services
  - Simplifies compliance requirements by allowing consistent backup policies
  - More cost-effective than managing individual service backups
  - Can help meet Recovery Point Objectives (RPO) through scheduled, automated backups
  - Provides a consistent way to protect AWS workloads across multiple services
  - Does not eliminate service-specific backup options, but provides a unified management layer

## Elastic Load Balancing (ELB)

[Back to Service Index](#aws-service-quick-index)

- **What:** A service that automatically distributes incoming application traffic across multiple targets, such as EC2 instances, containers, and IP addresses.

- **Types of Load Balancers:**
  - **Application Load Balancer (ALB)** - Layer 7 load balancer for HTTP/HTTPS traffic
  - **Network Load Balancer (NLB)** - Layer 4 load balancer for TCP, TLS, UDP traffic
  - **Gateway Load Balancer (GWLB)** - For deploying, scaling, and managing third-party virtual appliances
  - **Classic Load Balancer (CLB)** - Legacy load balancer (not recommended for new implementations)

- **Application Load Balancer (ALB):**
  - Works at Layer 7 (application layer)
  - Content-based routing with rules based on HTTP headers, paths, host headers
  - Support for WebSockets
  - HTTP/HTTPS traffic only (SSL termination)
  - Can route to multiple target groups based on URL paths or host headers
  - Integrates with AWS WAF for application-level protection
  - Slower than NLB but more feature-rich for HTTP/HTTPS applications

- **Network Load Balancer (NLB):**
  - Works at Layer 4 (transport layer)
  - Handles millions of requests per second with ultra-low latency
  - Supports static IP addresses for each AZ and elastic IP addresses
  - Preserves source IP address of clients
  - Supports TCP, UDP, TLS protocols
  - Ideal for non-HTTP protocols (SMTP, SSH, game servers, financial apps)
  - Supports unbroken encryption (end-to-end TLS)

- **Gateway Load Balancer (GWLB):**
  - Helps deploy, scale, and manage third-party virtual appliances
  - Examples: firewalls, intrusion detection/prevention systems, deep packet inspection
  - Uses GENEVE protocol (port 6081)
  - Combines functions of a transparent network gateway and load balancer
  - Works with GWLB endpoints to route traffic to/from virtual appliances

- **Common Features:**
  - Health checks to ensure traffic is only sent to healthy targets
  - Integration with Auto Scaling Groups
  - Support for sticky sessions (session affinity)
  - Cross-zone load balancing for even distribution across AZs
  - Access logs to S3 for detailed connection information
  - Connection draining (deregistration delay) for graceful target removal

- **Exam Tips:**
  - Choose ALB for HTTP/HTTPS applications and content-based routing
  - Choose NLB for highest performance, static IPs, or non-HTTP protocols
  - Choose GWLB for deploying security appliances and traffic inspection
  - Load balancers operate in multiple AZs for high availability
  - ALB terminates HTTPS connections; NLB allows end-to-end encryption
  - Internal load balancers have only private IPs; internet-facing have public IPs
  - Target groups can include EC2 instances, IP addresses, Lambda functions, or containers
