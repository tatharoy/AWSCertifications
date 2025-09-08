# AWS Databases & Cache

**Databases on EC2**

Only in the case of 

- Access to DB instance OS
- DB option tuning using DB Root access, mostly in the case of vendor demands
- DB or DB version which is not supported by AWS like DB2.
- Some kind of replication/resilience is not supported

Problems

- Admin overhead
- Version upgrade leads to downtime
- DR, taking snapshots, replicas 
- Not easy to scale down , some RDS has inbuilt scaling 

**AWS Relational Database Service (RDS): OLTP Online transaction processing**

RDS is a databaseServer as a Service and not Database as a Service DBaaS. The difference is when we provision DB via RDS, we get a database server which can host 1 or more databases . With DBaaS you will only get one database instance. 

**MS SQL Server**

**Oracle**

**MySql**

**PostgresSQL**

**MariaDb**

**Aurora (Architecture is different)**

**A typical RDS DB architecture (Not Aurora)**

**![Screen_Shot_2020-10-25_at_1-32-15_PM.png](image/Screen_Shot_2020-10-25_at_1-32-15_PM.png)**

- RDS DB instances like EC2 comes in various families like EC2, though they are prefixed as db. , eg db.m5, db.r5 (memory optimized) , db.t3 etc
- RDS is billed for CPU usage and also data stored GB/m. It is billed for storage allocated and not based on usage. So 100 GB for month and 200 GB for half month is same. If the underlying EBS is IO1  then one is also charged for IOPS 
- RDS instance is only pointed via CNAME 
- When setting up Relational DB , two important concepts are multiple AZ and Read replicas (Max 5)
- First thing to consider before backup is **RPO** (Recovery point objective) and **RTO** (Recovery time objective) : For backups architecting and exam try to find the RPO and RTO
    - RPO : The time between the last backup and Disaster, basically maximum dataloss. The less data loss appetite the frequent we need to schedule backups and it will be more costly
    - RTO : the time taken between the disaster and full recovery. Again the lower time taken to recover is more expensive . RTO value can be reduced by having good plans, backup devices, good documentation n team and proper on time alerting and action
- Two types of backups  i) automated backup and ii) manual snapshots. Both are saved n S3 bout AWS managed S3 so cannot be seen in account S3 bucket and available in multi AZ
- **Automated Backup :**
    - Automated backups are enabled by default.??
    - These backups are stored in S3 where the same storage space is allocated in S3 as per max Db storage.
    - These backups work at particular time the day during which time I/O may be suspended.
    - Automated backups can be configured for retention period **0 to 34** days, 0 means no backups. Automated backups allow db to be recovered to any point in time in that retention period.
    - Automated backups are deleted once the instance is deleted
    - RDS also stores the DB transaction logs every **5 minutes,**  this is really helpful during restore
    - If DB is encrypted , the backup, snapshots , read replicas are encrypted.
- **DB Snapshots :**
    - Are manual process and needs human trigger. Snapshots are not deleted if db is deleted. The snapshots are also saved in S3
    - Snapshots like EBS are incremental. 
    - During Snapshot there is a interruption, so if single AZ that may be a problem but in multi AZ since the backup is from standby here is no interruption 
    - The snapshots are for the db server store, so if there are many Db instances all instances will be taken backup for 
    - When DB is deleted, a final DB Snapshot is requested from the user.
- **Restore :�**�
    - Restore sets up a new DB instance server with new endpoint though the authentication remains the same.
    - The application should be updated to point to the new RDS address
    - The restore first uses the last manual snapshots or last auto backup and then runs the transaction logs till the direct point in time 
    - Restore time is slow and impacts RTO
- All db types except **Postgres supports encryption using AWS KMS.**
- At present aws does not support encrypting an existing db. To encrypt that migrate it to a new db with encryption enabled
- RDS now support storage auto scaling for most databases in many regions
- Multi AZ
    - Multi -AZ replication is used for disaster recovery and not for improved performance, or extra capacity.
    - If a db is enabled with AZ, AWS will take care to copy your data from one AZ to another. This is sync replication 
    - In case of failure to original db , AWS RDS automatically updated the CNAME to the standby db with no manual intervention and recovery time. there is also no need to change the hostname of the db. Its available for all RDS databases.
    - Standby instances cannot be access directly 
    - The AZs cannot be in other Regions
    - During the shift to standby there may be **60 to 120 secs** of disruption for the client, so Multi AZ does not support fault tolerance
    - Multi AZ is not free and not available in free tier 
    - In Multi AZ backups happen from the standby and not primary, thereby improves perf during backup
    - Aurora by default has multi AZ
- Read Replica:
    - Read replicas is used for performance improvement and also provide HA 
    - They are read only dbs.
    - In that ec2 instances can point to different read replicas of the same db, as each replica has its own dns route.
    - There can be upto **5 replicas** of one db instance.
    - Read replicas are created by async replication from the primary db instance. So there can be a lag between the replication. 
    - Read replicas are only supported by **mysql, Maria and Postgres**, and only **Maria and mysql supports read replicas in another region and zone**.
    -  Automatic backup of primary db need to be set up for read replicas. 
    - Read replicas do not support auto repointing in the case of failure of primary, one has to manually promote a RR into RW. This improves the RTO 

**Aurora:**

- Aurora architecture is different than regular RDS, it has cluster where there is one instance and multiple replicas, these replicas provide both scalability and HA. Unlike RDS there are no 2 modes like AZ with active/passive for DR or Read replicas. Moreover Aurora underlying storage is shared between all instances within a cluster. 
- Regular RDS we talk about DB Instance Server, in Aurora we talk about Cluster 

