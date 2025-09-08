# AWS Learnings

**A 10000 foot view of the services**

**Compute**

EC2 : Elastic compute cloud: VMs or dedicated machines

EC Container Service: a service for provisioning docker containers

Elastic Beanstalk: Auto provision of EC2, VPC so that dev can concentrate on their code. Its like PAAS

Lambda: serverless service,

Lightsail: Amazons VPS (Virtual Private Service ). A high level service for creating servers without working about Ec2 or VPCs. It comes with a management console

Batch : its used for batch computing

EKS : Elastic Kubernetes service

**AWS Fargate :Â** Â is a serverless compute engine for containers that works with both Amazon Elastic Container Service (ECS) and Amazon Elastic Kubernetes

**Storage**

S3: Simple Storage Service : Object storage service. It has buckets and files within buckets

EFS: Elastic File storage: A network based file storage which can be mounted to any Ec2 instance

Glacier : long term storage

Snowball: Bringing large amount of data in the datacenter . It comes with a device which can be sent to AWS to upload instead of seeing via network

Storage gateway : these are virtual machine that are installed in dc and then it will replicate data into S3

**Databases**:

RDS : Relational databases, mysql (aurora) PostgreSQL

DynamoDB: No sql database

Elasticache : the cache system

RedShift: data ware house or analytics db

Dads

**Migration Services:**

AWS Migration Hub: It tracks the migration once one migrates the app to AWS

Application Discovery Service: it shows the dependencies a service has which is deployed in AWS

Database Discovery Service: migration of SQL based database

Server migration service: it migrates the config of servers

**Networking and Content Delivery:**

VPC: Amazonâ€™s virtual private cloud: its like a virtual datacenter, where you can configure the firewalls and the network address ranges , route tables

Cloudfront: its AWS CDN (content delivery network) which basically hosts remote contents from other regions and azs into the edge service for faster delivery

Route53: its the AWS DNS service

**API Gateway:**

Direct Connect: Setting up a secured direct line, between AWS and private office datacenter

**Developer Tools :**

CodeStar: a collar tool for developers:

Codecommit : a source control service. It can have private git repo

Code build: it will compile and test

Code deploy: deployment service to EC2 or Lambda:

Codepipeline : a cd service

X-ray: and code quality service

Cloud9: a web based IDE

**Management Tools :**

Cloudwatch: its an admin tool for your instances and overall health of your account

Cloudformation : It a scripting tool for cloud admin, like chef puppet etc. It sets up information in to code

Cloudtrail : its basically a auditing service which audits every event we do under the management console. Cloudtrail can be enabled one per account and its for a region

Config: it monitors the Aws environment and also has snapshots for any point in time to back to an older config

Opswork: it uses chef and puppet to automate your environment

Service catalog : it s a catalog of all services that are provisioned in your was account

Systems Manager: a admin manager for EC2 instances, like apply updates security patches to EC2 instances etc.

Trusted Advisor: Gives advices on security, best practices etc for the was accounts. It can say how to properly use the services, save money etc

Managed services: it takes care of auto scaling EC2 instances etc

**Media Services:**

Elastic Transcoder: it basically converts an uploaded media file to an universally accepted format

Machine Learning (Does not come in exams)

Sagemaker: Its a deep learning algo

Comprehend: it does sentiment analysis over data

Deeplens: its anctually a device with camera which can recognize patterns and work on that. For e.g. a camera which recognizes a person and can work on the next steps

Lex: this drives Alexa. Its an AI tool for chatting to customers

Machine learning : it analyses the dataset . All amazon recommend service is used on ML

Polly: a text to speech tool. Alexa uses it. It can convert text to mp3

Recognition : as the name suggests it can recognize images or videos. Based on the image it can say what is going on there and tag that etc

Amazon translate : language translation

Amazon transcribe : Automatic speech recognition and it create closed captioning on the fly

**Analytics:**

Amazon Athena : It uses a query language to query objects stored in S3

EMR: Elastic map reduce: processing large amount of data for big data use cases

Cloudserach & elasticsearch services :

Kinesis: ingesting large amount of data into aws and analysis

On that.

Kinesis Video streams : Analysis video streams can

Quicksight : web based analytics tool

Datapipeline : moving data between different was services

Glue : AWSâ€™s ETL solution

**Security & Identity & Compliance**

IAM : Identity Access management

Cognito : Its an Authentication service for users to access secured resources using their social network using SAML

Guard Duty: monitors for malicious activity on aws account

Inspector : an agent that can be installed on EC2 instances which can look for security vulnerabilities etc.

Macie : scans S3 buckets for PII, PHI data

Certificate Manager: A way to manage SSL certificates

CloudHSM: Cloud Hardware Security Module : Its a key management service where public and private keys can be added to access other AWS services, or VMs (can be ssh keys etc)

Directory Services : Integrating AD with AWS services

WAF : Web Application Firewall, its a firewall over applications which prevents cross site scripting, SQL injection etc

Sheild : Prevents DDoS attacks.

**Mobile Services (Does not come in exams for certs)**

Mobile Hub :

Pinpoint : Alert push notification

Device Farm : testing the application on real mobile devices

Mobile Analytics :

**Application Integration:**

Amazon MQ: Similar to any message broker system

SQS : Simple queue service

SNS : Notification service, can be configured to send notifications

SWF: Simple Workflow service :

**AWS Organizations**

**![8956e6eeaafdb045d0fd3d193d7e0d19.png](image/8956e6eeaafdb045d0fd3d193d7e0d19.png)**
