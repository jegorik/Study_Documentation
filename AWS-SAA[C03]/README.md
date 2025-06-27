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
- [Auto Scaling](#auto-scaling-groups) - Dynamic Resource Scaling

### Storage

- [S3](#amazon-s3-expanded) - Simple Storage Service
- [EBS](#amazon-ebs) - Elastic Block Store
- [EFS](#amazon-efs) - Elastic File System
- [Storage Gateway](#storage-gateway) - Hybrid Storage

### Networking

- [VPC](#amazon-vpc-expanded) - Virtual Private Cloud
- [Route 53](#route-53) - DNS Service
- [CloudFront](#cloudfront) - CDN
- [Direct Connect](#direct-connect) - Dedicated Network Connection
- [API Gateway](#api-gateway) - API Management
- [Global Accelerator](#global-accelerator) - Network Performance

### Database

- [RDS](#amazon-rds) - Relational Database Service
- [DynamoDB](#dynamodb-expanded) - NoSQL Database
- [Aurora](#aurora) - Managed MySQL/PostgreSQL
- [ElastiCache](#specialized-database-services) - In-Memory Cache
- [Neptune](#specialized-database-services) - Graph Database
- [DocumentDB](#specialized-database-services) - Document Database
- [Redshift](#specialized-database-services) - Data Warehouse

### Security & Identity

- [IAM](#iam-identity-and-access-management) - Identity & Access Management
- [Cognito](#8-security-identity--compliance) - User Identity & Data Sync
- [KMS](#kms-key-management-service) - Key Management
- [WAF](#8-security-identity--compliance) - Web Application Firewall
- [Shield](#8-security-identity--compliance) - DDoS Protection
- [GuardDuty](#8-security-identity--compliance) - Threat Detection

### Management & Monitoring

- [CloudWatch](#cloudwatch) - Monitoring & Observability
- [CloudTrail](#cloudtrail) - API Activity Tracking
- [AWS Config](#aws-config) - Resource Configuration
- [Systems Manager](#9-monitoring--management) - Operations Management
- [Trusted Advisor](#9-monitoring--management) - Optimization Recommendations

### Integration

- [SQS](#amazon-sqs-simple-queue-service) - Simple Queue Service
- [SNS](#amazon-sns-simple-notification-service) - Simple Notification Service
- [EventBridge](#amazon-eventbridge-cloudwatch-events) - Event Bus
- [Step Functions](#step-functions) - Workflow Orchestration

### Migration

- [DMS](#aws-dms-database-migration-service) - Database Migration Service
- [Snowball](#aws-snowball) - Large-scale Data Transfer
- [Transfer Family](#aws-transfer-family) - Managed File Transfer

### Analytics

- [Athena](#amazon-athena) - Serverless Query Service
- [Glue](#aws-glue) - ETL Service
- [Kinesis](#amazon-kinesis) - Real-time Data Streaming
- [EMR](#amazon-emr) - Managed Hadoop Framework
- [QuickSight](#amazon-quicksight) - Business Intelligence

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

- **What:** Block storage for EC2 instances.
- **Key Concepts:**
  - Persistent, can be detached/reattached
  - Snapshots for backup
- **Exam Tips:**
  - Only EBS is billed when EC2 is stopped

### Amazon EFS

- **What:** Managed NFS file system for Linux EC2 instances.
- **Key Concepts:**
  - Shared, scalable, pay-per-use

### Storage Gateway

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

- **What:** Managed DNS service.
- **Key Concepts:**
  - Hosted zones, record types (A, AAAA, CNAME, MX, TXT, NS)
  - Routing policies: Simple, weighted, latency, failover, geolocation
- **Exam Tips:**
  - S3 static website hosting requires Route 53 for custom domains

### CloudFront

- **What:** Content Delivery Network (CDN) for caching and accelerating content.
- **Key Concepts:**
  - Edge locations, distributions, origins, behaviors
  - Supports static and dynamic content
  - Integrates with S3, EC2, ELB, Route 53
- **Exam Tips:**
  - Use CloudFront to reduce latency and offload traffic from origin
  - Can restrict access to S3 using signed URLs/cookies

### Direct Connect

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

---

## 7. Database Services

### Amazon RDS

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

- **What:** AWS's high-performance, MySQL- and PostgreSQL-compatible relational database.
- **Key Concepts:**
  - Up to 15 read replicas, automatic failover, storage auto-scaling

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
  - Managed RDS service with OS and database customization access
  - Supports Oracle and Microsoft SQL Server
  - Allows installation of custom agents and patches
  - Maintains automated backups, monitoring, and scaling of RDS

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
  - Fully managed time series database
  - Automatically scales up/down to adjust capacity
  - Stores recent data in memory and historical data in cost-optimized storage
  - Includes built-in time series analytics functions
  - Ideal for IoT applications, DevOps, industrial telemetry

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

- **What:** Create and manage cryptographic keys for encryption.
- **Key Concepts:**
  - Customer managed keys (CMK), automatic key rotation, grants
- **Exam Tips:**
  - KMS keys are regional
  - Use KMS for S3, EBS, RDS encryption

### AWS Organizations & SCPs

- **What:** Manage multiple AWS accounts centrally.
- **Key Concepts:**
  - Organizational units (OUs), consolidated billing, Service Control Policies (SCPs)
- **Exam Tips:**
  - SCPs set permission boundaries for accounts, but do not grant permissions

---

## 9. Monitoring & Management

### CloudWatch

- **What:** Monitoring and observability service for AWS resources and applications.
- **Key Concepts:**
  - Metrics, logs, alarms, dashboards, events
  - Custom metrics, CloudWatch Agent for on-premises/EC2
- **Exam Tips:**
  - Use alarms for auto-scaling and notifications
  - CloudWatch Logs for log aggregation and analysis

### CloudTrail

- **What:** Tracks API calls and user activity across AWS.
- **Key Concepts:**
  - Event history, trails, S3/CloudWatch Logs integration
- **Exam Tips:**
  - CloudTrail is enabled by default for 90 days of event history
  - Use for auditing and compliance

### AWS Config

- **What:** Tracks resource configuration changes and compliance.
- **Key Concepts:**
  - Rules, configuration history, snapshots
- **Exam Tips:**
  - Use Config Rules to enforce compliance
  - Snapshot feature for historical configuration tracking

---

## 10. Application Integration

### Amazon SQS (Simple Queue Service)

- **What:** Fully managed message queuing for decoupling and scaling microservices, distributed systems, and serverless apps.
- **Key Concepts:**
  - Standard vs. FIFO queues, message retention, dead-letter queues
  - At-least-once delivery (Standard), exactly-once processing (FIFO)
- **Exam Tips:**
  - Use SQS to decouple application components
  - FIFO queues for order and deduplication

### Amazon SNS (Simple Notification Service)

- **What:** Pub/sub messaging for application-to-application and application-to-person notifications.
- **Key Concepts:**
  - Topics, subscriptions (email, SMS, Lambda, SQS, HTTP/S)
- **Exam Tips:**
  - Use SNS for fan-out to multiple endpoints
  - Integrate SNS with SQS for message durability

### Amazon EventBridge (CloudWatch Events)

- **What:** Serverless event bus for application integration using events from AWS services, SaaS, and custom sources.
- **Key Concepts:**
  - Event buses, rules, targets
- **Exam Tips:**
  - Use EventBridge for event-driven architectures and cross-service automation

### Step Functions

- **What:** Serverless orchestration for workflows across AWS services.
- **Key Concepts:**
  - State machines, tasks, parallel execution, error handling

---

## 11. Migration & Transfer

### AWS DMS (Database Migration Service)

- **What:** Migrate databases to AWS with minimal downtime.
- **Key Concepts:**
  - Supports homogeneous and heterogeneous migrations
- **Exam Tips:**
  - Use DMS for live migrations and continuous replication

### AWS Snowball

- **What:** Physical device for large-scale data transfer to/from AWS.
- **Key Concepts:**
  - Snowball Edge for compute and storage at edge locations
- **Exam Tips:**
  - Use Snowball for petabyte-scale migrations

### AWS Transfer Family

- **What:** Managed SFTP, FTPS, and FTP for file transfers directly into and out of S3.

---

## 12. Analytics & Machine Learning

### Amazon Athena

- **What:** Serverless, interactive query service for analyzing data in S3 using SQL.
- **Key Concepts:**
  - Pay-per-query, integrates with Glue Data Catalog

### AWS Glue

- **What:** Serverless data integration service for ETL (extract, transform, load).
- **Key Concepts:**
  - Crawlers, jobs, data catalog

### Amazon Kinesis

- **What:** Real-time data streaming and analytics.
- **Key Concepts:**
  - Streams, Firehose, Analytics, Data Streams

### Amazon EMR

- **What:** Managed Hadoop framework for big data processing.

### Amazon QuickSight

- **What:** Scalable business intelligence (BI) service for data visualization.

### Amazon SageMaker

- **What:** Fully managed service for building, training, and deploying machine learning models at scale.

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