![Screen_Shot_2020-10-25_at_9-33-48_PM.png](image/Screen_Shot_2020-10-25_at_9-33-48_PM.png)

- Aurora has a Primary instance and **1** to **15** replicas. The best part is the underlying SSD storage (High IOPs) has 6 replicas so at a time data is stored in atleast 6 stores. So the primary or any of the replicas can read from any of the stores and the primary writes to one store, the greatly improves RTO since highly unlikes all the stores will be down at a time. During failover any of the replicas will be auto promoted to primary. 
- Storage is max **64TiB**. Billing is based on consumed data and not data provisioned unlike other RDS, **no free tier** 
- Aurora comes with multiple dns entries one for primary read/write and another endpoint for read only which can load balance between all read instances.
- Backups work the same way as RDS. Restores create a new Cluster 
- Aurora provides a new feature called backtrack, where one can backtrack to a certain point in time if the current data is corrupt. 
- Aurora is mysql compatible rdbms engine. Its **5 times better performance** than mysql and **1/10th of a cost**
- Aurora auto scales from initial **10Gb to increments of 10G until 64Tb.**
- **Aurora** also supports multi write cluster, where there can be more than one write servers . The default is singe master. The architecture for multi master is same at storage level just there is no cluster or read endpoints, applications are supposed to point to individual instance endpoints. This can be called fault tolerant application   

**Aurora Serverless**

- Has the same relation as ECS Fargate. Takes away the Admin overhead.
- Has the same concept as Cluster but internally uses **ACUs ,** Aurora Capacity Unit  which are allocate based on the capacity used . Though there re MIN or MA ACus. ACU can go to 0 

**Aurora Global Database**

- Allows global level replication from a master region to a max 5 region globally
- Replication is in 1 sec. so low RPO and RTO 

![Screen_Shot_2020-10-25_at_10-40-55_PM.png](image/Screen_Shot_2020-10-25_at_10-40-55_PM.png)

**Dynamo DB**

DDB is a fast and reliable no Sql, wide column public database as a service, supports single digit millisecond latency at any scale , it is a fully managed database, supports both key value and document data models. Unlike RDS, DDB provides database as a service and not db server, hence it has much less admin overhead. Provides backup, point in time recovery and encryption at rest

**DynamoDB Table Structure**

**![Screen_Shot_2020-10-26_at_9-46-18_AM.png](image/Screen_Shot_2020-10-26_at_9-46-18_AM.png)**

In DDB, each row is called items. All items in a table should share the same PK (Partition key) or Composite key (Partition Key and Sort Key). The rest of the attributes can be different, none, all , mixture i.e no rigid schema for the rest of the attributes , each item can have max of **400KB.** In DDB, one can set capacity but unlike other services capacity here means speed. If one sets 1 WCU Write capacity unit - means 1 KB data can be written in 1 sec, 1 RCU means 4 KB data can be read per sec. RCU and WCu means units of data read or written to DDB store and does not mean the data which is returned from queries. DDB is turely  demand DB where its billing only on use, on capacity and no infra. Also supports reserved capacity which is cheaper than on demand capacity

- OnDemand and Reserved capacity :
    - On Demand : used when capacity reqs unknown , unpredictable. Its charged based on per million read or writes. Charges are usually 5X times than provisioned
    - Provisioned : RCU and WCU set on each table
- Retrieve Data 
    - Query:  when querying, one can retrieve values based on the partition key or partition key and sort key. If the returned data is in 4 KB , 1 RCU is utilized
    - Scan : its used when querying based on other attributes and not PK or CK. It scans through entire table and based on the items size the capacity is utilized so scans uses a lot of capacity    
- Its always stored in SSD storage, unlike redis which in memory
- Default available across 3 different geographically AZs in the same region. The data is stored in Storage Node in each AZ
- Of the 3 AZ, one is a leader where all write operations happen
- Supports both eventual consistent and strong consistency reads: consistency across all copies is eventually reached in 1 sec and strongly consistent reads : it basically returns data after the latest write of the data. Eventual is cheaper because it can go to any of the AZ storage nodes but on certain conditions and time it can pick stale data when the record has not been persisted in the read node.

![Screen_Shot_2020-10-26_at_12-46-44_PM.png](image/Screen_Shot_2020-10-26_at_12-46-44_PM.png)

- It supports push button scaling so, there is no downtime to scale and can be done from the console without taking
- The pricing is based on storage and reads/swrites , .its 25 cents per 3 GB data and .0065 pe**r**
- **Backups:**
    - **On Demand :**  
        - is similar to manual Snapshots in RDS. 
        - Maintains a full copy of the table until removed**,** one has to manually remove backups
        - With backups can , restore the table in same or another region, the restore can be with or without index and can be encrypted
    - **Point in Time recovery :**
        - Not enabled by default 
        - Once enabled an automated backup is created everyday for a stream of 35 days. So it can be restored to any day within the last 35 days
- Global tables supports multi region replication for HA and DR. Global tables has to support streams which are time based stream of events for a DynamoDb Table 

**DynamoDb Accelerator DAX :�**� Its a fully managed memory cache for dynamoDb, 10x perf improvement. There is no change in the API when introducing DAX. Dax supports transaction

![a4f8f95bfb0bba9d5123486c13aa69da.png](image/a4f8f95bfb0bba9d5123486c13aa69da.png)

**ElastiCache**

In memory cache, supports both memcached and redis

![98cce30b942ced1c5241da833f83faf2.png](image/98cce30b942ced1c5241da833f83faf2.png)
