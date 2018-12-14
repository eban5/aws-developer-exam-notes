# AWS Certified Developer Associate June 2018 Exam Study Guide / Notes

|Domain|% of Exam
|--|--|
|Deployment|22%|
|Security|26%|
|Development with AWS Services|30%|
|Refactoring|10%|
|Monitoring and Troubleshooting|12%|

* 130 mins
* 65 questions: Multiple Choice / Multiple Response
* minimum passing score 720/1000
* Valid for 2 years
* Scenario based questions

----

## EC2

Provides resizable compute capacity in the cloud. Reduces the time required to obtain and boot new server instances to minutes, scale capabity up and down as requirements change.

* On-Demand - fixed rate by the hour. no commitment
* Reserved - capaticy reservation at significant discount.
* Spot - flexible start and end times, enables you to bid whatever price you want for instance capacity,  only feasible at very low compute prices. users with urgent need for large amount of additional computing capacity
    * If AWS kills your spot instance, you aren't charged for the hour. Only charged if you kill the instance.
* Dedicated Hosts - useful for regulatory requirements. can be on-demand. existing software licenses.

## EBS 

create storage volumes and attach them to EC2 instances. 

General Purpose SSD - GP2 - for less than 10,000 IOPS
Provision IOPS SSD - desciened for i/o intensive apps.

Throughput Opimizied HDD
Cold HDD
Magnetic


_always use roles_

## IAM 
### overlap with S3

Manage users and their level of access to AWS Console

* **Users** = end users (people)
* **Groups** = collection of users under one set of permissions
* **Roles** - create roles and then assign them to AWS resources
* **Policies** - a document that defines one or more permissions

IAM is universal, not region-by-region.
"root" has complete Admin access
New users have ***no*** permissions when first created.
Access key id / secret key ≠ aws mgmt console login
Always setup MFA

Least Priveleges

Create roles and apply them to EC2 instances to interact with AWS services on your behalf. Better than hardcoding secret keys into the instance.

Roles allow you to not use Acces Key ID's and Secret Access Keys
* Roles are preferred from a security perspective _exam question_
* Roles controlled by policies - JSON - key pairs
* Changes to role policies take affect immediately
* Attach/detach roles to running EC2 instances without having to stop/terminate them. *This is new.*

## RDS

RDS - OLTP
- SQL
- MySQL
- PostgresQL
- Oracle
- Aurora
- MariaDB

DynamoDB - NoSQL
RedShirt - OLAP

Database:
* collection - table
* document - row
* key value pairs - fields

**JSON / NoSQL**

In non relational DBs, you don't need to know how many columns ahead of time. 

Data Warehousing

Used for business intelligence. Pull in very large and complex datasets. ex. can query current performance vs targets. Production DB could push to data warehouse and that is where those complex querying happens.

**OLTP vs OLAP**

- Online Transaction Processing - online store, someone places order 20102, pulls up a row of data such as Name, Date, Address for delivery, etc.
    - "insert into ***this*** ***table***, ***this data***"
- Online Analytics Process - pulls in large number of records for complex querying. Infrequent
    - ex. net profit for EMEA and Pacific for the Digital Radio Product. 

## ElasticCache

in-memory cache in the cloud.
improve performance of webapps by allowing you to retrieve information from fast, managed, in-memory caches, instead of relying entirely on slower disk-based databases.

supported engines: Memcached and Redis.
good for frequently accessed information to reduce load on production databases. Improve latency.

__exam question__: if you need multi-az use Redis. If you aren't concerned about redundancy use Memcached.

Different in practice. 


ElastiCache manages Memcached nodes as a pool that can grow/shrink. Individual nodes are expendable.

Memcached: 
* *object caching* is primary goal
* simple
* scale horizontally as you go

Redis:
* advanced data types, lists hashes, and sets.
* ***Does sorting and ranking datasets in memory (e.g., leaderboard)***
* run in multi-azs.
* pub/sub capability

If read-heavy DB is under stress use Elasticache.
* Redshift good if OLAP transactions.

