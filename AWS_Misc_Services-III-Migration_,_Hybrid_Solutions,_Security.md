# AWS Misc Services - III - Migration , Hybrid Solutions, Security

**Site to Site VPN**

- A logical connection between a VPC and on premise network encrypted using IPSec, running over public internet
- Can be HA if architected well 
- Can be setup in less than an hour unlike Direct Connect
- Not as performant as Direct Connect since internet is being used instead of direct connection
- Works with VPC, so at the VPC end Virtual Private Gateway (VGW) can be set up to connect to VPN . For one VPC , there is one VGW and its the target of the route tables
- There is also Customer gateway CGW which is set at the customer end
- Last the connection is made between the CGW and the VGW
- Dynamic VPN uses BGP (Border Gateway Protocol) where one need not enter the subnet or router ip addresses , the only consideration is the customer router need to support BGP protocol

**Arch of VPN**

![Screen_Shot_2020-10-24_at_3-36-58_PM.png](image/Screen_Shot_2020-10-24_at_3-36-58_PM.png)

- There is the VPC with 3 private subnets with cidr 10.16.0.0/16 and the on premise has 192.168.10.0/24. Before setting up the 2 network ranges is needed and also the IP address of the physical router at the customer end
- A CGW need to set at customer end with a public IP which matches the physical router for IP
- The VGW at the VPC level has physical endpoints at diff AZs which has pubic endpoint.
- Te last step is set up vpn pointing to the VGW and the CGW which sets up secure tunnel between each public endpoint and the CGW. The tunel is secure and encrypted.
-  The above example is static connection since IP address range of the customer need to be provided. There is also a single point of failure here since the customer end has only one router and CGW. To make it HA, the customer can have 2 routers with 2 CGW in 2 separate buildings.

![Screen_Shot_2020-10-24_at_3-45-21_PM.png](image/Screen_Shot_2020-10-24_at_3-45-21_PM.png)

**Direct Connect DX**

- If there is a need to directly connect the data center to AWS , it can be done with Direct connect, which is a physical connection. 
- The difference with VPN is that VPN can be set up faster but then it is dependent on internet whereas direct connect uses intranet. It can only work from places which has a Amazon data connect facility . 
- There will be a dedicated line to the DX facility and from there amazon creates a direct line to the nearest region. The process to setup a direct line takes a lot of time, probably months. The network speed is faster than vpn
- DX supports 1 Gbps or 10 Gbps connection to AWS. DX is not exactly a connection but a port assigned to the customer at a DX location, 1000-BaseLx for 1 gbps and 10GBaseLR for 10 GB connected via fibre optic cables 

![Screen_Shot_2020-10-24_at_4-35-48_PM.png](image/Screen_Shot_2020-10-24_at_4-35-48_PM.png)

There is physical connection that need to be set up by telco companies between the premises and physical port of the nearer DX location . The data through the cable is secure but not encrypted. The connections are done VIFS one at client location and VIFS at AS based on the number of VPCs and Public Zone Services. The VIFS o public services is only for AWS services and not internet

**AWS Transit Gateway TGW**

The AWS Transit gateway is a network gateway which can be used to significantly simplify networking between VPC's, VPN and Direct Connect. Its a single Gateway object which simplifies networking connection complexity. Without a Transit gateway connections between Multiple VPCs and a Customer On Premise connection would look like

![Screen_Shot_2020-10-24_at_7-17-58_PM.png](image/Screen_Shot_2020-10-24_at_7-17-58_PM.png)

Since VPC peering and VPN s are not transitive, each VPC needs at least 2 public endpoints connected with atleast 2 CGW for HA and also a individual connection between each pair of VPC. This is a huge complicated mess. A TGW can improve this mesh mess. A direct connection between CGW and the TGW and individual connection between TGW and VPCs is enough for the same. Also can connect to other TGW in other account VPCs and also to Direct Connect

![Screen_Shot_2020-10-24_at_7-31-21_PM.png](image/Screen_Shot_2020-10-24_at_7-31-21_PM.png)

**Storage Gateway**

- The Gateway runs on the customers infrastructure and routes all storage need to AWS. The gateway is a VM which can be downloaded and installed in the customers infra, it supports both VMWare ESXi and Microsoft Hyper V.once its is installed and accosted with was account, you can use the console to manage and configure it.
- One can extend of file and volume storage into AWS. Mainly used for backup , migration of data
- Volume storage backups into AWS. Bacically used for volume backup and DR. Also Tape backups into AWS
- The SG has 3 modes
    - TAPE Gateway (VTL) mode: Virtual tapes are backed up in S3 and Glacier. The tape system thinks its using on premise tape backups but internally Gateway uses https to connect to S3 and Glacier to store the tapes
    - File Mode : uses NFS or SMB (Server Messaging Protocol for windows server) as a backup in AWS: the files are stored in S3. The internal connection between gateway and AWS happens through https. In file and SMB mode, AD can be used to authorize access to files
    - Volume mode: its like running Volume EBS but on premise. It has 2 modes **cache** and **stored.** Block storage backed by S3 or EBS Snapshots
        - Storage Mode : In this mode volume is just backedup async . The primary data is stored on premise but a backup is stored in AWS. Its mainly used for DR. Max of **16TB** per volume with max of **32** volumes

                            ![Screen_Shot_2020-10-24_at_8-26-18_PM.png](image/Screen_Shot_2020-10-24_at_8-26-18_PM.png)

        - Cached Mode : its used for extending your local volume capacity instead of only DR. The architecture is same as above the only difference is no data is stored on premise , everything goes to cloud, only the most used files is cached on premise

