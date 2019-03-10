<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [AWS Developer: The Big Picture](#aws-developer-the-big-picture)
  - [What is AWS?](#what-is-aws)
    - [Global Infrastructure](#global-infrastructure)
    - [How Does AWS Work?](#how-does-aws-work)
  - [Core Services](#core-services)
    - [Elastic Cloud Compute (EC2)](#elastic-cloud-compute-ec2)
      - [Create an instance (AWS Console)](#create-an-instance-aws-console)
      - [Amazon EC2 Pricing](#amazon-ec2-pricing)
    - [Simple Storage Service (S3)](#simple-storage-service-s3)
    - [Relational Database Service (RDS)](#relational-database-service-rds)
    - [Route53](#route53)
  - [Enhancing Your App with AWS Databases and Application Services](#enhancing-your-app-with-aws-databases-and-application-services)
    - [Elastic Beanstalk](#elastic-beanstalk)
    - [DynamoDB](#dynamodb)
    - [RedShift](#redshift)
    - [Virtual Private Cloud](#virtual-private-cloud)
    - [CloudWatch](#cloudwatch)
    - [CloudFront](#cloudfront)
  - [Harnessing the Power of AWS from the Command Line to Code](#harnessing-the-power-of-aws-from-the-command-line-to-code)
    - [Web Console](#web-console)
    - [Software Development Kits (SDK)](#software-development-kits-sdk)
    - [Command Line Interface (CLI)](#command-line-interface-cli)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# AWS Developer: The Big Picture

> My notes from [Pluralsight course](https://app.pluralsight.com/library/courses/aws-developer-big-picture/table-of-contents)

## What is AWS?

- Collection of cloud computing services
- Can work together or independently
- Run or support a computer program
- Services run in data centers around the globe
- Services provide some sort of computing operation (eg: computing, messaging, storing, routing, etc)
- Typically used for web app, but could be used for any type of app

### Global Infrastructure

- With AWS, gain ability to deploy to different global locations
- Benefits: reduce latency caused by geographic distance - have servers closer to your users, increased redundancy

**AWS Regions & Availability Zones**

AWS has many _regions_ around the world - physical location where services are hosted.

![regions](images/regions.png 'regions')

Within each region has _availability zones_ - collection of data centers that have separate power, networking and connectivity, connected to eachother via fibre optics. Extremely fault tolerant. If one data center goes down, the others provide redundant failover.

![availability zone](images/availability-zone.png 'availability zone')

Scaling across regions and availablility zones increases redundancy.

[AWS Health Status](https://status.aws.amazon.com/)

### How Does AWS Work?

- Most services interact with eachother over TCP connections
- Depending on service, could be over a different port (eg: particular DB may listen on particular port)
- Typically your app initiates a connection to one or more of these services
- If all services are in same VPC, will have local IP addresses, fast connections
- Can add services as needed, eg: database, S3 etc

  ![add services](images/availability-zone.png 'add services')

- Once main application is moved to the cloud, it must be secured
- Connectiond and permissions between each service are managed by _Security Groups_ - light firewalls around each server instance
- Controlling access and security is done via configuration with AWS web console, many service interaction bugs due to misconfigured security groups.

  ![security group](images/security-group.png 'security group')

## Core Services

### Elastic Cloud Compute (EC2)

- Foundational building block for many apps
- Basis for many other services
- Instance is basically a computer, use it to do any sort of computing such as...
- Run applications
- Virtual desktop
- Can have 3rd party software installed
- Elastic: Instances running computing operations can increase or decrease at will, scaling can happen automatically
- Basic building block: EC2 Instance - virtual server, OS agnostic

#### Create an instance (AWS Console)

Need to make decisions about each configurable option when creating an instance:

**STEP 1: Choose Amazon Machine Image (AMI)**

Combination of OS (eg: what flavour of Linux such asRed Hat, Suse, Ubuntu, etc), and some pre-installed software (eg: Java, Python, AWS cli tools). Amazon updates the image software (eg: patch security vulnerabilities).

CAVEAT: Amazon updates the image software... NOT YOUR INSTANCE. To get the updates, need to create a new instance based off new image, then migrate apps from old to new instance.

Can use images provided by Amazon, or search marketplace for other providers.

When using an open source image, you only pay for hardware its running on.

**STEP 2: Choose an Instance Type**

- Specs for instance including...
- Number of CPUs
- Amount of RAM
- Network performance

Types are categorized into _Families_ - easy profile to decide right image type per use case. eg - for running a file server, choose `Storage optimized` family. For ML, choose `Compute optimized`.

Within each family are _types_ - eg small, medium. large, extra large.

![image type compare](images/image-type-compare.png 'image type compare')

**STEP 3: Configure Instance Details**

- Security roles
- Further fine grained configuration such as networking
- Number of instances (of same type and image) to be replicated with this same image and type
- AWS creates _Auto Scaling Group_ - increase or decrease number of instances according to rules you configure
- Auto Scaling Groups can be modified after creation
- Also Elastic Beanstalk easiest to use for auto scaling an app, more on this further in course

**STEP 4: Add Storage**

- _Elastic Block Storage (EBS)_ - storage service, easy to reason about and calculate charges for storage
- NOT the same as simple storage service (covered further in course)
- Volumes are independent of EC2 instances
- Volumes can be retained or deleted when EC2 instance is interminated
- Use EBS for EC2 file systems
- Can select root volume size

**STEP 5: Tag Instance**

- Meta values you can add to instance for your own purposes

**STEP 6: Configure Security Group**

- IP-based communication rules for a single or group or service instances
- Controls which IP addresses an instance can talk to and which IP's can talk to it
- eg: Control who can SSH into EC2 instance, do not leave it open to ssh from anywhere, only allow incoming ssh session from current IP address
- eg: Allow access between EC2 instances
- eg: Allow access to Databases
- eg: Accept HTTP requests on ports 80/443

**STEP 7: Review Instance Launch**

- Create instance with existing key pair, needed to ssh into instance

#### Amazon EC2 Pricing

- EC2 instances are charted by the hour
- Price based on: instance type, AMI type
- Some images cost more (eg: Windows image costs more than Linux because windows image must be licensed)
- Only pay for what you use
- Start/Stop instance, will only pay for when instance was alive
- EBS has separate pricing
- Load balancer costs extra

### Simple Storage Service (S3)

- Stores files, any type
- Max file size is 5 terabytes
- Largest file upload in a single PUT is 5GB
- _Bucket_ - foundational structure, root resource to which you can add/delete/modify objects
- Buckets can be configured with permissions, hosting options, logging
- Buckets can be configured to trigger events when objects added/modified/deleted
- Can preserve older versions of objects
- Can auto replicate ojects across regions -> solve latency
- A better way to solve latency with S3 is to use another Amazon service - CloudFront - cache content (aka edging content )
- S3 buckets are accessed via URL to access objects contained within, eg: `https://s3-us-west-1.amazonaws.com/okfido.org/img/okfido_logo.png`
- With permissions set to allow anonymous access, can use S3 to host static files for websites -> very low cost
- Bucket can interact with Route 53 url's

![s3 bucket url](images/s3-bucket-url.png 's3 bucket url')

**S3 Pricing Structure**

Pricing based on...

- Amount of data stored
- Number of requestsd
- Amount of data transferred

Prices differ per region, get cheaper per unit as volume goes up.

### Relational Database Service (RDS)

- Collection of AWS services for managed relational databases
- Managed - AWS takes care of backups, software updates, infrastructure (easier than installing a database on an EC2 instance yourself)
- Easy configuration via webconsole
- Ability to easily create read replicas
- Offerings include: MySQL, PostgreSQL, SQLServer, MariaDB, Oracle, Amazon Aurora (pricing differs per db vendor)
- Also pick what type of EC2 instance DB will run on
- Take DB snapshots
- Change hardware
- Pricing depends on type of db, region and EC2 instance type
- For non-relational db, DynamoDB (NoSQL) or RedShift (Data Warehouse)

**Database Security**

- Access to RDS controlled via Security Group
- eg: Allow your EC2 instance running your app access to RDS but block all other external apps

![rds](images/rds.png 'rds')

### Route53

- Amazon service for DNS management (Domain Name System) inside and outside AWS
- Core to letting users interact with services in AWS
- Configure domain names (urls) to resolve to AWS services
- Use domain name you already own or register new one via AWS
- DNS: System to translate human-readable URL to IP address
- EC2 instances can be configured with public IP address, but other resources like S3 buckets, load balancer don't have static visible IP addresses
- Route53 allows to setup url resolution to AWS resources directly, bypassing need to know a particular IP address

![route 53](images/route53.png 'route 53')

**Setup**

- Setup _Hosted Zone_ - root domain name eg: `example.com`
- Use Hosted Zone to setup subdomains, eg: `www.example.com`, `mail.example.com` and configure these to route to AWS resources
- Create _Record Set_ - A, CNAME, MX etc. for subdomains
- Also have Route53 Health Check - setup regular health checks for a given url path, setup alerts, eg: if url request receives 503 or 404 response

## Enhancing Your App with AWS Databases and Application Services

### Elastic Beanstalk

- Makes it easy to run app code and scale
- Runs code on EC2 instances plus adds conveniences
- Otherwise, using EC2 directly requires manual configuration, manual code deployment, restricted command line interface, scaling with AMIs, manual monitoring
- Elastic Beanstalk does all this for you
- Easy deployment via web console, AWS CLI or SDK
- Set it and forget it configuration
- Aggregated Monitoring and Logging across multiple EC2 instances
- Main abstract structure is _Application_ - root level organization for web app or service, needs unique name, eg: `My Application`
- Can have multiple app versions inside an Application, this is the actual application code that's running
- App Version can eb deployed to an _Environment_ - rules and configuration that manage actual EC2 instances (eg: Test env, Prod env)
- Each environment can run with a different platform (eg: Java, Node) and be configured with certain EC2 instance types
- Will spend the most type configuring each environment - setup deployment, load balancing and scaling rules

![elastic beanstalk structure](images/elastic-beanstalk-structure.png 'elastic beanstalk structure')

- Application versions are stored in S3
- Max number of application versions is 500
- Comes with monitoring dashboard with aggregated metrics from all EC2 instances, eg: number of requests, CPU utilization, network traffics
- Logs dashboard, no need to ssh to individual instances to download logs
- Free service, you only pay for EC2 instances, load balancers, and S3 storage being used

### DynamoDB

- Managed NoSQL database from AWS
- Supports both document, and key/value store models
- Scalable and flexible
- Very different from RDS
- Unlimited, elastic storage
- No hardware choices
- Pay only for what you store and how often its queried
- Core structure is _Table_ - root point of data storage
- Each table requires a _Primary Key_ for partitioning and indexing data
- Optionally table can also have _Secondary Index_
- Most important aspect of table: _Provisioned Throughput Capacity_ - amount of reads or writes per second to provision table with
- Each read/write is limited to 4KB
- Any request > 4KB will consume extra read/write units
- Pricing (differs per region) depends on Provisioned Throughput Capacity and Amount of Data Stored

### RedShift

- Data warehousing solution
- Data Pipeline (ETL process) to move data from DynamoDB tables, RDS databases and S3 Buckets to RedShift for analysis and processing data
- Basic structure is _Cluster_ - collection of one or more nodes, virtual machine instances, that are hardware running data warehouse
- Nodes have _types_ (like EC2) with different combinations of cpu, memory, storage, network performance
- Security options - encrypt entire data warehouse, setup in virtual private cloud (VPC), no public IP
- Only pay for what you use
- _Dense Compute_ nodes - cheaper, less storage
- _Dense Storage_ nodes - expensive, more storage

### Virtual Private Cloud

- Secure resources into groups that follow access rules and share logical space
- Commonly used when launching EC2 instances to secure and control access
- Security Groups secure single instaces whereas VPCs secure groups of instances
- Configure VPC routing tables
- Use NAT gateways for outbound traffic
- Internal IP Address allocation
- Contains one or more _Subnets_ - group resources and assign rules to each
- Primary purpose of having multiple subnet within VPC - setup both public (access to internet, use security groups) and private (databases, application servers, no access to internet) subnets
- Private subnet may use NAT gateway in public subnet to securely access internet
- Could launch EC2 instance in public subnet as _tunnel_ to ssh into private EC2 instances

![vpc](images/vpc.png 'vpc')

**Virtual Private Cloud Security**

- Control routing with _Routing Table_ and _Network ACL (Access Control List)_
- Route table - override certain IP ranges and redirect traffic elsewhere, eg: direct all outgoing traffic to NAT gateway to filter traffic and mask the instances IP address
- Network ACL - subnet level firewall - allow/disallow IP address ranges for incoming and outgoing connections
- Basic VPC Configuration is Free!

![vpc security](images/vpc-security.png 'vpc security')

### CloudWatch

- Monitoring service, integrated into most other AWS services for monitoring and alarms
- Setup alarms choosing from pre-existing set of metrics for each service, eg: EC2 CPUUtilization, Dynamo DB ConsumedReadCapacityUnites, S3 NumberOfObjects, Route53 HealthCheckStatus, RedShift DatabaseConnections
- Set threshold for alarm and actions to take - eg: notify via email or SMS, triger auto scale action
- Can also consume/aggregate/monitor logs - install & configure `awslogs` agent on EC2 instances
- Tell it which logs to send to CloudWatch, configure to filter logs and send alerts on user defined criteria, eg: how many times a specific exception occurs in logs and send notification if it occurs a set number of times

![cloud watch](images/cloud-watch.png 'cloud watch')

### CloudFront

- CDN (content delivery network) - serve files globally with fast connections
- Works with S3, EC2, AWS Load Balancer, Route53, to serve content from location closest to incoming requests
- Easiest to setup of all services
- Create a _Distribution_ - set of content to be served from CloudFront
- For each distribution, specify original location for content (eg: S3 bucket)
- Unique CloudFront url assigned to distribution, eg `https://d3nwl6hikok169.cloudfront.net`, use this to access content
- Configuration options include - Allowed HTTP Methods, Edge Locations, SSL Certificates
- Pricing is convoluted

## Harnessing the Power of AWS from the Command Line to Code

### Web Console

- First point of interaction with AWS
- Similar services grouped together to build a taxonomy
- Compute: EC2, Elastic Beanstalk (services to run your app on a virtual machine)
- Storage & CDN: S3, CloudFront
- Database: RDS, DynamoDB, Redshift
- Networking: Route53, VPC
- DeveloperTools
- Management Tools: CloudWatch
- Security & Identity

- AWS dropdown - select any _Resource Groups_ you've created - group resources according to function
- Services dropdown - same as on front page but in alpha order
- Drag frequently used services to menu bar
- Some resources not visible to each other across regions

### Software Development Kits (SDK)

- Code libraries that shorten or simplify complex operations
- Makes it easier for your code to connect to AWS
- Supported SDK languages include: Android, IOS, Node.js, JavaScript, Java, PHP, Ruby, Rails, Go, .NET C++, Unity
- Projects hosted on [Github](https://github.com/aws)
- Most AWS requests can be accessed via HTTP, therefore most SDK's are language wrappers around HTTP Request
- Installation depends on your langauge of choice, eg: `npm install aws-sdk`, `gem install aws-sdk`, etc.

Example, Node.js program to insert data into DynamoDB:

```javascript
const AWS = require('aws-sdk');
AWS.config.update({ region: 'us-east-1' });
const dynamodb = new AWS.DynamoDB();
const item = {
  Item: {
    id: {
      S: new Date().getTime().toString()
    },
    value: {
      S: process.argv[2]
    }
  },
  TableName: 'aws-developer'
};
dynamodb.putItem(item, (err, res) => {
  if (err) throw err;
});
```

To run it:

```shell
$ node index.js "pizza or death"
```

[Docs](https://docs.aws.amazon.com/#lang/en_us)

### Command Line Interface (CLI)

- Unified tool to interact with AWS resources from command line
- Useful in shell scripts or batch files
- Good for automated tasks
- eg: automated build system to deploy built app artifacts to cloud
- `aws <service> <command> <arguments>`
- On Mac, use `pip` to install
- First time run `aws configure`, then provide AWS Access Key ID, AWS Secret Access Key and Default region name (add/remove access keys in security credentials section of console)

Eg: insert item in db:

```shell
aws dynamodb put-item --table-name aws-developer --item "{...}"
```

[Docs](https://docs.aws.amazon.com/cli/latest/reference/)