### RDS Lab
RDS Security Group - allow inbound from security group where web server is located over port 3306.

EC2 instance in one security group.
RDS instance in another security group. 
Something is not connecting, open up port 3306 to the sec group the RDS group is in. 

### RDS: Backups, Multi-AZs, and Read Replicas

Backups:
* Automated - recover to any point in time within retention period (1-35 days). Will take full daily snapshot and transaction logs. AWS will first choose the most recent dail back up and apply transaction logs relevany to the day. Point in time recovery.
    * enabled by default
    * backup data stored in S3, free storage space equal to size of your db. RDS 10GB == 10GB worth of storage free.
    * backups taken within defined window. during window i/o may suspend, elevated latency.
    * deleted when rds instance is deleted.
* Database Snapshots
    * manually done. stored even after original db is deleted.

Restoring from snapshot makes new DNS endpoint. Completely separate DBs.

Encryption - at rest supported for all. Done with AWS KMS. Data stored at rest in underlying storage is encrypted, as are its automated backups, read replicas, and snapshots.
* today - can't encrypt existing DB instance. Must create copy and encypt copy. __exam question__

**Multi-AZ** - ***for disaster recovery only***. not for performance improvement (you use read replicas for that). Writes to us-east-1a are replicated to replica at us-east-1b. 

You never deal with IP addresses when dealing with RDS instances, instead you deal with DNS names for endpoints so that Amazon can point the DNS at whatever instance it needs to for availbility mechanisms to work.

RDS will automatically failover to the standby so db operations resume quickly in the event of DB maintenance, db instance failover, AZ failure.

**Read Replicas** - ***use for scaling***. writes to db are copied out to other replicas for reading only. frequent read queries get sent to read replicas. scale out db reads to lessed load on production db. asynch replication. Can have up to 5 replicas. Replicas can be promoted to their own databases (breaks replication).


## EC2 Exam Tips

- Pricing models: On-demand, Reserved, Spot, Dedicated Hosts.
- If a Spot instance is terminated by Amazon EC2 you will not be charged for a partial hour of usage. However, if you terminate the instance yourself, you will be chaged for the complete hour in which the instance ran.
- FIGHT DR MC PX - all instance types
- EBS Types:
    - SSD:
        - General Purpose SSD: balances price and performance for a wide variety of workloads
        - Provisioned IOPS - high performance, for mission critical, low latency ops.
    - Magnetic:
        - Throughput Optimizied: low cost HDD designed for frequently accessed, throughput intensive workloads
        - Cold HDD - low cost HDD designed for less frequently accessed workloads. Can't be boot volume
        - Magnetic - older. can be boot volume.
- ELBs:
    - Application
    - Network - expensive, millions of requests per second. think netflix.
    - Classic
    - Error 504 means gateway timeout. application not responding within idle timeout period. X-forwarded header sends IP address of your end user forward to the instance, otherwise it only sees private ip address of elb.
- Route53: DNS service, map domain names to S3 EC2 or ELBs.
- CLI: Least Privilege, create groups for users and apply policies to groups.
    - Secret Access Key:
    - Don't use just 1 access key to share across developers. Make one per developer. Safety to delete just one key if needed.
    - Roles allow you to not use Access Key ID's and Secret Access Keys.
    - Roles are preferred from security perspective.

Practice Quiz incorrect:
    HTTP 4xx worded strange. Client-side issue.
    Amazon ECS description
    Individual instances are provisioned in AZs.
    No encryption on the fly.

# Route53

* Simple routing policy – Use for a single resource that performs a given function for your domain, for example, a web server that serves content for the example.com website.
* Failover routing policy – Use when you want to configure active-passive failover.
* Geolocation routing policy – Use when you want to route traffic based on the location of your users.
* Geoproximity routing policy – Use when you want to route traffic based on the location of your resources and, optionally, shift traffic from resources in one location to resources in another.
* Latency routing policy – Use when you have resources in multiple AWS Regions and you want to route traffic to the region that provides the best latency.
* Multivalue answer routing policy – Use when you want Route 53 to respond to DNS queries with up to eight healthy records selected at random.
* Weighted routing policy – Use to route traffic to multiple resources in proportions that you specify.

