# AWS Mics Services - I

**KMS (Key Management Service)**

It's a regional and Public service, so can be accessed from anywhere provided the rights access is there. Its fixed for the Region created so no multi region KMS

Its used to create, store and manage keys. Supports both symmetry and asymmetric keys. It internally encrypts and decrypts data. One can also upload keys in KMS. One cannot download keys from KMS, it supports **FIPS 140-2 (L2)�**�security complaiance. 

**CMK (Customer Master Keys)** : its like a container on top of physical keys. It has an id, date, policy, desc & state. CMKs can be used encrypt or decrypt a max of an **4kb of data.** CMK can be 

- **AWS Managed:** this is default. Supports rotation of data every 1095 days, i.e **3 years.** Rotation means that the backing data which does the actual crypto work is changed
- **Customer managed:** customer uploads the keys. Rotation is optional but if enabled will happen **once every year �**�

The 4kb limit of CMK can be a major limitation **but KMS create** **DEK  (Data Encryption Keys)** from **CMK** which can work on data more than once 4 KB. KMS does not store DEK

![Screen_Shot_2020-10-19_at_7-25-53_PM.png](image/Screen_Shot_2020-10-19_at_7-25-53_PM.png)

 

1. The user first use the createKey API to create a CMK which is stored in KMS in disk. The CMK is not stored in plain text but in encrypted format
2. The process to encrypt n decrypt has apis where the KMS used the decrypted CMK to encrypt and decrypt the data
3. NO time, the CMK leaves the KMS

**KMS Policies:�**�

KMS has a resource policy (like IAM Role Trust policy) which provides in Principal which account can use KMS. And subsequent only the identities which has identity policy for KMS can use it. Eg

{

    “Sid": “allow KMS for Account”,

    “Effect”: “Allow”,

    “Principal”: {“AWS”:”arn:aws:iam:::accountid:root”},

    “Action”: “kms:*”,

    “Resource”:”*"

}

The identity policy for a user will be like

