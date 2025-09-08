# AWS EC2 (Elastic Compute Cloud)

**EC2 Architecture**

**![Screen_Shot_2020-10-20_at_10-49-43_AM.png](image/Screen_Shot_2020-10-20_at_10-49-43_AM.png)**

- Virtual machines which run in only one AZ. The EC2 hosts also runs on only one AZ, so does the Subnet and the EBS Volume . So with EC2 all services are in same AZ, so if AZ goes down everything for that EC2 instance fails. One cannot point EBS volumes or Network running in One EZ to  EC2 instance in another AZ. 
- EC2 instances run on the same host machine if restarted. It is assigned a different host machine only when its stopped and started, or the host was stopped for maintenance. But in either case the host will also be in the same AZ as previously. 
- One can migrate EC2 instances to another AZ by creating snapshots and AMIs
- Its logically that instances of same types in same AZ may share the same host. Two different types of EC2 instance will not share the same EC2 Host
- When to use EC2
    - A certain OS and Hardware needs
    - Burst of CPU or Memory
    - Long running compute
    - Monolithic stack 

**Pricing Models:**

**On Demand**: where EC2 instances are charged by hourly rate of using them. By second is recent update and it is only for linux systems and not for windows yet. When we hourly rate but linux t3 systems can charge based on seconds (min 60 secs) but on the hourly rate, for e.g is hourl rate is 2$ and we run for 65 secs it will 2/3600 * 65. Used for short term , unpredictable load, uncertain application requirements and can’t tolerate any disruption 

There is no commitment. This is mostly used for new applications whose usage is not known, can be spiky and needed to be scaled up nd down when needed.

**Reserved** : Here one can reserve an instance for 1 to 3 years contract. You can get huge discounts. These are for applications whose usage is known . Can be booked for a region and or for a AZ. The customers have dough to put up upfront

There are 3 kinds of RI (Reserved Instnces) :

    Standard: unto 75% discount from on demand

    Convertible RI : 54% discount than on demand, has the capability to change the configuration e.g change from unix instance to windows

    Provisioned RI : It can be provisioned to a specific date and time. For needs when you know certain time it needs to be provisioned  

**Dedicated** : Here instead of multi tenant instances, the customer will get EC2 instances on dedicated machines.

This are for customers who has regulatory needs for dedicated infra instead of virtual machines in a multi tenant infra.

It can be purchased as on demand (pay per hour) or reserved for discount (just like RI). When provisioning dedicated instance, one has to mention which family of instances . No charge for instances running on the dedicated host. Dedicated can also be

- On demand 
- Reserved 

Dedicated instances have physical cores and sockets 

![Screen_Shot_2020-10-22_at_2-31-17_PM.png](image/Screen_Shot_2020-10-22_at_2-31-17_PM.png)

**Spot:** You bid for prices for instance capacity . So if you bid lower than the current price, the instances are not provisioned, as soon as it reaches the bid price the instance is provisioned . Again as soon as the price goes up than the max you can pay, the server is decommissioned. Pricing is 90% off from on demand. The spot prices fluctuates based on the spare capacity of that type of EC2 config in that region. Mainly used by companies to do high compute backend processes which are over in a while and then don’t have to pay when the prices are high  

**The different types of EC2 instances**

![Screen_Shot_2020-10-20_at_11-31-47_AM.png](image/Screen_Shot_2020-10-20_at_11-31-47_AM.png)