Topics
----

# S3

- Object based (key, value)
- Data is stored in **buckets**. 
- Unlimited storage.
- S3 is a universal namespace, names must be unique globally. like an internet address.

Core Fundamentals:
- Key (name)
- Value (data)
- Version ID
- Metadata
- Subresource - used to manage bucket specific configuration
    - Bucket Policies, Access Control Lists (ACLs)
    - Cross Original Resource Sharing (CORS)
    - Transfer Acceleration

Data consistency
* **read after write** consistency for PUTS of *new* objects. can access file straight away.
* **eventual consistency** for overwrite PUTS and DELETES. can take time to propagate.

**Availability**: 99.99% for S3 platform. Up time, time service is available. Amazon guarantees 99.9%.
**Durability**: 11 x 9s. Amount of data you can't afford to lose in a year so you want that to be close to 100%. Designed to sustain the loss of 2 facilities concurrently.

Storage Classes:
- S3 regular: durable, imeediately available, freq access
- IA: Infrequent Access. Durable. Lower fee but charged for each retrieval.
- One Zone IA: Cost is 20% less than regular IA. Don't have same level of availability.
- Reduced Redundancy Storage:  designed for data that can be recreated if lost—e.g., like thumbnail, they can be recreated later.
- Glacier - very cheap but for infrequent archival storage. 3-5 hour for data access.

Charged for:
- Storage per GB
- Requests (get, put, copy)
- Storage Management Pricing - inventory, analytics, and object tags
- Data Management Pricing - data transferred out of S3
- Transfer Acceleration - uses CloudFront to accelerate.

Max filesize transferred via PUT request - 5 GB.
To achieve better performance, add hex hash to prefix.
Key name determines which partition it will store file on.
If you want to enable a user to download your private data directly from S3, you can insert a pre-signed URL into a web page before giving it to your user.


### S3 Security

Allowing access to buckets.
- By default, all new buckets are PRIVATE. Hitting a url to a new file upload in a bucket will failed Access Denied. Have to grant public read access.
- Bucket Policies - bucket level. apply to everythig inside.
- Access Control Lists - object level
- S3 buckets can be configured to create access logs, which log all requests made to the S3 bucket. These logs can be written to another bucket.

**Encryption**
1. In Transit: SSL/TLS (HTTPS)
2. At Rest:
    1. Server Side Encryption
        - _AES-256_, S3 Managed Keys: each object with master key encryption. AWS manages keys. **SSE-S3**
        - AWS Key Management Service, Managed Keys, **SSE-KMS**
        - Server Side Encryption with Customer Provided Keys **SSE-C**
    2. Client Side Encryption - you encrypt yourself before it hits the cloud.

**Enforcing Encryption**
* PUT request each time a file is uploaded

```
PUT /myFile HTTP/1.1
...
Expect: 100-continue
```

Expect: 100-continue - don't send the body until acknowledged by S3 that content headers are set a certain way.

* If file is to be encrypted at upload time, use `x-amz-server-side-encryption` will be included in the request header.
* two options available:
  * `x-amz-server-side-encryption`: AES256 (SSE-S3 - S3 managed keys)
  * `x-amz-server-side-encryption` ams:kms (SSE-KMS - KMS managed keys)
* When this parameter is included in the header of the PUT request, it tells S3 to encrypt the object at the time of the upload, using the specified encryption method.
* You can enforce the use of Server Side Encryption by using a Bucket Policy which denies any S3 PUT request which doesn't include the `x-amz-server-side-encryption` parameter in the request header.

This tells S3 to encrypt the file using SSE-S3 at the time of upload:

```
PUT /myFile HTTP/1.1
...
Expect: 100-continue
x-amz-server-side-encryption: AES256
```

