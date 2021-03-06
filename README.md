# Notes for AWS Certified Solutions Architect Associate

I recently got the AWS solutions architect associate certificate in July 2019, and wanted to share my notes with anyone who might benefit from it. The path I followed was

- Go through the [ACloudGuru course](https://click.linksynergy.com/deeplink?id=aosskmXRdYk&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Faws-certified-solutions-architect-associate%2F).
- Attempt the [Whizlabs practice tests](https://www.whizlabs.com/aws-solutions-architect-associate/). After each test, note down the concepts I had difficulties with. 
- Attempt the [practice tests by Jon Bonso at Udemy](https://click.linksynergy.com/deeplink?id=aosskmXRdYk&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Faws-certified-solutions-architect-associate-amazon-practice-exams%2F). Again, after each test, note down the concepts I had difficulties with.

So you should go through the notes only after you have done a course that explains the basics, such as the one from ACloudGuru. Also, full disclosure, the links to the above courses are referral ones. So if these notes helped you and you're planning to buy the courses or practices tests, please consider going through the links when you're buying.

__Note__ — You can also check out [this blog post](https://sumit-ghosh.com/articles/aws-solutions-architect-associate-preparation/) where I describe my preparation strategy in detail. 

## Contents

- [Well-Architected Framework](#well-architected-framework)
- [Route 53 / DNS](#route53)
- [S3](#s3)
- [RDS, Redshift and ElastiCache](##rds-redshift-and-elasticache)
- [EC2 - EBS](#ec2-and-ebs)
- [EFS](#efs)
- [ELB and Autoscaling](#elb-and-autoscaling)
- [SQS](#sqs)
- [SNS](#sns)
- [API Gateway](#api-gateway)
- [Lambda](#lambda)
- [VPC](#vpc)
- [DynamoDB](#dynamodb)
- [ECS](#ecs)
- [Elastic Beanstalk](#elastic-beanstalk)
- [Storage Gateway](#storage-gateway)
- [IAM, Cognito and Directory Services](#iam-cognito-and-directory-services)
- [KMS and CloudHSM](#kms-and-cloudhsm)
- [Kinesis](#kinesis)
- [EMR](#emr)
- [CloudFront](#cloudfront)
- [CloudTrail](#cloudtrail)
- [Misc](#misc)



# Well-Architected Framework

The five pillars are —

1. Operational Excellence 
2. Security
3. Reliability
4. Performance Efficiency
5. Cost Optimization


## Operational Excellence

### Design Principles

- Perform operations as code
- Annotate documents
- Make frequent, small, reversible changes
- Refine operations procedures frequently
- Anticipate failure
- Learn from all operational failures

### Best Practices

- Prepare
- Operate
- Evolve

__Key AWS Service__ — AWS CloudFormation.


## Security

### Design Principles

- Implement a strong identity foundations
- Enable traceability
- Apply security at all layers
- Automate security best practices
- Protect data in transit and at rest
- Keep people away from data
- Prepare for security events

### Best Practices

- Identity and Access Management
- Detective Controls
- Infrastructure Protection
- Data Protection
- Incident Response

__Key AWS Service__ — AWS Identity and Access Management (IAM).


## Reliability

### Design Principles

- Test recovery procedures
- Automatically recover from failure
- Scale horizontally to increase aggregate system availability 
- Stop guessing capacity
- Manage change in automation

### Best Practices

- Foundations
- Change Management
- Failure Management

__Key AWS Service__ — Amazon CloudWatch.


## Performance Efficiency

### Design Principles

- Democratize advanced technologies
- Go global in minutes
- Use serverless architecture
- Experiment more often
- Mechanical sympathy

### Best Practices

- Selection
    - Compute
    - Storage
    - Database
    - Network
- Review
- Monitoring
- Tradeoffs

__Key AWS Service__ — Amazon CloudWatch.


## Cost Optimization

### Design Principles

- Adopt a consumption model
- Measure overall efficiency
- Stop spending money on data center operations
- Analyze and attribute expenditure
- Use managed and application level services to reduce cost of ownership

### Best Practices

- Expenditure Awareness
- Cost-Effective Resources
- Matching Supply and Demand
- Optimizing Over Time

__Key AWS Service__ — Cost Explorer.





# Route53


__Main functions of Route53__ —
1. Register domain names.
2. Route internet traffic to the resources for your domain.
3. Check the health of your resources.

It's not used to _distribute_ traffic.

__CNAME vs ALIAS__ —  

- For routing to S3 bucket // Elastic load balancer // CloudFront distribution use A record with ALIAS.  
- For routing to RDS instance use CNAME with NO ALIAS // without ALIAS.

A CNAME is only suitable for subdomains.

ALIAS only supports the following services —
- API Gateway
- VPC interface endpoint
- CloudFront distribution
- Elastic Beanstalk environment
- ELB load balancer
- S3 bucket that is configured as a static website
- Another Route 53 record in the same hosted zone


Route53 does not directly log to S3 bucket, we can forward that from Cloudwatch, but can't do it directly.

Types of __Route53 health checks__ —
1. Health checks that monitor __an endpoint__. This __can be on-premise__ too.
2. Health checks that monitor __other health checks__.
3. Health checks that monitor __Cloudwatch alarms__. 

__Multivalue answer routing policy__ responds with upto 8 healthy records selected at __random__.

__Weighted routing policy__ is a good fit for __blue-green deployments__.


**Routing policy**
- **Simple** routing policy – Use for a single resource that performs a given function for your domain, for example, a web server that serves content for the example.com website.

- **Failover** routing policy – Use when you want to configure active-passive failover.

- **Geolocation** routing policy – Use when you want to route traffic based on the location of your users.

- **Geoproximity** routing policy – Use when you want to route traffic based on the location of your resources and, optionally, shift traffic from resources in one location to resources in another.

- **Latency** routing policy – Use when you have resources in multiple AWS Regions and you want to route traffic to the region that provides the best latency.

- **Multivalue answer** routing policy – Use when you want Route 53 to respond to DNS queries with up to eight healthy records selected at random.

- **Weighted** routing policy – Use to route traffic to multiple resources in proportions that you specify.

By design, The AWS DNS service does not respond to requests originating from outside the VPC.

**Private zones** need **enableDnsHostnames** and **enableDnsSupport** enabled on a VPC.

Amazon Route 53’s DNS services does not support DNSSEC at this time.

# S3

In a newly created S3 bucket, everything // every additional option is turned off by default. Also, no bucket policy exists.

__S3 bucket properties__ are —
1. Versioning
2. Server access logging (track every request access to S3 buckets including the requester, bucket name, request time, request action, referrer, turnaround time, and error code information.)
3. Static website hosting
4. Object level logging // Essentially CloudTrail
5. Transfer acceleration
6. Events

__Object level properties__—  
Metadata and Storage class are object level properties. All object level properties are
1. Storage class
2. Encryption
3. Metadata
4. Tags
5. Object lock

__DELETE operation__ does not keep a copy unless you have versioning enabled. From the docs
> The DELETE operation removes the null version (if there is one) of an object and inserts a delete marker, which becomes the current version of the object. If there isn't a null version, Amazon S3 does not remove any objects. 

S3 is a __managed service__. It can't be part of a VPC.

__S3 object metadata__—
1. System metadata
2. User-defined metadata

User defined metadatas must start with `x-amz-meta`.

When you enable logging on a bucket, the console both enables logging on the source bucket and adds a grant in the target bucket's access control list (ACL) granting write permission to the Log Delivery Group.

__S3 bucket endpoints formats__ —
1. http://bucket.s3.amazonaws.com
2. http://bucket.s3.aws-region.amazonaws.com
3. http://s3.amazonaws.com/bucket (this is only valid if the bucket is in us-east-1)
4. http://s3.aws-region.amazonaws.com/bucket


__Object sizes__ —
S3 can store objects of size 0 bytes to 5 TB.
A single PUT can transfer 5 GB max. For files larger than 100MB, multipart upload is recommended.

__Cross-region replication__ requires that versioning be enabled on both the source bucket and the destination bucket. Files in an existing bucket are not replicated automatically.

__AWS Glacier archive retrieval__ options —
- Expedited: Costly, 1-5 minutes.
- Standard: Default, 3-5 hours.
- Bulk: Cheapest, 5-12 hours.

Glacier has a management console which you can use to create and delete vaults. However, **you cannot directly upload archives to Glacier by using the management console.**

To increase performance, we can __prefix each object name with a hash key__ along with the current date. But, according to the new S3 performance announcement, this is __not needed anymore__.

__Increasing performance in S3__ —
- If workload is mainly GET requests, integrate Cloudfront with S3.
- If workload consists of PUT requests, use S3 transfer acceleration.

In the __CORS__ configuration, the __exact URLs__ must be added, with the correct protocol, i.e. __http vs https__.

__S3__ does not support `OPTIONS`, `CONNECT` and `TRACE` __methods__. 

__S3 encryptions__ —
- Server-Side Encryption (uses __AES 256__)
  - SSE-S3: Data and master keys managed by S3.
  - SSE-C: The user manages the encryption keys.
  - SSE-KMS: AWS manages the data key, the user manages the master key.
- Client-Side Encryption (encrypting data before sending it to Amazon S3)
  - Use a customer master key (CMK) stored in AWS Key Management Service (AWS KMS).
  - Use a master key you store within your application.

To **restrict access** to content that you serve from Amazon **S3 buckets**, you create **CloudFront signed URLs or signed cookies** to limit access to files in your Amazon S3 bucket, and then you create a **special CloudFront user** called an **Origin Access Identity (OAI)** and **associate it with your distribution**. Then you configure **permissions so that CloudFront can use the OAI to access and serve files** to your users, but users can't use a direct URL to the S3 bucket to access a file there.

We can create __event notification in S3__ to __invoke lambda__ function.

__Customer managed S3 encryption workflow__ —  
Generate a data key using Customer managed CMK. Encrypt data using data key and delete data key. Store encrypted data key and data in S3 buckets. For decryption, use CMK to decrypt data key into plain text and then decrypt data using plain text data key.

AWS S3 __performance__ —

- 3,500 requests per second to add data
- 5,500 requests per second to retrieve data

__Provisioned capacity__ should be used when we want to guarantee the availability of fast expedited retrieval from S3 Glacier within minutes.

S3 __event notification targets__:

- SQS
- SNS
- Lambda

An 80 TB __Snowball__ appliance and 100 TB Snowball Edge appliance only have 72 TB and 83 TB of __usable capacity__ respectively.

For [__S3 static website hosting__, the default provided __URL__](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html) is https://bucket-name.s3-website-aws-region.amazonaws.com. Looks like, as well, it's https://bucket-name.s3-website.aws-region.amazonaws.com.

For __static website hosting__ with S3, the name of the bucket must be the same as the domain or subdomain name.

__Preventing accidental deletion__ of S3 objects —

- Enable versioning
- Enable MFA delete

Read after Write consistency for PUTS of new Objects. Eventual Consistency for overwrite PUTS and DELETES (can take some time to propagate). Translation: new objects can be retrieved (almost) immediately while updates or deletes are not immediate and take some time to propagate.

**Cross-origin resource sharing (CORS)** defines a way for client web applications that are loaded in one domain to interact with resources in a different domain. With CORS support, you can build rich client-side web applications with Amazon S3 and selectively allow cross-origin access to your Amazon S3 resources.

Objects must be stored at least 30 days in the current storage class before you can transition them to STANDARD_IA or ONEZONE_IA. Amazon S3 doesn't transition objects within the first 30 days because newer objects are often accessed more frequently or deleted sooner than is suitable for STANDARD_IA or ONEZONE_IA storage.

# RDS, Redshift and ElastiCache

Redshift it's primarily **used for OLAP applications** and not for OLTP. It doesn't scale automatically to handle the exponential growth of the database.

Amazon __Redshift Enhanced VPC Routing__ provides VPC resources the access to Redshift.

Amazon __ElastiCache__ offers fully managed __Redis and Memcached__. 

__Cross-region replication__ can be setup for __Redshift Clusters__.

__Redshift encryption__ —
- Using AWS KMS to encrypt the underlying data.
- Using S3 and its encryption.

__RDS data size limits__ —
- Aurora: 64 TB
- Others: 16 TB.

**RDS Read replicas**:
- Aurora: Up to 15.
- Others: Up to 5.

**Read replicas** help with:
- Disaster recovery: Cross-region, promoting replicas
- Read performance

**Multi-AZ deployments** help with *High Availability*. Plus, automatically creates a primary DB Instance and **synchronously replicates** the data to a standby instance in a different Availability Zone (AZ).

During automated backup, Amazon RDS performs a storage volume snapshot of entire Database instance. Also, it captures transaction logs every 5 minutes.

__AWS RDS is a service__ whereas __AWS Aurora is a database engine__.

For __Redshift__, spot instances are not an option.

__Encryption of RDS__ — Have to enable it on database creation. Also, not all instance classes support encryption, we have to choose one which supports it.

To enable __multi-region replication of RDS__, we have to use __Read Replicas__. Multi-AZ is not the solution here.

__RDS Read Replicas__ are __synced asynchronously__, so it can have __replication lag__.

__Redshift automated snapshot retention period__ — 1 day to 35 days.

We can't use auto-scaling with __RDS__. To improve __performance__, we should look to __sharding__ instead. Starting from __June 20__, we __can use auto-scaling__ with RDS instances.

We configure __RDS engine configurations__ using __parameter groups__.

To use __REDIS AUTH with ElastiCache__, __in-transit encryption__ must be enabled for clusters.

In a **ElastiCache cluster**, to **secure the session data** in the portal by **requiring** them to enter a **password before** they are **granted permission** to **execute Redis commands**, you can authenticate the users using Redis AUTH by creating a new Redis Cluster with both `--transit-encryption-enabled` and `--auth-token` parameters enabled.

For RDS, __Enhanced Monitoring__ gathers its metrics from an __agent on the instance__.

**RDS Enhanced Monitoring metrics**:

- **RDS child processes**.
- **RDS processes**.
- **OS processes**.

In case of a __failover__, Amazon RDS flips the canonical name record (__CNAME__) for your DB instance to point at the standby. Aurora will attempt to create a new instance in the same availability zone as the original. If unable to do so, Aurora will attempt to create it in a different AZ.

__Aurora endpoints__, by default — 

- A reader endpoint. It load balances all read traffic between instances.
- A cluster endpoint. For write operations.

We can create __additional custom endpoints__ that load balance based on specified criteria.

With __Redshift Spectrum__, we can run complex queries on __data stored in S3__.

We can use __WLM in the parameter group configuration__ of Redshift to define number of query queues and how queries are routed to those queues.

The memory and processor __usage by each process__ in an RDS instance can not be monitored by Cloudwatch, we have to use __RDS Enhanced Monitoring__ for that. Because Cloudwatch monitors the hypervisor, not the individual instances.

__IAM DB authentication__ can be used with __MySQL and PostgreSQL__. With this, you don't need to use a password when you connect to a DB instance. Instead, you use an __authentication token__.

**MS SQL** and **Oracle** database engines have a **limit** to the **number of databases that can run per instance**

In case you require intense, **full-text search capability, use MyISAM storage** engine instead.

**IAM database authentication** is only supported in **MySQL and PostgreSQL** database engines.

# EC2 and EBS

__Instance store__ —
You cannot add instance store volume to an instance after it's launched.
Not all EC2 instance types support instance store volume.

Persistence — Instance store persists during reboots, but not stop or terminate. EBS volumes however persists accross reboot, stop, and terminate.

When you create an encrypted EBS volume and attach it to a supported instance type, the following types of data are encrypted:

   - Data at rest inside the volume

   - All data moving between the volume and the instance

   - All snapshots created from the volume

   - All volumes created from those snapshots

__EBS volume types__ —
1. Provisioned IOPS SSD. I/O-Intensive NoSQL and relational databases. 4GB - 16TB. Max 64000 IOPS. Max 1000MB/s throughput. IO1 is designed to deliver a consistent baseline performance of **up to 50 IOPS/GB**. *For critical high performing databases.*
2. General purpose SSD. Boot volumes, low-latency interactive apps, dev&test. 1GB - 16TB. Max 16000 IOPS. Max 250MB/s throughput. The performance of gp2 volumes is tied to volume size. Volumes earn I/O credits at the baseline performance rate of 3 IOPS per GiB of volume size. *For web applications // most use cases.*
3. Throughput optimized HDD. For Big Data, data warehouses, log processings. 500GB - 16TB. Max 500 IOPS. Max 500MB/s throughput. *For Big Data.*
4. Cold HDD. Colder data requiring fewer scans per day. 500GB - 16TB. Max 250 IOPS. Max 250MB/s throughput. *For infrequently accessed data.*

Also, to note, __HDDs cannot be boot volumes__.

We can use Amazon __Data Lifecycle Manager__ to automate taking backups // snapshots of EBS volumes, and protect them from accidental or unwanted deletion.

__EBS-optimized EC2 instances__ provide additional, dedicated capacity for EBS IO. Helps squeeze out the last ounce of performance.

Encrypted EBS volumes are not supported on all instance types.

__To get more performance out of EBS volumes__ —
1. Use a more modern Linux Kernel.
2. Use RAID 0.

VolumeRemainingSize is not an Cloudwatch metric for EBS volumes.

__EBS volume types__ —
- For throughput, Throughput optimized HDD.
- For large number of transaction, i.e. IOPS, Provisioned IOPS SSD.

By default, __EBS volumes are automatically replicated within their availability zone__, and offers a significant high availability.

__AWS Cloudwatch Logs__ can be used to __monitor and store__ logs from EC2 instances. The instance needs __awslogs log driver__ installed to be able to send logs to CloudWatch. We don't need any database or S3 for storage.

Cloudwatch logs agent is __more efficient__ than AWS SSM Agent.

With __EC2 dedicated hosts__ we have control over __number of cores__, not anywhere else.

**Placement groups**
- **Cluster**: packs instances **close together inside an Availability Zone**. This strategy enables workloads to achieve the **low-latency network performance** necessary for **tightly-coupled node-to-node communication** that is typical of **HPC applications**.

- **Partition**: spreads your instances across logical partitions such that groups of instances in one partition do not share the underlying hardware with groups of instances in different partitions. This strategy is typically used by large distributed and replicated workloads, such as Hadoop, Cassandra, and Kafka.

- **Spread**: strictly places a small group of instances across distinct underlying hardware to reduce correlated failures. Maximum of 7 running instances per Availability Zone

The __console does not support placement groups__, have to do it from CLI.

__Cluster Placement groups__ have very __low inter-note latency__.


__Hibernation of EC2 instances__ —
- When EC2 instance is hibernated and brought back up, the public IP4 address is renewed. All the other IP addresses are retained.
- When EC2 instance is in hibernate, you are only charged for elastic IP address and EBS storage space.

__Default Cloudwatch metrics__ —

- CPU utilization
- Disk reads and writes
- Network in and out

__Custom metrics__ —

- Memory utilization
- Disk swap utilization
- Disk space utilization
- Page file utilization
- Log collection

__Reserved Instances that are terminated__ are **still billed** until the end of their term according to their payment option.

Upon __stopping and starting an EC2 instance__ —

- Elastic IP address is disassociated from the instance if it is an EC2-Classic instance. Otherwise, if it is an EC2-VPC instance, the Elastic IP address remains associated.
- The underlying physical host is possibly changed.

EBS is __lower-latency__ than EFS.

The maximum ratio of __provisioned IOPS__ to requested volume size (in GiB) is 50:1.

For __new accounts__, Amazon has a __soft limit of 20 EC2 instances per region__, which can be removed by contacting Amazon.

You can attach a network interface (ENI) to an EC2 instance in the following ways —

1. When it's running. Hot attach.
2. When it's stopped. Warm attach.
3. When the instance is being launched. Cold attach.

EBS snapshots are more efficient and cost-effective solution compared to __disk mirroring using RAID 1__.

EBS volumes can only be attached to an EC2 instance in the __same Availability Zone__.

__EBS snapshot creation__ — In usual scenarios EBS volume snapshots can be created at the same time it's in usage. But when using RAID configuraions, there are additional complexities and we should stop every IO operation and flush the cache before taking a snapshot.

__Cloudwatch alarm actions__ can automatically start, stop or reboot EC2 instances based on alarms.

With __scheduled reserved instances__, we can plan out our future usage and get reserved instances in those planned time-frame only.

__Throughput optimized HDD vs Cold HDD__ — Throughput optimized is used for frequently accessed data, whereas Cold HDD is used for infrequently accessed data. Also the later is more cost-effective.

__RAID 0 vs RAID 1__ —

- RAID 1 is used for mirroring, high-availability and redundancy.
- RAID 0 is used for higher performance, it can combine multiple disk drives together.

Larger EC2 instances have higher disk data throughput. This can be used in conjunction with RAID 0 to __improve EBS performance__.

An Elastic IP address doesn’t incur charges as long as the following conditions are true:

- The Elastic IP address is associated with an Amazon EC2 instance.
- The instance associated with the Elastic IP address is running.
- The instance has only one Elastic IP address attached to it.

If you’ve stopped or terminated an EC2 instance with an associated Elastic IP address and you don’t need that Elastic IP address anymore, consider disassociating or releasing the Elastic IP address .

If your Spot instance is terminated or stopped by Amazon EC2 in the first instance hour, you will not be charged for that usage

# EFS

EFS supports cross availability zone mounting, but it is not recommended. The recommended approach is __creating a mount point in each availability zone__.

You can mount an EFS file system in only one VPC at a time. If you want to access it or mount it from another VPC, you have to create a __VPC peering connection__. You should note that all of these must be within the same region.

__NFS port 2049__ for EFS.

__Encryption__

1. Encryption at rest must be specified at the creation of file system. If you want to modify it later on, create a new EFS file system with encryption enabled and copy the data over.
2. Encryption at transit is supported by EFS // NFS, and must be enabled from the client side. It simply uses SSL to encrypt the connection.

__Performance mode__

1. General purpose must be used for most purposes, it has low latency, so ideal for web applications.
2. Max IO is ideal for big data and parallel connection and processing from a large number of hosts. It has higher latency but large throughput.

__Throughput mode__

1. Bursting is ideal for arbitrary large amount of data, because it scales properly.
2. But for cases with high throughput to storage ratio, such as common in web applications, provisioned mode is better.

Provisioned Throughput mode is available for applications with high throughput to storage (MiB/s per TiB) ratios, or with requirements greater than those allowed by the Bursting Throughput mode. For example, say you're using Amazon EFS for development tools, web serving, or content management applications where the amount of data in your file system is low relative to throughput demands. Your file system can now get the high levels of throughput your applications require without having to pad your file system. 

General Purpose Performance Mode

We recommend the General Purpose performance mode for the majority of your Amazon EFS file systems. General Purpose is ideal for latency-sensitive use cases, like web serving environments, content management systems, home directories, and general file serving. If you don't choose a performance mode when you create your file system, Amazon EFS selects the General Purpose mode for you by default.

Max I/O Performance Mode

File systems in the Max I/O mode can scale to higher levels of aggregate throughput and operations per second. This scaling is done with a tradeoff of slightly higher latencies for file metadata operations. Highly parallelized applications and workloads, such as big data analysis, media processing, and genomics analysis, can benefit from this mode. 

# ELB and Autoscaling

__Patching an AMI for an auto scaling group__, the procedure is —  
1. Create an image out of the main patched EC2 instance
2. Create a new launch configuration with new AMI ID
3. Update auto scaling group with new launch configuration ID. 

Note that AMI ID is set during creation of launch configuration and cannot be modified, so we have to create a new launch configuration.

__Default metric types for a load balancer__ —
1. Request count per target.
2. Average CPU utlization.
3. Network in.
4. Network out.


__Monitoring Application Load Balancers__ —
1. Cloudwatch metrics
2. Access logs
3. Request tracing
4. Cloudtrail logs.

Adding __lifecycle hooks__ to ASGs put instances in __wait state__ before termination. During this wait state, we can perform custom activities. Default wait period is 1 hour.


ASG __Dynamic Scaling Policies__ —
- Target tracking scaling. The __preferred__ one to use, this should be the first one we should consider.
- Step scaling
- Simple scaling

If you are scaling based on a utilization metric that increases or decreases proportionally to the number of instances in an Auto Scaling group, we recommend that you use target tracking scaling policies. Otherwise, we recommend that you use step scaling policies. 

The ELB service does not consume an IP address, it's the nodes that cosume one IP address each.

__Auto-scaling__ ensures —
- Fault tolerance
- Availability

__ELBs__ can manage traffic within a region and not between regions.

__For unstable scaling behavior__, that is scaling multiple times frequently, the following things can be done —

- Increasing __auto-scaling cooldown timer__ value would give scaling activity sufficient time to stabilize.
- Modify the __cloudwatch alarm period__ that triggers scaling activity.

__Default cooldown period__ is 300 seconds.

__Port based routing__ is supported by __Application Load Balancer__.

__Network Load Balancer__ can be used to __terminate TLS connections__. For this, NLB uses a security policy which consists of protocols and ciphers. The certificate used can be provided by __AWS Certificate Manager__.

__Connection draining__ enables the load balancer to complete in-flight requests made to instances that are de-registering or unhealthy. 

- AutoScaling Group termination policy —

1. Availability Zones have the most instances, and at least one instance that is not protected from scale in.
1. Oldest launch configuration.
2. Closest to next billing hour.
3. Random.

Load balancer does not create or terminate instances, that's done by auto scaling group.




# SQS

Consumers must __delete an SQS message__ manually after it has done processing the message. To delete a message, use the ReceiptHandle of a message, not the MessageId.

Incoming messages can __trigger a lambda function__.

We can use __dead letter queues__ to isolate messages that can't be processed right now.

SQS does not __encrypt__ messages by default.

**Visibility timeout** is the amount of time that a message is hidden for other consumers after a consumer reads such message. Default __visibility timeout__ for SQS is __30 seconds__. Maximum is 12 hours. Min 0 seconds.

Each __FIFO Queue__ uses —
- Message Deduplication ID 
- Message Group ID. Message Group ID helps preserve order.

For application with identical message bodies, use unique deduplication ID, while for unique message bodies, use content-based deduplication ID.

Both the default and maximum batch size for `ReceiveMessage` call of SQS is 10.

**Reducing SQS API calls** —

- Use long polling.
- Send `DeleteMessage` requests in batch using `DeleteMessageBatch`. Other batch actions are SendMessageBatch and `ChangeMessageVisibilityBatch`. 

__Message retention period__ in SQS — 1 minute to 14 days. The default is 4 days.

Limit on number of __inflight messages__ — 120,000 for standard queue and 20,000 for FIFO queue.

Amazon SQS long polling is a way to retrieve messages from your Amazon SQS queues. While the regular short polling returns immediately, even if the message queue being polled is empty, long polling doesn’t return a response until a message arrives in the message queue, or the long poll times out.

# SNS

__Available protocols for AWS SNS__ —
- HTTP // HTTPS
- Email
- Email-JSON
- SQS
- Application
- Lambda
- SMS

We can add __filter policies__ to individual subscribers in an SNS topic.

__SNS message attributes__ are —
- Name
- Type
- Value

With __Amazon SNS__, there is a possibility of the client receiving __duplicate messages__.




# API Gateway

API Gateway can __integrate with any HTTP based operations__ available on the public internet, as well as other AWS services.

__Integration types__ —
- Lambda function, can be from __another AWS account__ as well.
- HTTP
- Mock
- AWS Service
- VPC Link

For connecting API Gateway to a set of services hosted in an __on-premise network__, we can use
1. __DirectConnect__ to connect the private network to AWS directly.
2. Then use __VPCLink__ to set up API Gateway connection.


__API Gateway Throttling__ —

- __Burst limit__ refers to the first millisecond.
- __Steady-state limit__ refers to an one second interval.

__Throttling behaviors__ —
- If an user exceeds the burst limit but not the steady-state limit, the rest of the requests are throttled over the one second steady-state interval. 
- If an user exceeds the steady-state limit, AWS returns `429 Too Many Requests` error.

When it comes to throttling settings, you can __override stage settings on an individual method__ within the stage. That is, there is an option for method level throttling to override stage level throttling.

__Access control mechanisms__ for API Gateway —
- Resource policies
- AWS IAM roles and policies
- CORS or Cross-origin resource sharing
- Lambda authorizers
- Amazon Cognito user pools
- Client side SSL certs
- Usage plans

API Gateway __automatically protects the backend systems from DDoS__ attack.

__Cache properties and settings__ —
- Cache status
- Flush entire cache
- Enable API cache
- Cache capacity
- Encrypt cache data
- Cache TTL
- Require authorization
- Handle unauthorized requests

__Monitoring__ API Gateway usage — we can use __CloudWatch__ or __Access logging__. Access logging logs who accessed the API and how the caller accessed the API, CloudWatch does not include this data.

__Protect backend systems__ behind API gateway from __traffic spikes__ —

- Enable throttling.
- Enable result caching.




# Lambda

Lambda functions __can be run within a private VPC__.

Lambda can __read events from__ —
- Amazon Kinesis
- Amazon DynamoDB
- Amazon Simple Queue Service

Services that can __invoke Lambda functions__ —
- Elastic Load Balancing (Application Load Balancer)
- Amazon Cognito
- Amazon Lex
- Amazon Alexa
- Amazon API Gateway
- Amazon CloudFront (Lambda@Edge)
- Amazon Kinesis Data Firehose
- Amazon Simple Storage Service
- Amazon Simple Notification Service
- Amazon Simple Email Service
- AWS CloudFormation
- Amazon CloudWatch Logs
- Amazon CloudWatch Events
- AWS CodeCommit
- AWS Config

AWS __CodePipeline__ and AWS __OpsWorks can't invoke lambda__ functions.

__For failures__ we can configure lambda to send non-processed payloads to __SQS Dead letter queue__. Then we can configure __SNS to send a notification__ if we want. Lambda __does not have an in-built mechanism__ for notification upon failure.

A policy on a role defines which API actions can be made on the target, it does not define whether the source can access the target or not.

Each lambda function has an __ephemeral storage of 512 MB__ in the `tmp` directory.  

AWS __CloudWatch rule__ can be configured to trigger a lambda function. While configuration, the following can be used as __input to the target lambda function__ —
- Matched event
- Part of the matched event
- Constant (JSON text)

The following __CloudFront events can trigger lambda function__ —
- Viewer request
- Viewer response
- Origin request
- Origin response

__Lambda function update has eventual consistency__. Which means, for a brief window of less than a minute, it may execute either the old version or the new version.

We can use __alias versions__ to point to another version. This can enable easier upgradation from the viewpoint of a consumer.

__Limits__ —
- Function memory allocation: 128 MB to 3008 MB, in 64 MB increments.
- Function timeout: 900 seconds.
- Deployment package: 50 MB * 5 layers.
- `tmp` directory storage: 512 MB.

To grant __cross-account permission to a function__, we have to modify the function policy, not the execution role policy.

The console doesn't support directly __modifying permissions in a function policy__. You have to do it from the CLI or SDK.

If we run __lambda functions inside a VPN__, they use __subnet IPs or ENIs__. There should be sufficient ones otherwise it will get throttled.

__ENI capacity__ = Projected peak concurrent executions * (Memory in GB / 3 GB).

The __lambda console__ provides __encryption and decryption helpers__ for encryption of environment variables. 

By default, the a KMS default service key is used for encryption, which makes the information visible to anyone who has access to the lambda console. For further restriction, create a custom KMS key and use that to encrypt.

__CloudWatch metrics for Lambda__ —
- Invocations
- Errors
- Dead Letter Error
- Duration
- Throttles
- IteratorAge
- ConcurrentExecutions
- UnreservedConcurrentExecutions

We can get the __function version__ within the function using —
- `getFunctionVersion` from the Context object.
- `AWS_LAMBDA_FUNCTION_VERSION` environment variable. 

__Lambda Retry upon Failure Behavior__ —
- Event sources that aren't stream-based
    - Synchronous invocation — Returns error with __status code 200__. Includes __FunctionError__ field and __X-Amz-Function-Error__ header.
    - Asynchronous invocation — __Retry twice__, then sent to __Dead Letter Queue__.
- Poll-based and stream-based event source (Kinesis or DynamoDB) — Lambda keeps __retrying until the data expires__. The exception is __blocking__, this ensures the data are processed in order.
- Poll-based but not stream-based event source (SQS) — On unsuccessful processing or if the function times out of the message, it is __returned to the queue__, and ready for further reprocessing after the visibility timeout period. If the function errors out, it is sent to __Dead Letter Queue__.

__Lambda traffic shifting__:

- **Canary**: Traffic is shifted in two increments. You can choose from predefined canary options that specify the percentage of traffic shifted to your updated Lambda function version in the first increment and the interval, in minutes, before the remaining traffic is shifted in the second increment.
- **Linear**: Traffic is shifted in equal increments with an equal number of minutes between each increment. You can choose from predefined linear options that specify the percentage of traffic shifted in each increment and the number of minutes between each increment.
- **All-at-once**: All traffic is shifted from the original Lambda function to the updated Lambda function version at once.



# VPC

We cannot route traffic to a __NAT gateway__ or __VPC gateway__ endpoints through a __VPC peering__ connection, a __VPN connection__, or __AWS Direct Connect__. A NAT gateway or VPC gateway endpoints cannot be used by resources on the other side of these connections. Conversely, a NAT gateway // VPC gateway endpoints cannot send traffic over VPC endpoints, AWS VPN connections, Direct Connect or VPC Peering connections either.

Every route table contains a __local route__ for communication within the VPC over IPv4. We __cannot modify or delete__ these routes.

__VPC Endpoints always take precedence__ over NAT Gateways or Internet Gateways, as they are more specific routes.

**VPC Gateway Endpoints**:
- Are used via **route table entries**: they are gateway devices. **Prefix lists** for a service are used in the destination field with the gateway as the target.
- Are **HA across AZs**, but are set **per region**.
- They don't use external DNS, they use its own AWS network to resolve AWS public services.

**VPC Interface Endpoints add or replace the DNS** for the service, no route table updates are required. They are **interfaces in a specific subnet**. For **HA**, you need to add multiple interfaces, **one per AZ**. If **Private DNS Names** option is set, **it overrides the default public endpoint DNS from the service**

Network ACL __rules are evaluated in order__, starting with the lowest numbered rule. As soon as a rule matches, it is applied regardless of any higher numbered rule that may contradict it.

SSH connections are between port 22 of the host and __an ephemeral port of the client__. In fact, this is true for any TCP service.

Security groups are __stateful__, this means any connection initiated successfully will be completed.

**Security groups referencing** can be **cross-account** but **no cross-region**.

We can create __S3 proxy server__ for enabling use cases where S3 has to be accessed privately through VPN connection, AWS Direct Connect or VPC peering.

AWS __reserves 5 IPs for every subnet__, not for every VPC.

Instances in __custom VPCs don't get public DNS hosts by default__, we have to set the `enableDnsHostnames` attribute to true. The `enableDnsSupport` is to be set to true too, but that is done by default.

We can set a __custom route table as the main route table__.

We can add __secondary CIDR ranges__ to an existing VPC. When a secondary CIDR block is added to a VPC, a route for that block with target as "local" is automatically added to the route table.

__VPC peering__ connection route contains Target as `pcx-xxxxxx`.
__VPN connection__ // __Direct Connect__ connection route contains Target as `vgw-xxxxxx`.

__VPN__ is established over a __Virtual Private Gateway__.

AWS __VPC Endpoints support S3 and DynamoDB__. For __Amazon ECR__, we have to use __AWS PrivateLink__.

__Difference between DirectConnect and VPN__ — DirectConnect does not involve the Internet, while VPN does.

__AWS Direct Connect__ doesn't __encrypt in transit data__, while __VPN__ does.

To establish a __VPN connection__, we need —
- A public IP address on the customer gateway for the on-premise network.
- A virtual private gateway attached to the VPC.

To setup __AWS VPN CloudHub__ —

- Each regional site should have non overlapping IP prefixes.
- BGP ASN should be unique at each site.
- If BGP ASN are not unique, additional ALLOW-INs will be required.

The __allowed block size__ in VPC is between a /16 netmask (65,536 IP addresses) and /28 netmask (16 IP addresses).

 The following __VPC peering connection configurations__ are __not supported__ —

1. Overlapping CIDR Blocks
2. Transitive Peering
3. Edge to Edge Routing Through a Gateway or Private Connection

We can move part of our __on-premise address space to AWS__. This is called BYOIP. For this, we have to acquire a __ROA, Root Origin Authorization__ from the the regional internet registry and submit it to Amazon.

An __Internet Gateway__ performs __Static NAT__ and __handles the communication to and from the public internet__.

When creating a __default VPC__, the __CIDR block address given__ is `172.31.0.0/16`.

__Dynamic NAT Gateways__ are __scalable__.

**NACLs** operate on **layer 4**, while **security groups** use **layer 5**.

Assuming we are working with a **custom VPC**, these are the three steps needed to **enable Internet access to a subnet**:
1. Create an internet gateway and attach to a VPC
2. Add a routing rule to forward internet bound traffic to the internet gateway
3. Allocate public IPs for a subnet and enable auto-assign for resources within the subnet

Each **NAT gateway** is created in a **specific Availability Zone** and implemented with redundancy in that zone.

**Individual NAT gateways** can **handle 5 Gbps of bandwidth** and can **scale up to 45 Gbps.**

**Reserved IPs** within a **subnet**:
- `.0` - Network
- `.1` - Router
- `.2` - DNS
- `.3` - Future
- `.255` - Broadcast

When using VPC peering, you need to enable DNS settings in the Peer Connections menu to resolve public IPs to private IPs.

## IPv6
To use it, the first step is to request an IPv6 allocation. Each VPC is allocated a /56 CIDR from the AWS and this cannot be adjusted.

All IPv6 addresses used within AWS are publicly routable. Resources have public IPv6 addresses directly attached to them, unlike IPv4.

DNS names are not allocated to IPv6 addresses.

Not currently supported for VPNs, customer gateways and VPC endpoints.

## VPC Flow Logs

Are basically a `tcpdump`.

You can create flow logs for network interfaces that are created by other AWS services, such as:

- Elastic Load Balancing
- Amazon RDS
- Amazon ElastiCache
- Amazon Redshift
- Amazon WorkSpaces
- NAT gateways
- Transit gateways



# DynamoDB

AWS __DynamoDB__ is durable, ACID compliant, can go through multiple schema changes, and changes to the database does not result in any database downtime.

    Note: we think is not ACID compliant by design but can achieve ACID compliance with secondary indexes.
    Note2: ACID = Atomicity, Consistency, Isolation and Durability

__DynamoDB Global Tables__ can be used to deploy a multi region, multi AZ, fully managed database solution.

We can create __secondary indexes__ for __DynamoDB__ tables. Always choose DynamoDB when possible.

A **secondary index is a data structure that contains a subset of attributes from a table, along with an alternate key to support Query operations.** You can retrieve data from the index using a Query, in much the same way as you use Query with a table. **A table can have multiple secondary indexes**, which gives your applications access to many different query patterns.

DynamoDB supports two types of secondary indexes:

- **Global** secondary index: **An index with a partition key and a sort key that can be different from those on the base table**. A global secondary index is considered "global" because queries on the index can span all of the data in the base table, across all partitions.

- **Local** secondary index: **An index that has the same partition key as the base table, but a different sort key**. A local secondary index is "local" in the sense that every partition of a local secondary index is scoped to a base table partition that has the same partition key value.

**DynamoDB streams** can be used to **monitor changes** made to a database, and they can **trigger Lambda functions**.

We can turn on __autoscaling for DynamoDB__.

For __write heavy__ use cases in __DynamoDB__, use **partition keys with large number of distinct values**.

__DynamoDB Accelerator, DAX__ is an __in-memory cache for DynamoDB__ that reduces response time from milliseconds to microseconds.

If there are a lot of occurrences where a **shard iterator expires unexpectedly**, to increase DynamoDB table capacity to store data, we can **increase the write capacity assigned to the shard table**.

**Throughput setup** for DynamoDB can be assigned at **tables level**.

DynamoDB allows for the **storage of large text and binary objects**, but there is a **limit of 400 KB**

When reading data from DynamoDB, users can specify whether they want the read to be eventually consistent or strongly consistent:

- Eventually consistent reads (the default) – The eventual consistency option maximizes your read throughput. However, an eventually consistent read might not reflect the results of a recently completed write. All copies of data usually reach consistency within a second. Repeating a read after a short time should return the updated data.
- Strongly consistent reads — In addition to eventual consistency, DynamoDB also gives you the flexibility and control to request a strongly consistent read if your application, or an element of your application, requires it. A strongly consistent read returns a result that reflects all writes that received a successful response before the read.
- ACID transactions – DynamoDB transactions provide developers atomicity, consistency, isolation, and durability (ACID) across one or more tables within a single AWS account and region. You can use transactions when building applications that require coordinated inserts, deletes, or updates to multiple items as part of a single logical business operation.

Related to the previous item: 

- Perform strongly consistent reads of up to 24 KB per second (4 KB × 6 read capacity units).
- Perform eventually consistent reads of up to 48 KB per second (twice as much read throughput).

Valid DynamoDB Headers attributes over HTTP using the POST request method:
- x-amz-date
- host
- content-type
- x-amz-target

# ECS

Launch types —

- Fargate
- EC2

All types of instances, i.e. __on-demand, spot and reserved can be used with ECS__.

Docker containers and ECS are particularly __suited for batch job workloads__ as they can get embarassingly parallel. 

Amazon ECS enables you to __inject sensitive data into your containers__ by storing your sensitive data in either —

- AWS Secrets Manager secrets
- AWS Systems Manager Parameter Store parameters



# Elastic Beanstalk

AWS __Elastic Beanstalk__ can be used to create —

- Web application using DB
- Capacity provisioning and load balancing of websites
- Long running worker process
- Static website

It should not be used to create tasks which are run once or on a nightly basis, because the infrastructure is provisioned and will be running 24/7.

__Elastic Beanstalk__ can be used to host __Docker containers__.



# Storage Gateway

__AWS Storage Gateways__—

- **File gateway**: Flat files, stored directly on S3.
- **Volume gateway**
  - **Stored volumes**: Entire Dataset is stored on site and is asynchronously backed up to S3.
  - **Cached volumes**: Entire Dataset is stored on S3 and the most frequently accessed data is cached on site.
- **Tape gateway**: Irrelevant.



# IAM, Cognito and Directory Services

__Amazon Cognito__ has two __authentication methods__, __independent__ of one another —

- Sign in via third party federation
- Cognito user pools

__AWS Directory Service__ options:

- **AWS Managed Microsoft AD**
- **Amazon Cognito**
- **Amazon Cloud Directory** is a cloud-native, highly scalable, high-performance, multi-tenant directory service that provides web-based directories to make it easy for you to organize and manage all your application resources such as users, groups, locations, devices, and policies, and the rich relationships between them. Cloud Directory is a foundational building block for developers to create directory-based solutions easily and without having to worry about deployment, global scale, availability, and performance.
- **AD Connector** is a directory gateway with which you can redirect directory requests to your on-premises Microsoft Active Directory without caching any information in the cloud. AD Connector comes in two sizes, small and large. You can spread application loads across multiple AD Connectors to scale to your performance needs. There are no enforced user or connection limits. 
- **Simple AD** is a standalone managed directory that is powered by a Samba 4 Active Directory Compatible Server. Provides a subset of the features offered by AWS Managed Microsoft AD, including the ability to manage user accounts and group memberships, create and apply group policies, securely connect to Amazon EC2 instances, and provide Kerberos-based single sign-on (SSO). However, note that Simple AD does not support features such as trust relationships with other domains, Active Directory Administrative Center, PowerShell support, Active Directory recycle bin, group managed service accounts, and schema extensions for POSIX and Microsoft applications. It is available in two sizes:
  - Small: Supports up to 500 users (approximately 2,000 objects including users, groups, and computers).
  - Large: Supports up to 5,000 users (approximately 20,000 objects including users, groups, and computers).


There is no __default policy__ ever, anywhere. When permissions are checked, roles and policies are considered together, and in the default case there is no policy, so only the role is considered.

We can configure __IAM policies__ that allows __access to specific tags__.

__Connecting AWS SSO to On-Premise Active Directory__ —

- __Two-way trust relationship__: __Preferred__. Users can do everything from both portals.
- __AD connector__: SSO does not cache user credentials. Users can't reset password from SSO protal, have to do it from on-premise portal.

For __two-step verification__, SSO sends __code to registered email__. It can set to be either —

- Always-on
- Context-aware

__Cross-account IAM roles__ allow customers to securely grant access to AWS resources in their account to a third party.

If our identity store is not compatible with SAML, we can develop a custom application on-premise and use it with STS.

__Microsoft Active Directory__ supports __SAML__. 



# KMS and CloudHSM

AWS KMS API can be used to encrypt data.

__KMS__ master keys are region specific.

## CloudHSM

__CloudHSM backup procedure__ — Ephemeral backup key (EBK) is used to encrypt data and Persistent backup key (PBK) is used to encrypt EBK before saving it to an S3 bucket in the same region as that of AWS CloudHSM cluster.

With __AWS CloudHSM__, we can control the entire lifecycle around the keys. It helps you **meet corporate, contractual, and regulatory compliance** requirements for data security by using dedicated Hardware Security Module (HSM) instances within the AWS cloud. AWS and AWS Marketplace partners offer a variety of solutions for protecting sensitive data within the AWS platform, but for some applications and data subject to contractual or regulatory mandates for managing cryptographic keys, additional protection may be necessary.

You should consider **using AWS CloudHSM** if you require:

- Keys stored in dedicated, third-party validated hardware security modules under your exclusive control.
- FIPS 140-2 compliance.
- Integration with applications using PKCS#11, Java JCE, or Microsoft CNG interfaces.
- High-performance in-VPC cryptographic acceleration (bulk crypto).

# Kinesis

__Kinesis stream data retention period__ — 24 hours (default) to 168 hours.

For __Kinesis__, we have to use __VPC Interface Endpoint__, powered by __AWS PrivateLink__.

Amazon __Kinesis Scaling Utility__ is a __less cost-effective__ solution compared to doing it with __Cloudwatch alarms + API Gateway + Lambda function__.

__Kinesis data streams__ store the data, by default for 24 hours and upto 7 days. Whereas __Kinesis Firehose__ stream the data directly into either —

- S3
- Redshift
- Amazon Elasticsearch Service
- Splunk

Kinesis — If ShardIterator expires immediately and data is lost, we have to increase the write capacity assigned to the Shard table.



# EMR

__AWS EMR__ — AWS Elastic MapReduce, Hadoop based big data analytics.

__AWS EMR__ is preferred for __processing log files__.

__EMR__ can use __spot instances__ as underlying nodes.

We can access the underlying EC2 instances in AWS EMR cluster.


# CloudFront
- **Edge location**: location where content will be cached. Separated to an AWS Region/AZ. They are **not READ only**
- **Origin**: Can be an EC2 Instance, S3Bucket, ELB or Route53
- **Distribution**: Name given to the CDN which consists of a collection of Edge Locations.
  - **Web Distribution**: For websites
  - **RTMP**: For media streaming

We can use __signed URLs and signed cookies with Cloudfront__ to protect resources.

Slower login time and 504 errors in front of Cloudfront can be optimized by:
- Lambda @ Edge.
- Setting up an Origin Failover Policy.

Enabling __multiple domains to serve HTTPS__ over same IP address —- Generate an SSL cert with AWS Certificate Manager and create a Cloudfront distribution. Associate cert with distribution and enable Server Name Indication (SNI).


# CloudTrail
Max 5 trails per region.

We can create a __CloudTrail log across all regions__.

By default, __CloudTrail logs are encrypted__ using S3 server-side encryption (SSE). We can also choose to encrypt with AWS KMS.

# EC2

Each instance that you launch into a VPC has a tenancy attribute\. This attribute has the following values\.


| Tenancy Value | Description | 
| --- | --- | 
|  `default`  |  Your instance runs on shared hardware\.   | 
|  `dedicated`  |  Your instance runs on single\-tenant hardware\.   | 
|  `host`  | Your instance runs on a Dedicated Host, which is an isolated server with configurations that you can control\. | 

After you launch an instance, there are some limitations to changing its tenancy\.
+ You cannot change the tenancy of an instance from `default` to `dedicated` or `host` after you've launched it\.
+ You cannot change the tenancy of an instance from `dedicated` or `host` to `default` after you've launched it\.

You can change the tenancy of an instance from `dedicated` to `host`, or from `host` to `dedicated` after you've launched it\.

Each VPC has a related instance tenancy attribute\. This attribute has the following values\.


| Tenancy Value | Description | 
| --- | --- | 
|  `default`  |  An instance launched into the VPC runs on shared hardware by default, unless you explicitly specify a different tenancy during instance launch\.  | 
|  `dedicated`  |  An instance launched into the VPC is a Dedicated Instance by default, unless you explicitly specify a tenancy of `host` during instance launch\. You cannot specify a tenancy of `default` during instance launch\.  | 

You can change the instance tenancy of a VPC from `dedicated` to `default` after you create it\. You cannot change the instance tenancy of a VPC to `dedicated`\.

# Misc

For **disaster recovery** we can assume that an **entire region** fails.

AWS STS — The __policy of the temporary credentials__ generated by STS are defined by the intersection of your IAM user policies and the policy that you pass as argument.

AWS __VM Import__ // Export can be used to transfer virtual machines from local infrastructure to AWS and vice-versa.

AWS __Trusted Advisor__ is a resource that helps users with cost management, performance and security.

__CloudFormation Drift Detection__ can be used to detect changes in the environment. Drift Detection only __checks property values which are explicitly set__ by stack templates or by specifying template parameters. It does not determine drift for property values which are set by default.

__AWS Server Migration Service (SMS)__ is an agentless service which makes it easier and faster for you to migrate thousands of on-premise workloads to AWS.

__AWS Athena__ is a managed service which can be used to make interactive __search queries to S3 data__.

AWS __Athena pricing__ is based upon per query and amount of data scanned in each query. To __reduce price__ —
- Partition data based on different parameters so that amount of data scanned gets reduced.
- Create separate workgroups based upon user groups.

__AWS Athena is simpler__ and requires less effort to set up __than AWS Quicksight__.

__Amazon Inspector__ is a security assessment service, which helps improve security and compliance of applications.

__AWS Opsworks__ is a configuration management service for Chef and Puppet. With __Opsworks Stacks__, we can model our application as __a stack containing different layers__.

__Amazon ECS for Kubernetes (EKS)__ exists, it's a managed service.

Changes to __CloudTrail global service event logs__ can only be done via the CLI or the SDKs, not the console.

For __CloudFront query string__ forwarding, the parameter names and values used are __case sensitive__.

__AWS Polly__ — Lexicons are specific to a region. For a single text appearing multiple times, we can create alias using multiple Lexicons.

Amazon __Quicksight__ is a managed service for __creating dashboards__ with data visualization.

__AWS CloudSearch__ helps us add search to our website or application. __Like Elasticsearch__.

__AWS Glue__ is a fully __managed ETL service__ for data. It __keeps a track of processed data using Job Bookmark__. Enabling Job Bookmark will help to __scan only changes since last bookmark__ and prevent processing of whole data again.

__AWS X-Ray__ — Helps debug and __analyze microservices architecture__.

__Reducing cost with AWS X-Ray__ — Sampling at a lower rate.

__Amazon WorkDocs__ has a __poweruser__ facility, which on enabling restricts sharing of documents to that user only.

__AWS Data Pipeline__ can automate the movement and transformation of data for data-driven workflows. For example, transferring older data to S3 from DynamoDB.


__Disaster recovery solutions__ —
- Backup and Restore. Cheapest.
- **Term pilot** light is often used to describe a DR scenario in which a minimal version of an environment is always running in the cloud. The idea of the pilot light is an analogy that comes from the gas heater. In a gas heater, a small flame that’s always on can quickly ignite the entire furnace to heat up a house. This scenario is similar to a backup-and-restore scenario.
- Warm Standby
- Multi-Site
- Multiple AWS Regions. Costliest.

With __AWS Config__, we can get a snapshot of the current configuration of our AWS account.

For __queue based processing__, **scaling** EC2 instances based on the **size of the queue** is a preferred architecture.

It's best practice to launch Amazon __RDS instance outside an Elastic Beanstalk environment__.

__RI Coverage Budget__ reports number of instances that are part of Reserved Instance. For an organisation using default IAM policy, each member account owner needs to create a budget policy for individual accounts and not by master account.

__Consolidated Billing__ in AWS Organisations combines usage from all accounts and billing is generated based upon total usage. Services like __EC2 and S3 have volume pricing tiers__ where with more usage volume the overall charge decreases.

To automatically trigger __CodePipeline__ with changes in source __S3__ bucket, use __CloudWatch Events rule__ and __CloudTrail trail__.

__Amazon Data Lifecycle Manager__ can be used for creation, retention and deletion of EBS snapshots.

With __AWS Organizations__, we can centrally manage policies across multiple AWS accounts. With __Service Control Policies (SCPs)__, we can ensure security policies are in place.

In __AWS Managed Blockchain network__, the format for __resource endpoint__ is — `ResourceID.MemberID.NetworkID.managedblockchain.us-east-1.amazonaws.com:PortNumber`.

When you want to keep your expenditure within a budget, use __AWS Budgets__, not AWS Cost Explorer.

__Cloudwatch monitoring schemes__ —

- Basic. 5 minutes.
- Detailed. 1 minute.
- Custom. Can be down to 1 second.

__Transferring data__ from an EC2 instance to Amazon S3, Amazon Glacier, Amazon DynamoDB, Amazon SES, Amazon SQS, or Amazon SimpleDB __in the same AWS Region has no cost at all__.

__Amazon MQ__ is a message queue which supports industry standard messaging protocols. Good fit for migrating existing application.

__AWS Shield__ is a service that protects resources against DDoS attacks to EC2, ELB, Cloudfront and Route53.

__AWS IoT Core__ is a managed service that lets IoT devices connect and interact with AWS applications and resources.

The following storage have __encryption at rest by default__ —

- AWS Glacier
- Storage Gateway in S3

__Perfect Forward Secrecy__ is supported by —

- Cloudfront
- Elastic Load Balancing

Classic Load Balancer does not support __SNI__, we have to use Application Load Balancer or Cloudfront.

The following services enable us to __run SQL queries directly against S3 data__ —

- AWS Athena
- Redshift Spectrum
- S3 Select

By default, each workflow execution can run for a __maximum of 1 year__ in Amazon SWF. 

In __AWS SWF__, a __decision task__ tells the decider the state of the workflow execution.

**SWF** is a **fully-managed state tracker and task coordinator service**. It does **not provide serverless orchestration to multiple AWS resources**. It ensures that a **task is never duplicated** and is **assigned only once**.

__Third party SSL cert__ can be imported into —


- AWS Certificate Manager
- IAM Certificate Store

**AMIs** cannot be *shared*.

To **decouple** architecture:
- SWF
- SQS

Amazon Cognito service is primarily used for user authentication and not for providing access to your AWS resources

AWS SSO service uses STS, it does not issue short-lived credentials by itself. AWS Single Sign-On (SSO) is a cloud SSO service that makes it easy to centrally manage SSO access to multiple AWS accounts and business applications.


**To give Aurora MySQL access to Lambda**

Create an AWS Identity and Access Management (IAM) policy that provides the permissions that allow your Aurora MySQL DB cluster to invoke Lambda functions. For instructions, see Creating an IAM Policy to Access AWS Lambda Resources.

1. Create an IAM role, and attach the IAM policy you created in Creating an IAM Policy to Access AWS Lambda Resources to the new IAM role. Set the aws_default_lambda_role DB cluster parameter to the Amazon Resource Name (ARN) of the new IAM role. If the cluster is part of an Aurora global database, apply the same setting for each Aurora cluster in the global database.

2. To permit database users in an Aurora MySQL DB cluster to invoke Lambda functions, associate the role that you created.

3. Configure your Aurora MySQL DB cluster to allow outbound connections to Lambda. If the cluster is part of an Aurora global database, enable outbound connections for each Aurora cluster in the global database.

**Amazon Macie** uses **Machine Learning to protect sensitive data**

Services for web servers **stateless**:
- RDS
- DynamoDB
- Elasticache

When editing permissions (policies and ACLs), to whom does the concept of the "Owner" refer?  The "Owner" refers to the identity and email address used to create the AWS account.

SQS uses multiple hosts, and each host holds only a portion of all the messages. When a staff member calls for their next message, the consumer process does not see all the hosts or all the messages. As such, messages are not necessarily delivered in the order in which they were generated.

AWS services allow native encryption of data (AWS Key Management Service (KMS) or, in some cases, using customer provided keys), while at rest:
- EFS
- EBS
- S3
- Elasticache for **Redis** (NOT for Memcached)

AWS WAF conditions:
- IP Match
- String Match
- Size Constraint


Once a VPC is set to Dedicated hosting, it can be changed back to default hosting via the CLI, SDK or API. Note that this will not change hosting settings for existing instances, only future ones. Existing instances can be changed via CLI, SDK or API but need to be in a stopped state to do so 

With NAT instances, the most common oversight is forgetting to disable Source/Destination Checks

For **file operations**, use **EFS**.

**Elastic Fabric Adapter (EFA)** is a network device that you can attach to your Amazon EC2 instance to **accelerate High Performance Computing (HPC) and machine learning applications**. EFA enables you to achieve the application performance of an on-premises HPC cluster, with the scalability, flexibility, and elasticity provided by the AWS Cloud. **OS-bypass capabilities of the Elastic Fabric Adapter (EFA) are not supported on Windows instances.** Although you can attach EFA to your Windows instances, this will just act as a regular Elastic Network Adapter

**AWS Global Accelerator** is a service that improves the availability and performance of your applications with local or global users. It provides **static IP addresses** that act as a fixed entry point to your application endpoints in a single or multiple AWS Regions, such as your Application Load Balancers, Network Load Balancers or Amazon EC2 instances.

**Amazon FSx** have full support for the SMB protocol and Windows NTFS, Active Directory (AD) integration, and Distributed File System (DFS). 

Because overlay multicast is a method of building IP level multicast across a network fabric supporting unicast IP routing, such as Amazon Virtual Private Cloud (Amazon VPC).

AWS hosts a variety of **public datasets** that anyone can access **for free**.

Amazon FSx for Lustre provides a high-performance file system optimized for fast processing of workloads such as machine learning, high performance computing (HPC), video processing, financial modeling, and electronic design automation (EDA). These workloads commonly require data to be presented via a fast and scalable file system interface, and typically have data sets stored on long-term data stores like Amazon S3.