A useful site for Ec2 instance comparisons and sizes [https://aws.amazon.com/ec2/instance-types/](https://aws.amazon.com/ec2/instance-types/),  [https://ec2instances.info/](https://ec2instances.info/)

**General Purpose :**

For general purpose load, Web Applications, micro services , equal resource ratio

    T2 : The baseline version. Has optimal CPU and the ability to burst above baseline . Suited for low latency services, micro services, small databases. This should be selected when there are chances of CPU bursts but memory and network may not be used that much

    M5: a general purpose instance which has optimized compute, memory, network resources. For more memory intensive use cases than T2 instance. Some data processing tasks that require high memory, Medium to high DBs

    M4: Same as M5 but previous genx

**Compute optimized : (CPU)**

Media Processing, Scientific modeling, Gaming and Machine Learning

    C5: high optimized compute intensive workload , 3.0 GHz Intel Xeon Platinum processors, High performance web servers, scientific modelling, batch processing, distributed analytics, machine/deep learning inference, ad serving, highly scalable multiplayer gaming,

    C4: Similar purpose as C5 but of previous generation and CPU processing power is lesser than c5

**Memory Optimized :  (RAM)**

For in memory databases like Redis

    X1e: X1e instances are optimized for high-performance databases, in-memory databases and other memory intensive enterprise applications,     One of the lowest price per GiB of RAM. Meant for High performance databases, in-memory databases . Business Warehouse on HANA (BW), and Data Mart Solutions on HANA on the AWS cloud

    X1 : Similar purpose and X1e but lower memory options

    R4: Same purpose as X1e but again different memory options

**Storage : (SSD, HDD)**

H1 : instances feature up to 16 TB of HDD-based local storage, deliver high disk throughput, and a balance of compute and memory. Up to 16TB of HDD storage,MapReduce-based workloads, distributed file systems such as HDFS and MapR-FS, network file systems, log or data processing applications such as Apache Kafka, and big data workload clusters

I3: provides non volatile memory express SDD storage with optimized memory and cpu. NoSQL databases (e.g. Cassandra, MongoDB, Redis), in-memory databases (e.g. Aerospike), scale-out transactional databases, data warehousing, Elasticsearch, analytics workloads. Since its SSD over H1’s HDD, it is supposed to be faster for fatser nodal dbs.

D2:  instances feature up to 48 TB of HDD-based local storage, deliver high disk throughput. MapReduce and Hadoop distributed computing, distributed file systems

**Accelerated Computing :**

P3 : instances are the latest generation of general purpose GPU instances.Machine/Deep learning, high performance computing, computational fluid dynamics, computational finance, seismic analysis, speech recognition, autonomous vehicles, drug discovery.

P2: Same as above with lesser options

G3: Graphic intensive applications :3D visualizations, graphics-intensive remote workstation, 3D rendering, application streaming

F1 instances offer customizable hardware acceleration with field programmable gate arrays (FPGAs).

**This is a boot script for EC2**

#!/bin/bash

yum update -y

yum install httpd24 -y

service httpd start

chkconfig httpd on

echo "<html><body><h1>Hello Cloud Gurus, this is Tatha</h1></body></html>" > /var/www/html/index.html

yum remove java-1.7.0-openjdk

yum install java-1.8.0

**Security Groups**

- These are virtual firewall over the VM.
- Security groups applied/removed are affected immediately
- Security group rules are stateful, I.e if the rule applies when the request is made , the same rule applies for the response of the first request. Also if the rules applies to inbound the same applies for outbound . Network ACL are stateless, I.e inbound rules are not replicated for outbound rules. Basically it means that with security groups we can only allow rules but cannot block. With Network ACL we can block certain its or ports etc. this will be covered under VPC
- All outbound routes is allowed by default
- There are no deny rules
- Security groups span across different AZs
- One can assign multiple security groups to one EC2 instance

**EBS**

Elastic Block Storage provides network storage for EC2 which are persistent and separate from the EC2 instance , meant for installing operating systems, databases etc, can have file systems. EBS volumes are placed in the same AZ where EC2 instances created and are automatically replicated against any failures. By replication is auto replicated in the same AZ. EBS is billed based on GB/month and some type of EBS also are billed based IOPs (input/output operations per sec). Even if EBS volumes are shutdown ie EC2 is shutdown , the EBS will be built on no of GB allocated. Also EBS can be attached to EC2 instances anytime and snapshots can be taken. EBS can support max of 64,000 IOPS or 1000mib/secs per volume and 80,000 IOPs or 2,375 MB/sec per instance  

There are 5 kind of EBS

1. **General purpose SSD (****Solid State Drive****)(GP2) :** good balance of performance and price. 3 IOPS per gb. This is fixed ratio. Fixed ration means the IOPs depends on the size low size doesn’t mean faster IOPs, you can max tops based more size. The ration between IOPs to size is 3:1. It has a min of 100 IOPS and max of 16000 which is 16kb

![Screen_Shot_2020-10-21_at_10-17-58_AM.png](image/Screen_Shot_2020-10-21_at_10-17-58_AM.png)

1. **Provisioned IOPS SSD (IO1) :** designed for io intensive ops like large relational db and nosql db . Unlike GP2 its performance is not fixed ratio with size. Even with low volumes high IOPs can be reached. The ratio here can be maximum 50:1, i.e max of 60,000 IOPs as long aeration is 50:1. IO1 for exam should be any reqm where low volume, high IOPs is required, any high IOPs is mentioned , the default should be IO1. IO1 supports multi attach i.e one EBS IO1 volume can be attache to max of 16 EC2 instances

![Screen_Shot_2020-10-21_at_10-28-08_AM.png](image/Screen_Shot_2020-10-21_at_10-28-08_AM.png)

1. **Throughput optimized HDD (ST1) :**  Meant for data which are sequential. They cannot be boot volumes. Unlike SSD which is good for random access, HDD is good for sequential data processing , mostly used for high processing of data in and out in sequence and with high capacity. Eg for big data, data warehousing log processing. The throughput is 40mb/s to 250mb/s per TiB. Max is 500 MB/s for 2 TB data

1. **Cold hard disk (SC1) :** lowest cost storage for for infrequent access like for files (which you don’t want in S3). Cannot be boot volumes as well. Similar to St1 but only meant for data which is not frequently accessed. Speed is 12 mb/s to 80mb/s per TiB. Max is 250MB/s
2. **Magnetic std :** lowest cost for bootable volume. Should be used for apps which are accessed infrequently . Sometimes option 4 and 5 are grouped together and considered one

The SSD based EBS types like GP2, and IO1 are meant for use cases with high IO. The ST1 and SC1 which HDD based are meant for high througput. The HDD are not meant for boot volumes and they are very cheap with very high capacity, but HDD are based on hard disks so they are meant for high volume so does not perform well for low volume 

EBS Volume cannot be shared between EC2 instances. For that EFS (Elastic file storage) can be used

Few key things to know for EC2

** While creating EC2 instances , you can choose Subnets. Subnets cannot span multiple AZ

** basic CloudWatch monitoring is only every **5 minutes** , it pings every 5 minutes. There is system checks which checks the underlying hypervisor and

there is instance checks which checks the instances. There is detailed monitoring which will ping every 1 min, for that have to pay

** the root volume that comes as default AMI cannot be encrypted but we can add more SSD or HDD, magnetic volumes and encrypt those. If we create our own AMI we can encrypt the root Volume

** the default root volume is in the same AZ and  is deleted when the instance is terminated. This can be unchecked and set to do not delete when instance is terminated

**Instance Store**

Instance store unlike EBS is not network based storage but its local EC2 host storage and not persistent. It is physically attached to the EC2 host, only instances on that host can access it. If the Host is terminated the storage is lost. It has the highest storage performance in AWS more than any EBS backed storage. Instance store has to be attached to EC2 only during startup. Instance store volumes are ephemeral. If instances are moved from one host to another host may be due to stop/start the instance store is lost. Not all EC2 supports instance store, which will be mentioned as EBS Only. Local store provides very high IOPs and throughput , for e.g D2 instances provide 3.5 GB/s read to 3.1 GB/s writes . Snapshots cannot be taken

**Volumes n Snapshots n AMIs**

- Volumes exist on EBS and are basically virtual hard disks. The backup of volume snapshots are in S3 internally. Volumes should be in the same AZ
- Volumes can be modified on the fly, except standard which are magnetic. There is no downtime when changing volumes
- You can move a EBS volume from one AZ to another. To do that , take a snapshot and then from the snapshot view , create volume to another AZ. While doing that the volume type, size can be changed and also encrypted
- Volume snapshots can be copied to other regions. Snapshots which are encrypted are also encrypted in another region
- To copy EC2 instances  from one region to another, first a snapshot needs to be created of the root volume, from the snapshot create an image and then use the image to create another Instance in another region. In this case if the snapshot while converting to image was encrypted, then all new instances with the image have root volumes as encrypted. Its not nesessary to stop the instance to take a snapshot, but to encrypt it is necessary
- Remember, only the root volume is terminated when instance is terminated, the other volumes remain which has be to manually deleted
- Snapshots are incremental, I.e only the changes from the previous snapshots are saved in to the new snapshot in s3. Snapshots can be taken when both running and stopped. Best practice is to take a Snapshot when the machine is stopped. The first snapshot creates a snapshot of the all used data, subsequent snapshots are incremental
- Snapshots when restored in another Ec2 instance data is fetched lazily hence initialization n data retrieval is slow. The is FSR Fast Snapshot retreival which is immediate retreival which is quite fast
- One can save Max of **50 snapshots** per region 
- AMI can be created from snapshots. The process of creating an image of the snapshot is equivalent to creating an AMI.Snapshots of volumes can be shared publicly or with another account provided they are not encrypted
- AMIs can be created from EBS backed storage like mentioned above or **Instance store** which are backed by templates which are stored in S3
- AMIs are regional and instances created from AMIs in one region will only be allowed to create instances in that region. Though you can run that in any AZ. It can be copied between regions and in that case the snapshots in S3 are also copied
- The default permission of AMI is for only the account. One can change the permission to make it public, or accessible to other accounts
- AMIs are immutable. 
- AMIs are billed based on the storage of the snapshots
- The root device volume can be encrypted (which was not allowed earlier). To convert an non encrypted volume to encrypted , one has to create a snapshot and then make a copy of the snapshot , which coping one can change it to encrypted 

**EBS Encryption**

- EBS volumes are not encrypted by default, so they are stored in same AZ in plain text.
- When set to encryption, EBS uses KMS - CMK (Customer managed key). The default is AWS managed keys , but one can also use customer managed keys.
- The architecture is 
    - When the data need to be encrypted, CMK creates a DEK (data encryption key) per volume and that is used to enc/decrypt the data.
    - The encrypted DEK is stored along with the encrypted data
    - To do the actual crypto work, the Encrypted DEK is decrypted b yKMS and the plain DEK is stored in memory in the EC2 instance host, and its work of tiger EC2 host to use that key to do the enc/decrypt of data between EC2 and EBS volume
    - If a snapshot I created of the volume or a new instance is created from the snapshot , the same encrypted DEK is stored with the data
    - **NOTE** the actual crypto work is done by the EC2 host and not the oS, the OS does not know the volume data need to be encrypted. Hence it is much faster.
    - There are places where full disk encryption is needed , in that case that happens in the OS and there will be perf impact

**Instance Status Checks**

 There are 2 status checks done on EC2 instances

- System Status : may be low of sys per or network or software or hardware issue on the Ec2 host
- Instance Status : corrupt files sys, incorrect network settings

Both need to be success for instance to be success. There is auto recovery settings which can be set so if there are status check issues, the instance can be moved to another Ec2 host 

**Instance Bootstrap�**�

Instance bootstrap user data can be passed which will run on instance launch. Important data or password should not be sent. If bootstrap commands run successfully then EC2 status will be success else it will be config error. Max can be 16 kb, for more than 16 KB , a bootstrap file should be used. **Bootstrapping and AMI baking** can be used to improve the boot time . AMI baking is used when we need certain installations to be part of the AMI but do not want to create a separate AMI, with that all the installations will be part of AMI and during bootstrapping we can configure the minimum stuff

**EC2 hibernation**

Instead of stopping or terminating instances one can also hibernate an instance. On hibernation the contents of the RAM is saved in EBS, hence on restarting from hibernation it is much faster because unlike restarting from stopped state , the OS and other programs are not loaded from scratch

**EC2 roles and Profile**

When a role is assigned to an EC2 instance, actually internally a InstanceProfile is created which is attached to the instance which takes care of generating the STS temp. token and setting fit up in the instance metadata. The EC2 and STS takes care of creating a fresh token from STS

**SSM Parameter Store**

If we need to pass password to EC2 instances, user data/bootstrap is a very bad practice, since anyone with ssh rights can see it . A better option is Parameter store , its a secure store used for storing configuration and secrets. The datatypes for the Param store can be String, StringList and SecureString . One can use Param store to store config in hierarchical structure and also maintain versioning . It can store plaintext or cipher text where it integrates with KMS to encrypt the text. You can also use public Param stores to pass params to public AMIs. Since param store is a managed service to access it you need the IAM permissions 

**Auto Scaling Group**

Major config of auto scaling group consists of 

- Group : A group consists of Db Servers, Web Servers
- Template : The script for creating new instances
- scaling options : when to scale, based on cpu, memory , disk usage, desired instances, maximum instances etc

There are 5 Different scaling options

- Maintain current instance level : say always maintain 5 instances
- Scale manually
- Scale based on schedule
- scale based on demand
- predictive scaling

RAID : Redundant Array of independent disks. Its like multiple disks which act like one disk to the OS

    RAID 0: striped, non redundancy (if one disk fails the entire volume fails), very good performance

    RAID 1 : mirrored, high redundancy

    RAID 5: good for reads, bad for writes. Not at all recommended by AWS

    RAID 10 : benefits of both RAID 0 and 1, high redundancy and performance

From was perspective if your need is not satisfied by single volume then combine multiple volumes to create one. If the need is to increase the disk IO then create a RAID.

To take a snapshot of the RAID array volume, we need to stop the EC2 instance and then take a snapshot or freeze all IO operations and flush the data to the disk

**Elastic File System EFS**

Its the elastic file system for EC2 instances. It will grow or shrink based on the number of files in the EC2 instance. Unlike EBS volumes EFS volumes can be mounted to two or more EC2 instances, something like shared access network SAN. Used the Network File System protocol . It is stored across multiple AZs and can support thousands of concurrent NFS connections. Its a block based storage like EBS

Once the EFS is created where you can choose the az of the EFS and the security group. The EFS can be mounted via api from running instances using the metadata api to point to one directory in the EC2 instance. If the same EFS instance is mounted in another EC2 , the 2 instances can share/update the same file system

It is possible to add Access control of EFS directories and files

EFS also provides file sync services which can be used to sync files from one machine or other cloud providers to EFS

**Placement Groups**

Cluster Placement Groups : grouping of instances in the same AZ only. Can span VPCs using peering but both VPS need to be in same AZ. Meant for applications with very low latency and high throughput, this is form the highest form of performance. AWS will try to place the instances in PG in the same AZ, Rack and Even EC2 host. To make the best use of Clustered PG, it makes sense to provision all instances in same PG together, if not done there are chances that capacity is not there in the same Host or Rack. Not meant for High resilience, DR, or HA. Not all instances are supported under Cluster PG. 

Spread Placement Groups : Max amount of resilience and availability.spreading instances across different AZ, racks, diff network, different power source. Ideal for applications which need to keep instances away from each other. Hard limit **7 instances per AZ**. i.e if you have 2 AZ and your instance count is 15, at least one instance will share the hardware rack with another instance They can still be on the same AZ but it will be on separate hardware. Not supported for dedicated EC2 instances

Partitioned Placement groups : Are similar to spread, but uses the concepts of partitions, this should be used when there is requirement to have more than 7 instances per AZ. Each partition can have multiple EC2 instances but each partition will be separated from other partitions by rack, power, network, AZ etc. there still a max 7 partition per AZ. But each partition can have many instances. With Partition PG one can choose which instances will be in which partition and has access to the physical location Partition PG is ideal for huge load and need more control to place in which hardware

![Screen_Shot_2020-10-22_at_2-22-06_PM.png](image/Screen_Shot_2020-10-22_at_2-22-06_PM.png)

[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)