**CORS**
allowing code in one bucket or resource to access code in another.
by default they can't access other resources in other buckets because they are PRIVATE by default upon creation.

## CloudFront

CDN service

* edge location - where content is cached. separate to an AWS Region/AZ.
  * READ and WRITE
  * Used by _S3 for Transfer Acceleration_ - optimized network path for s3 transfer path.
* origin - could be an s3 bucket, ec2 instance, etc. where files originate.
* distribution - name given the CDN, which consists of edge locations.
  * Web Distribution - HTTP/HTTPS, used for websites.
  * RTMP Distribution - used for media streaming.

delivery of content using global network of edge locations/ requests for your content are autoamtcially routed to the nearst edge location, so content is delivered with the best possible performance.

## CloudFront + S3 Transfer Acceleration

S3 Transfer Acceleration enables fast, easy, and secure transfers of files over long distances between your end users and an S3 bucket.

Transfer Acceleration takes advantage of Amazon CloudFront's globally distributed edge locations. As the data arrives at an edge location, data is routed to Amazon S3 over an optimized network path.

Signed URLs or Signed Cookies are a way to restrict users access via URLs.
WAF helps protect you at the application layer

## S3 Performance Optimizations

GET intensive workloads you should use CloudFront.

```markdown
optimize if you get > 100 PUT/LIST/DELETE or > 300 GET requests per second.
```

_Mixed Request Type Workloads_ - key names can impact performance. avoid sequential key names for your S3 objects. Instead, add a random prefix like a hex hash to the key name to prevent multiple objects from being stored on the same partition. Reduces likelihood of I/O contention.

> 2018 July, new increase in performance negates the best practice of using sequential / random hash prefixes in keynames.


----

# Serverless

## Lambda 

Event-driven compute service.
Compute service to run your ode in response to HTTP requests using AWS API Gateway.
Supports: NodeJS, Python, Java, C#, Go
Price: 1st 1 million requests per month are free. $0.20 per 1 million requests after that.
- Calculated on duration of function execution
Scales out (not up) automatically
1 event = 1 function
1 event can trigger any number of events.

## Lambda Optimizations

1. Separate the Lambda handler (entry point) from your core logic.

```javascript
exports.myHandler = function(event, context, callback) {
	var foo = event.foo;
	var bar = event.bar;
	var result = MyLambdaFunction (foo, bar);
 
	callback(null, result);
}
 
function MyLambdaFunction (foo, bar) {
	// MyLambdaFunction logic here
}
```
1. Take advantage of Execution Context reuse to improve the performance of your function.
2. Use AWS Lambda Environment Variables to pass operational parameters to your function
3. To control the dependencies in your function's deployment, package all dependencies with your deployment package.
4. Minimize your deployment package size to its runtime necessities.
5. Reduce the time it takes Lambda to unpack deployment packages
6. Minimize the complexity of your dependencies (prefer simpler frameworks.)
7. Avoid using recursive code. Could lead to unintended function volume and escalated costs. To prevent on accident, set _function concurrent execution kimit_ to 0 immediately to throttle all invocations to the function.


AWS X-ray lets you debug what's happening.

## API Gateway

API is like a waiter relaying orders from customers to chefs and bringing results from chefs to customers.

Fully managed service for APIs at scale.
Expose HTTPS endpoints to define a RESTful API
Serverlessly connect to service like Lambda and Dynamo
Send each API endpoint to a different target
Scale effortlessly
Track and control usage by API key. Limit usage with _Usage Plans with API Keys_
Throttle API requests
Connect to Cloudwatch
Versioning of API
- Import custom on-prem API using Swagger Importer tool

1. Define an API (container)
2. Define resources and nested resources (url paths)
    3. select supported HTTP methods
    4. set seuciruty 
    5. choose target
    6. set rquest and response transformations
7. deploy api to a stage

API caching for endpoint response.
Caching lets you reduce the number of calls made to your endpoint and also improve the latency.
ex. McDonald's has most commonly ordered burgers ready to go to reduce latency.