**Snowball**

Snowball replaces Import/Export disk which as a way for customers to ship their card disks to or out of Amazon for them to upload data from the disk to aws storage or outside. This was needed when network that customers had was very slow.

3 types of snowball

**Snowball:** This is just a device which is sent to customer for them to upload all their data and then send that back to amazon. This is needed when huge amount of data like **50 TB** needs to be backup and instead of insuring network costs just ship the device to AWS. It is AES256 bit encryption

**Snowball Edge:** similar to snowball but with **100 tb** storage space and also compute power in it. Sort of a mini datacenter

**Snowmobile:** its basically a truck which can carry **100PB** of data

**AWS Data Sync**

Data transfer services to and From AWS. Can be used for migration, data processing into AWS and moving back to on premise after processing, Archival or cost effective storage, DR.

Huge scale, each agent can handle **10 Gbps** which is lead to **100TB of transfer per day.**  Each job can transfer **50 million�**�

Few very useful features:

- Built in Data validation. This is huge feature since it validates most migration or transfer 
- Bandwidth limiters : instead of continuous backup transfer, limiters make sure the backup does not hog the network 
- Incremental and scheduled transfer: we can schedule transfer during offpeak time
- Compression and encryption
- Service integration - to S3 , EFS, FSx and also move from one account to another 
- Billed per GB transferred 
- It supports TASK, which is job which defines what is being synced , how quickly FROM where TO 

![Screen_Shot_2020-10-25_at_10-01-58_AM.png](image/Screen_Shot_2020-10-25_at_10-01-58_AM.png)

**AWS Directory Service**

Its AWS version of MS Active Directory.Runs within a VPC. Its HA and available in multiple A. Some AWS Services need Directory service like Amazon Workspaces, Chime, Connect, RDS, Quicksight, Sign in Console. This can run in 3 modes

- Simple AD mode : the AWS DS is totally isolated. Its based out of Samba 4 which is an open source version of MS AD with some compatibility with AD. It can be configured from 500 to 5000 users . It doesn’t have all the features of Microsoft AD
- Managed Microsoft AD : Integrated with an on premise AD, just as an extension of the on premise AD. The integration with on premise AD happens through VPN or Direct Connect . This is fully fetch directory service in cloud with a trust with on premise. So even if the connection with On Premise AD fails, the DS and integrated apps works
- AD connector : just acts as a proxy of the on premise AD. The cloud version holds no data but proxies to the on premise. Should be used only when a on premise AD is available and in cloud only one or two service requires AD. Fails if direct connection fails 

**AWS Database Migration Service (DMS)**

The DMS is used to migrate database IN or OUT of AWS. The concept is based on having Replication instance as part of DMS which has the replication tasks which can run in parallel.

Task Jobs can be 

- Full Load : Moving the entire DB on one go. Does not handle changes during the transfer process , hence src db should be stopped before transfer process
- Full Load + CDC (change data capture) : along with above also handles changes during the transfer process
- CDC only 

**AWS Secret Manager SM**

It does share functionality with SSM Parameter Store, though SM is designed for secrets (passwords, API keys).

Can be integrated with console, cli, API , SDK, can be integrated with RDS

Supports secret rotation using lambda functions and encrypted at rest

**AWS Shield**

AWS Shield protects AWS resources against DDoS (Distributed denial of service) attacks

**Shield standard** is free for Route53 and CloudFront. If access to APIs are via R53 or CF then shield comes for free. Shield protects against **layer 3 and layer 4** DDoS attack

**Shield Advanced** is 3000 per month for an organization and it can protect EC2, ELB, Global Accelerator, CloudFront and R53. Adv also provides response team an also provides financial security against extra charge against DDoS attack

**AWS WAF (Web Application Firewall)**

Usually firewall works at layer 3 (IP) , 4(TCP/UDP) and 5 (Sessions) securing sockets.

WAF understand http/s and its **layer 7** firewall. It protects against complex layer 7 attacks like SQL Injections, Cross Site Scripting , Geo Blocks , Rate Awareness

The way WAF works is it has WEBACL Web Access Control lists which are integrated with ALB, API Gateway and CloudFront. Rules are added to a WEBACL which the WAF evaluates at runtime when traffic arrives 

![Screen_Shot_2020-10-25_at_11-54-30_AM.png](image/Screen_Shot_2020-10-25_at_11-54-30_AM.png)

 

 