{

    “Sid": “allow identity access to KMS”,

    “Effect”: “Allow”,

    “Action”: [“kms:encrypt”,“kms:decrypt”],

    “Resource”:”arn:aws:kms:*:accountid:key/*"

}

In the above case the user with that policy can only perform encrypt/decrypt but not upload keys

**Cloud Front**

CloudFront is a content delivery Network (CDN) which basically hosts remote contents from other regions and azs into the edge service for faster delivery

The most imp concept of CDN is Edge locations: They cache the data can be websites/s3 bucket files, static content.

To create a CDN, we have to create distributions. A distribution is telling cdn where the data origin is, we can set get all files from this origin or only of specific type.

By default CDN will come with [cloudfront.net](http://cloudfront.net/) domain name but we can added our own provided we have the certs.

Once the CDN is created we can choose to add more origins, route to different origins based on the rules. blacklist or white list particular countries

The origins for can can be S3, Elastic load balancer, EC2 or Route53

There are two kinds of distribution : Web Distribution which we usually use and RTMP distribution for video streaming distribution.

Objects are cached for TTL and you can delete content from cache but its charged

**Route53**

AWS Global DNS service. Route53 like IAM is a global service, Ione cannot choose the region. This managed service can do i) register domains ii) host zones (any domain name is a zone, .com is zone hosted by one company,eg Verisign, similarly google or amazon or any domain name is hosted by a zone. A zone consists of a zone file which is a database has all the details of a host name/zone  ),  managed nameservers . Since its a global service it maintains one database which obviously is HA and highly resilient 

**Registering a domain**

![Screen_Shot_2020-10-22_at_8-59-23_PM.png](image/Screen_Shot_2020-10-22_at_8-59-23_PM.png)

1. When a new domain is registered via AWS R53, R53 from the root zone hosted by IANA finds the registry that hosts .org. in that zone it checks whether the domain names exists already, if not next steps
2. a zone file is created e.g [animals4life.org](http://animals4loife.org/) and allocates nameservers
3. The zone file is copied to 4 nameserver with aws infra.
4. The ip of the 4 nameservers and registered in .org zonefile as NS records with the animals4life name  thereby making the 4 Nameservers as the authoritative for the domain 

**Hosting Zones�**�

A Hosted zone is a DNS for a domain. For hosting a zone , AWS maintains the zone files which are maintained in at least 4 hosted name servers . If the zone is public it will be part of AWS public infra . If the zone is private it will be accessible only through VPC. Route 53 is globally resilient. There is afee for hosted zone and DNS queries for that zone 

![Screen_Shot_2020-10-23_at_8-54-59_AM.png](image/Screen_Shot_2020-10-23_at_8-54-59_AM.png)

 

There is **50 domain names available by default**, however it is a soft limit and can be raised by contacting AWS support. 

**Difference between SOA, NS, A, CNAME and Alias records**

**SOA :** Start of Authority :  an SOA record store the name of the server that supplies the data  for the zone, admin of the zone, also contains TTL of the resource which determines how much time names should be cached

**NS: Name Server:�**� The NS records allow authoritative delegation. The Root zone file db maintained by IANA will have NS records pointing to Nameservers of .org, .in, .com etc . In each of the registries will intern maintain one files which again will have NS records pointing to the NSServers of a domain  for e.g amazon. This I how delegation works. 

**A** a.k.a address records are meant to point to Ipv4 addresses for a given domain name. There can be multiple A records with the same name like “www” which basically means multiple underlying servers for the same domain

**AAAA** address records for IPv6 addresses

**CNAME** can map multiple domain names to the same ipaddress. CNAME does not point to the IP address but to A record. The problem is it can only map to non naked domains (domains with no www) and cannot be used to point to ELB as that has naked names and do not have ipv4 addresses

**MX :** records should be used to find the mail server  to sent emails via SMTP protocol. The process is still the same to find the zone name using NS records and finally finding the zone, search MX records to find the ip of the mailing server. There can be many MX records and ones with highest priority is returned 

**Alias :** Alias records are only for AWS Route53 which can map to naked domain names  

ELB cannot be access by ipv4 address but only with its DNS name. Thats the reason Alias records should be created in DNS to point to ELB

TTL : its the time in seconds for the domain details to be cached at the client. after that it invokes the domain registrar (go daddy etc) too get the DNS records again

**Routing policies**

**Simple:�**�Default routing policy for a record A set. Mostly used when there is only one server (web server) to server one domain. So from lab , if we create an A record of type ALIAS for route53 domain and target is the ESB we created, since its only one.  Even if there are multiple EC2 instances instead of one ELB , we can create A records with multiple entries. The problem with simple is there is no health check and if one of the underlying server is down the user may get errors. 

**Weighted:**This is used when we want some percentage of requests to be routed to one loadbalancer and the rest to another. For example one the request hits Route53 we want 80% of requests to call US-EAST-1 loadbalancer and 20 % to call US-WEST-1 region load balancer. We may have load balancers in both locations. Its not necessary that weighted only works with multiple region. Its mostly used for load balancing or trying new version of software

**Latency:�**�This policy is used based on which region will give the user the lowest lateens policy. If we have two region ELB , based on the user where the request should be routed to

**Failover:�**�As the name suggests , this policy is used for primary/secondary structure where if I want I want secondary load balancer to be called only when the primary fails.Internally it does a health check. The secondary record can pe the path to S3 static host. Unlike simple the failover returns the primary by default but will return the failover only when primary is unhealthy

**Geolocation:�**�It lets you route requests based on the users geolocation

**MultiValue:�**� Multivalue answer routing lets you configure Amazon Route 53 to return multiple values, such as IP addresses for your web servers, in response to DNS queries. Route 53 responds to DNS queries with up to eight healthy records and gives different answers to different DNS resolvers. The choice of which to use is left to the requesting service effectively creating a form or randomization.

**Health Checks**

There are 2 kinds of checks done by R53. One is TCP , where it checks whether TCP connection can be established against the servers and 2nd HTTP/HTTPS . The response from Health checks can be configured with CloudWatch alarms. Healthchecks do not check the  A records, rather you have to mention the actual endpoint. Health checks can also be configured to check other cloud watch alarms , for example if the underlying EC2 or ELB are under clouddwatch, the health check of R53 can check that rather than do its own checking

**Elastic Load Balancer**

There are three types of load balancer, 

**Application LB:�**�application routing happens in level 7 application . They are best suited for http, https LBs, they are intelligent and can do content based routing

**Network LB**: Network load balancer are for high performance, low latency

**Classic LB:**  For classic routing happens in transport layer.  These are the legacy Elastic LB, one can LB http/ https , sticky sessions can be done here. This should be used when there is major routing rules and basic round robin would do. it also Supports X-Forwarded-For header which maintains the ip address of the actualy client and not the LB, if checked in the web server which is behind the LB

Advanced Load balancing using ELB

Sticky Session : requests from one session can go to one target 

Application ELBs are by default no cross AZ , ie ELB will be associated with only one AZ, a typical example is. As one can see, US east 1b is overloaded

![4c5d22c0994ed2ca912167283d3ba89f.png](image/4c5d22c0994ed2ca912167283d3ba89f.png)

With Cross Zone

![c20e4abcdc728ee39d7dfebb9d5d6172.png](image/c20e4abcdc728ee39d7dfebb9d5d6172.png)

Application LB also provides path based routing, a particular context path can go to one AZ. These are done through Target Groups

**Cloud Formation (CFN)**

It is the AWS equivalent of Chef, basically Infra as code. One can create own stack and script, or use an existing script which is there in quickstart 

[https://aws.amazon.com/quickstart/?quickstart-all.sort-by=item.additionalFields.sortDate&quickstart-all.sort-order=desc](https://aws.amazon.com/quickstart/?quickstart-all.sort-by=item.additionalFields.sortDate&quickstart-all.sort-order=desc)

What makes a template 

![Screen_Shot_2020-10-16_at_8-56-36_PM.png](image/Screen_Shot_2020-10-16_at_8-56-36_PM.png)

**Resources** :  This tells CFN what to do. If the resources already exists it means CFN will update the resource. Resources section is the only mandatory section of CFN template

**Description**: just plain text desc of the template. If AWSTemplateFormat version is provided, Description should immediately follow that

**Metadata:** key value pairs

**Parameters:**  provide parameters based on the resources, these can take input params from the end user who runs the template. This also provides default values

There are no extra charges for CFN. We are charged based on the resources being created and used like EC2, S3 etc. CFN templates are stored in S3. 

**Cloud Formation Init**

CFN Init is used to pass complex bootstrap instructions via CFN. There is a helper script in EC2 OS called cfn-init which is a simple configuration management script which maintains the desired state of the the EC2 instance similar to K8 architecture. When we pass bootstrap via AWS Console that is run as procedural script but with cfn-init we can maintain the set a state for things like packages, users, groups, sources and files within resources inside a template - and it will make that change happen on the instance, performing whatever actions are required. Its similar to Java 8 streams, i.e declarative programming, we just mention what the state is, and cfn-init takes care of maintaining the state. The information is provided in the metadata of CFN templates in AWS::CloudFormation::Init

**CFN init flow�**�

Unlike user data which runs only on load, Cfn-init also checks with CFN templated for updates and updates the instance . Based on the init template in CFN and user data is internally created and passed to the tool during start up, there is also cfn-signal which is used to signal to  template that the steps where successful, else the template never knows whether the startup steps were success. We can also pass timeout in creating policy segment of CFN template. With  timeout if no response is sent by EC2 to CFN by timeout it is deemed as error

![Screen_Shot_2020-10-22_at_12-13-06_PM.png](image/Screen_Shot_2020-10-22_at_12-13-06_PM.png)

  

**CloudWatch** 

Collects and manages operational data on user’s behalf . IT may be performance data, logs, etc. It has 3 components

- **Metrics** : metrics on AWS services, like cpu, memory , disk utilization. Since its a managed services, it can be used as metrics gathering for on premise applications too. Few services come with native cloud watch support like EC2 etc, for others there are CW agents. An metric consists of data points in an time sequence. The data point will contain the value of the metic, timestamp and dimensions (key value pair metadata on the service) which identifies which service instance and its values. All metrics are saved in its own namespace like AWS/EC2, AWS/VPC etc. Based on metrics Alarms can be raised .
    - Alarms has 3 states: insufficient data,  OK and ALARM
- **Cloudwatch** logs : 
    - Its a regional service
    - to gather logs like application logs, product logs etc. 
    - it is used to store, monitor and access logging data. Its integrated with EC2, VPC Flow Logs, Lambda, CloudTrail, R53, etc.
    -  It can also be integrated with other applications using agent. 
    - It can also generate metrics based on the logs
    - Logs are set as log streams which are orders of log events from a specific source like application, vas, ec2 etc
    - Log streams are grouped by **Log Groups** where at the retention and permissions can be set

![Screen_Shot_2020-10-19_at_11-57-09_AM.png](image/Screen_Shot_2020-10-19_at_11-57-09_AM.png)

 

Logging in EC2

![Screen_Shot_2020-10-22_at_1-25-44_PM.png](image/Screen_Shot_2020-10-22_at_1-25-44_PM.png)

The cloud watch agent need to be installed in EC2 and the EC2 needs the right permissions to to send logs to cloudWatch . The agent config can be in Parameter store 

- **Cloudwatch** events :  to generate events from aws services , like event can be generated when EC2 instances are stopped, terminated etc. it can also generate events to do something on a certain time. Cloud watch events cannot listen to custom applications or external services, that’s where **EventBridge** services comes handy. It has the same features as CloudWatch events 
    - EventBridge or CloudWatch Events are one way to handle EDA in AWS
    - With EB one can trigger an event X happens or trigger when X happens at Y time , the do Z kind of statements. These are called rules 
    - Both CloudWatch events and EventBridge underlying architecture is same. They use an **Event Bus** arch which is an stream of events. There is a default Event bus per account which is implicitly used by both CW Events . EventBridge can use additional event buses , external    
    - The events are JSON document 

 

For EC2 instances default metrics is enabled which is free and at a period of 5 minutes. So its not granular.

**CloudTrail**

A managed service to log the API events which affect AWS services. Examples can be stopping/starting EC2, changing security groups, CRUD of S3 buckets. All these are logged using CloudTrail. 

- Each activity is called CloudTrail Event
- By default each event is stored for 90 days for free and it is not stored in S3
- To save more than 90 days, one need to create trails
- Events are categorized as management and data events. Management events a.k.a Control pane events will be creating , deleting services. Data is resource operations within the aws resource, like objects uploaded , downloaded in S3. By Default CloudTrail only logs management events. To log data events trails need to be created 
- Trails can be configured for One Region or All Region. One Region only logs events for that region and in that region, it's the default. Global services like S3, STS, IAM log events in us-east-1 (N Virginia) and that trail need to be manually configured
- Trails store events can be saved 2 ways
    - as compressed json in S3 buckets 
    - Into cloudWatch logs
- If the trail is created from the root org master account, it can trail from all accounts in the org
- Its not real time, trails can be viewed only after ~15 minutes of the actual events generated

**Elastic BeanStack**

**On Premise Services**

These are the services that can be used on premise too 

- Database Migration Service :
    - This allows 2 migrate database from and to AWS.
    - One can keep DR in AWS and the actual db in on premise
    - works with both RDS and DynamoDB
- Server Migration Service : 
    - Same as Db but for servers
- AWS  Application Discovery Service
- VM IMPort/Export
- Download Amazon Linux 2

**Application Services**

AWS Security token service

The STS service grants temporary access to aws resources. One can get temporary IAM credentials from STS service by passing IAM policy, for which resources and TTL

The flow can be

The user enters their AD or google credentials to login to an application. The application needs to access say S3 on users behalf

The application invokes an identity broker, the IDB validates the credentials with AD or with other web based identity provider

Once validated, the IDP invokes its function getFederatedToken  

[https://sts.amazonaws.com](https://sts.amazonaws.com/)

The STS service returns 4 values, access key, secret access key, access token and TTL

Workspaces

Workspaces are virtual desktops with windows 2008 installed giving a win7 experience. Its a VDI (Virtual Desktop Infrastructure) in the cloud.One can login to workspaces using AD credentials provided it has been configured to integrated with AD. The workspaces are persistent and users get local administrative  access. Data is backed up every 12 hours

**ECS**

- Elastic container service uses services to deploy in a region across multiple AZs. ECRs are the registry, you can use IAM policies to manage access to ECRs.
- Container definition contains the docker image and the port details. There is one container def for each image
-  Task definitions are like docker compose, it maintains the cpu, memory utilization, networking mode of container tasks. Most important thing about task definition is the task role which the task can assume to interact with AWS resources. One task definition contains one or more containers, hence like docker compose. 
- The task definition does not contain scaling info and HA info. Those are mentioned in service definition which informs how many tasks to run, load balancer etc, just like K8 service. 
-  ECS creates a default cluster in a region, all instances in the cluster will be in the same region .  
- ECS can be triggered using scheduler service which  one can configure to runs tasks when another fails. Also it allows to leverage third party schedules . 
- ECS also has container agents which allows container to join clusters. 
- We can also install ECS agent in an existing EC2 instance and there are AMIs which comes with agents but only linux based AMIs. 
- ECS is secured by IAM roles, to access ECS service and also the service to access other resources , security groups are only on the host and not on the container within the host

**Cluster Modes/Types:** the difference on the modes is how much admin overhead and control one needs , cost differences 

**EC2 Mode:** with EC2 mode, one is responsible and billed for the EC2 instances which are running . One needs to worry about the availability and scaling of infra. Even if no tasks/containers are running , in EC2 mode you will be paying for the EC2 instances. One should use EC2 mode when the work load is more and price conscious so that we have control over the infra

![Screen_Shot_2020-10-22_at_10-46-39_AM.png](image/Screen_Shot_2020-10-22_at_10-46-39_AM.png)

 

**Fargate Mode:** this is server less mode to run ECS. No management overhead, no need to worry about underlying EC2 instances. The arch is there is a shared Fargate Infra where the containers are deployed. If we have our own custom VPC an Elastic Network Interface ENI is injected into the VPC which acts as a interface to my containers in the shared Fargate infra. If overhead conscious then Fargate is the solution. Ideal for scenarios when we do not know the load or spikes. Periodic or batch loads

**![Screen_Shot_2020-10-22_at_10-54-22_AM.png](image/Screen_Shot_2020-10-22_at_10-54-22_AM.png)**