Same-origin policy
Cross-origin resource sharing - can relax same-origin policy restrictions

Authorization:
* Lambda authorizers
* Cognito User Pools
* IAM user permissions

## Lambda Triggers

*   [Amazon S3](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-s3)
*   [Amazon DynamoDB](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-dynamo-db)
*   [Amazon Kinesis Data Streams](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-kinesis-streams)
*   [Amazon Simple Notification Service](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-sns)
*   [Amazon Simple Email Service](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-ses)
*   [Amazon Simple Queue Service](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-sqs)
*   [Amazon Cognito](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-cognito)
*   [AWS CloudFormation](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-cloudformation)
*   [Amazon CloudWatch Logs](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-cloudwatch-logs)
*   [Amazon CloudWatch Events](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-cloudwatch-events)
*   [AWS CodeCommit](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-codecommit)
*   [Scheduled Events (powered by Amazon CloudWatch Events)](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-scheduled-events)
*   [AWS Config](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-config)
*   [Amazon Alexa](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-echo)
*   [Amazon Lex](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-lex)
*   [Amazon API Gateway](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-api-gateway)
*   [AWS IoT Button](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-iot-button)
*   [Amazon CloudFront](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-cloudfront)
*   [Amazon Kinesis Data Firehose](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html#supported-event-source-kinesis-firehose)

$LATEST version of a function
Can have multiple versions
Can split traffic using aliases to different versions
    - canot split traffic with $LATEST, instead create version instead.

## Step Functions
Visualize serverless application
Automatically triggers and tracks each step
logs the state of each step so if something goes wrong you can track what went wrong and where.

## AWS X-Ray

Service that collects data about requests that your application serves, and provides tools you can use to view filter, and gain insights into that data to identify issues and opportunities for optimization.
- Client handlers to instrument AWS SDK clients that your app uses to call other AWS services.
- An HTTP client to use to instrument calls to other internal and external HTTP web services.

X-Ray integrates with Java Go Node Python Ruby .NET

# KMS
manaaged services to create and control encryption keys


# CI / CD

Multiple developers contributing to the same application. Committing code to the same repository.

Code repository integrated with build management system. Code changes trigger an automated build.

Test framework helps prevent breaks from commits. Focuses on small code changes to the main repository.

CD = development practice where merged changes are automatically build, tested, 

Dev commit -> repo -> build management system -> test framework -> deploy packaged app

CD deploys new code automatically foolowing successful testing.

CodePipeline - automate and orchestrate all of the pipeline activities.

CodeCommit (code repo) -> CodeBuild (code management system) -> CodeDeploy (deploy pakcaged app)

- [ ] read the whitepaper

CI is about integrating merging the code changes frequently at least once per day

CD all about automaticng build test and deployment. CD fully automates

## CodeCommit
* based on Git, enables collaboration, just like any other git hosting 

## CodeDeploy

Fully managed automated deployment service and can be used as  part of CD process.

**AppSpec**  file
Used to define the parameters that will be used for a CodeDeploy deployment.

```yaml
## Lambda

version: future use only
resources: name and props of lambda function to deplouy
    - myLambdaFunction
        - Type
        - Properties
            - Name
            - Alias
hooks: run at set points  in deployment lifecycle
    BeforeTrafficAllowed
    AfterTrafficAllowed
```

Before and After traffic allowed is good to test that the deployment is ready to accept traffic and that it is working as expected once traffic shifted to it.

BeforeBlockTraffic - run tasks on instances before they are deregistered from a load balancer
BlockTraffic - deregister instances from a load balancer
AfterBlockTraffic - run tasks on instances after they are deregistered from a load balancer.



```yaml
## EC2

version: 0.0
os: linux
files:
    source: file.txt
    location: where/it/is
    
```


**In Place**
* application is stopped on each instance and the latest fevision installed.
* The instance will be out of service / cpaacity is reduced
* ELB can stop sending requests for instances that are down
* Roling Update

**Blue/Green**
* new instances are provisioned and the latest revision is installed on the new instances. Blue represents the active, Green is the NEW release.
* The new instances are registered with an ELB traffic is then routed to the new instances and the oroginal intances are eventually terminated.

ADVANTAGE: new instances can be created ahead of time, release means you just switch ELB routing to new servers.
Switching back to original env is faster and more reliable and is just a acase of routing the traffic back to the original servers.

Deployment Group - set of EC2 instances that accept a deployment

## CodePipline

fully managed CI/CD service

Orchestrate the build, test, and deployment of app on every change.

Can be configured to auto trigger your pipeline on changes.

Docker commands for exam:
**BUILD, TAG, PUSH** to an ECR repository.

```bash
docker build -t mydockerrepo .
docker tag mydockerrepo:latest url.com/mydockerrepo:latest
docker push url/mydockerrepo:latest
```

Use `buildspec.yml` to define the build commands and settings used by CodeBuild to run your build.

You can override/insert build commands directly using an editor inline or use a `buildspec.yml` predefined.

Check CodeBuild console for logs or see in CloudWwatch.

## CloudFormation

* manage configure and provision infrstrastructure as code.
* YAML or JSON
* idempotent instances and configurations
* free to use except what underlying you use
* rollback/delete easily
* manage updates and dependencies

**Template**
- upload template using S3, CF reads it and calls API calls to make it happen.
- Parameters: custom values, environment type prod or test.
- Conditions: provision resources based on environment.
- Mappings: create custom mappins like Region:AMI.
- Tranformation: reference code yaml/json in S3 and run it. Code re-use here. The optional Transform section specifies one or more macros that AWS CloudFormation uses to process your template. The Transform section builds on the simple, declarative language of AWS CloudFormation with a powerful macro system. INTRINSIC FUNCTION.
- Resources: resources you are deploying. **MANDATORY**
- Outputs: user-defined 

## CloudFormation and SAM

Serverless Application Model (SAM)



## SQS 
Message queue - enables web app components to queue messages for other components to read from.

Pull-based
FIFO queues, only once, order guaranteed
Default Standard Queues
Short poll - return immediately if no messages in queue.
Long poll - spin

Message - ChangeVisibility
Minimum life 30 seconds, max 12 hours
Max retention period - 14 days
Max size 256KB

**Dead Letter Queues**
Amazon SQS supports dead-letter queues, which other queues (source queues) can target for messages that can't be processed (consumed) successfully. Dead-letter queues are useful for debugging your application or messaging system because they let you isolate problematic messages to determine why their processing doesn't succeed. 

## SNS
- scalable and highly available notification service.
- Push-based, no polling.
- $0.50 per 1 million
- pub/sub - users subscribe
- notifications delivered to clients using push 

## SES 
**Email only** service - for marketing teams.
Incoming and outgoing emails.
- status updates, shipping notifications, order status
- 

## Elastic Beanstalk
- you upload code, EB handles deployment and everything else. 
- Scales your app up and down
- You maintain admin control
- Supports Java, PHP, Ruby, NodeJS
- App Servers: Tomcat, Passenger, Puma, IIS

**Updating**:
* All at once - updates take all instances down while updates get applied to all intances. You need to roll all back. Not great for prod.
* Rolling - deploys new version in batches. Good for high performance systems.
* Rolling with batch - launches an additional batch of instances. can't afford downtime.
* immutable - deploys new version to fresh group of instances in their own new autoscaling group. preferred for mission critical. Roll backs just entail killing the unwanted instances.

**Configuring**

YAML or JSON file has configs. Needs to end in `myconfig.config` and in `.ebextensions` folder in top-level directory of app.

**Coupling with RDS**
- In dev OK, not in prod. Ties lifecycle together.

## Kinesis
Streaming data 

1. Streams - data producers feed KStreams. Data stored in shards, data is passed to consumer (EC2). The COnsumers then pass that to other AWS services. SHARDS
1. Firehose - analyzing data. real-time analytics for BI tools.
1. Analytics - SQL type queries off collected data

