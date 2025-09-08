# AWS S3

**AWS S3**

**Simple Storage Service**:

-  An global , durable , secure object storage service, only meant for objects like files and not for database or OS which needs block level storage.
-  It is stored in one region but bucket names need to be globally unique. There are no limits to the number of objects within a bucket. 
- There is no concept of a separate folder type in S3. If the object is stored under a folder like /folder1/file.jpeg, the key of the object is /folder1/file.jpeg and there is separate object folder1 is stored. Only when it is shown in UI, the folder is shown as folder. Folders are therefore called prefix
- Bucket names should be lowercase with no _ and 3 to 63 characters. Should start with lower case character to number , cannot be formatted as IP address. Soft Limit of **100** buckets per account , hard limit of **1000 buckets**
- S3 is not file or block storage so cannot be mounted as drives or be used as NFS 

**An S3 object consists of**

    Key : globally unique identifiable file name for buckets. Unique identifier for the object within the bucket

    Value : the actual file, byte

    id :    id incase of version, important for versioning. Id is null when versioning is disabled 

    Metadata:

    Access Control List : ACLs to secure the buckets

The durability of s3 is 99.999999999 % and availibility is 99.99

The file path of a bucket will be [https://bucketname.s3](http://s3.region/).[amazonaws.com/](http://amazonaws.com/bucketname) - The Default region, West Virginia , US East 1

[https://bucketname.region](http://s3.region/).[amazonaws.com/](http://amazonaws.com/bucketname) for other regions

**S3 Data consistency**

Read after write consistency : for PUT of new objects : one can read the file immediately after one adds that

Eventual consistency for put and delete for old object : it will take some time to see the changes after a file is updated or deleted. This is valid for S3 object which has versioning

**S3 Pricing**

 S3 can be charged for 

- Storage as per GB / per month
- Data transfer per GB per month 
- Request and data retreival i.e when we use the api to list files etc 

**Types of S3**

|**Type**                |**Description**                                                                                                                                                                                                                                                             |**Storage Charges**             |**Transfer Charges**                                                                                                                                                 |**Request charges**   |**Latency**                                                                             |Others                                                                                               |
|------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------|----------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
|**S3 Std**              |0 bytes to max 5 TB. By default saved in at least 3 AZs. 99.999999999% durability i.e for 10,000,000 objects 1 object loss in 10, 000 years. Should be used for important and irreplaceable data and need frequent access                                                   |GB/month charges                |A $ charge for each GB transferred out. Transfer in is free. There is no retrieval fee                                                                               |Price per 1000 request|Data is available immediately in milliseconds and publicly available                    |No minimum duration, charged based on time its stored                                                |
|**S3 Std-IA**           |

Not suitable for objects that need frequent access, are tiny sizes and need to be stored for less than 30 days and is replaceable
0 bytes to max 5 TB. By default saved in at least 3 AZs. 99.999999999% durability i.e for 10,000,000 objects 1 object loss in 10, 000 years|Storage is much cheaper than std|Along with transfer there is high retreival fee                                                                                                                      |                      |                                                                                        |

Minimum capacity of 128 KB charged
Minimum duration of 30 days charged. Even if stored for less days. |
|**S3 One Zone IA**      |

Shouldn’t be used for irreplaceable objects
Stored only on one Zone, data loss possibility. It still provides the same durability , data is replicated but only on the same AZ,                                                                                              |                                |Same retreival fee                                                                                                                                                   |                      |                                                                                        |Same as Above                                                                                        |
|S3 Glacier              |

Should be used for archival needs, like RDS backup 
Same durability and AZ for S3 Std . Data is retrieved to S3 IA temporarily . Data is stored in cold storage                                                                                                              |Storagecost of 1/5 of S3 std    |- Expedited : 1- 3 mins
- Standard : 3 to 5 hours
- Bulk : 5 to 12 hours

 The faster the retreival its more expensive 
Retreival process is a backend job. 3 kind of jobs|                      |

First byte latency can be minutes or hours
Not available immediately and cannot be public|90 day minimum billable storage and 40 kb minimum billable size                                      |
|S3 Glacier deep archive |Cheapest storage . Data is stored in frozen state. Available in 3 zones. For regulatory documents                                                                                                                                                                           |1/4 of price of glacier         |Bulk : is 48 hours
Std retreival is 12 hours                                                                                                                          |                      |                                                                                        |180 day minimum storage and 40 kb minimum size                                                       |
|S3 Intelligent tiering  |Data is stored in possible 2 layers: Frequent and infrequent access tiers. Based on use it will be stored in frequent or infrequent layers . If not accessed for 30 days move to infrequent                                                                                 |                                |No Retreival fee                                                                                                                                                     |                      |                                                                                        |Minimum 30 day duration                                                                              |

S3 IA One Zone : For infrequent access and only in one zone

S3 - Intelligent Tiering : Uses ML to determine which tier the object should go to

(NOt from 2018)RRS : reduced redundancy storage :Its has less durability guarantees. Can be used for files which can be regenerated. This is not there post 2018

Glacier: for Archival, return type can be configurable. Pricing can be 0.01$ per GB per month

Deep Glacier : the cheapest archival, return time is fixed at 12 hours

Data stored in S3 can be charged for

- Storage

- Access

- Movement (from one region to another)

- Storage management pricing (an object can be stored with tags which can incur different pricing)

- Transfer Acceleration : This takes into account the nearby edge location.So if Transfer Acc is enabled , then the files are not directly stored/retrieved in S3 but the edge location. The edge location later send the file to S3 in the region

Each folder/file object in S3 has the following properties which can be changed

    Permissions : can be granted public and or resource policy.

    Storage type: default its regular storage , but can be changed to Infrequent use or glacier .

    Encryption: Can have no encryption , or AES Server side encryption which is AWS S3 managed keys, or AWS KMS or Customer managed keys SSE.

    Metadata : used defined key- value pairs or http system headers. The metadata of bucket is not inherited by the child objects.

    NOTE: the minimum size for S3 standard is 0KB. For IA its 128KB

BUCKETS CAN BE access controlled by Access control or by bucket policies. 

By default buckets and files within it are private

**S3 Versions:**

Version can be enabled for the bucket. Once enabled it cannot be disabled. It can be suspended

Objects without version are uniquely identified by key. With versioning its identified by key and id

When an object with versioning is deleted , it doesn’t delete but puts a delete marker on all the versions. To permanently delete we need to provide the id

S3 maintains all the versions of the object. If FileA is uploaded with 18KB and its again uploaded. Then the version 2 will have 32 or more KB since it contains details about version 1 too .

Adding a new version makes the new version private again, if the old version was private

Versioning supports Multifactor authentication for change in version or delete versions. In case of deleting version via APIs one need to pass the MFA serial number + MFA code to the API calls

**S3 Replication:**

There 2 types of replication **CRR** (Cross region replication) and **SRR** Same region Replication. Both supports replication in same or another account. In case of another account, IAM Role needs to be assigned to source bucket to assume role of destination bucket

With this contents of a bucket in one region can be copied/replicated in another region. Versioning needs to be allowed for CCR to function

To enable this , a bucket needs to be created in another region. The source bucket can be configured from the management tab , replication button . You can choose to replicate **the entire bucket or only specific prefix/tags**. One can also choose a s separate storage class in destination. One can also set Replication Time Control or **RTC** to provide replication SLAs of 15 minutes to replicate , if not set there is no guarantees on the time taken to replicate . Its only one way replication from source to destination. If objects added manually to destination that is not replicated to source. Only replicates objects which are unencrypted, SSE-S3 and SSE-KMS, does not support SSE-C since it does not have the key to replicate the encrypted object. Also Glacier and Glacier Deep Archive cannot be replicated. Deletes are not replicated 

The destination should have versioning enabled if source has too. It does not copy existing objects to the destination bucket but only new or updated objects. The existing objects should be manually deleted

In most cases CRR is used for backup, so it makes sense to change S3 object type to IA

At this time, objects cannot be replicated to more than one bucket. And source n destination bucket should be in separate regions

To copy existing objects from source to dest bucket, it has t be done programmatically, using AWS cli or any other sdk. By default permissions are are coped, so if one object is public in source it is still private in dest. The delete/updates for versions are not copied

**Life cycle management:**

Object can be move to IA Std or IA One Zone only after **30 days** in std storage and move to glacier after another **30 days** after being in IA, this limitation is only in the case of one rule. In case of multiple rules object can be moved to glacier from Std S3. Objects cannot be permanently deleted before 61 days because the objects need to be moved to IA and Glacier before that. Life cycle is done though a set of rules which can be applied to either a bucket or a set of objects with a tag or metadata. Life Cycle actions can be i) Transition Action : rules to move objects from one storage type to another or II) expiration Action: delete objects or versions of objects after a stipulated days 

Objects to be moved to IA should atlas be 128KB size

If IA is not enabled, objects can be moved to glacier 1 day after it has been created

On version objects expire makes the current version as previous version and puts a delete marker on the current version

**S3 Performance:**

S3 upload relies on single stream upload, S3:PutObject API, which is atomic in nature. Its a simple process but can have performance impact and if the network fails after 4.5 gb of upload of a 5 GB file, the whole file needs to be uploaded again. Enable Multipart requests for large files upload (Similar to BitTorrent), recommended for files over **100 MB** and each part but be **5 MB -> 5 GB** and there can be max of **10000** parts. Parts can individually fail and also restarted . Transfer rate of Multi part transfer is much faster.

Transfer Acceleration using CDN or edge locations. To enable bucket for transfer acceleration the bucket names should be dns compatible and cannot have dots in the name 

 S3 prefix is the folder names before a file, for e.g if you have a file bucket/folder1/folder2/text.txt, then the S3 prefix for the file is /folder1/folder2

S3 has extreme low latency , 3k requests per PUT/DELETE and 5K request per GET per prefix

If SSE:KMS is used, it will impact the performance , for the obvious reason. Since the KMS API is going to be called

S3 Byte Range fetchs can be used for downloading large files

**S3 Select :�**�

SQL can be used to retrieve data instead of looping through all files and folders 

**Securing Buckets**

Buckets can be secured by access control lists (ACLs), bucket policies, access point policies, or all

i) Bucket policies : these are policies only for the bucket. Its kind of a resource policy. Bucket policies can be used to give accesss to any identity within the account or otherwise and also to anonymous users. Along with other typical resource policies, bucket policy can be used to deny particular ip addresses 

ii) Access control lists (not used much):this can be applied to the bucket as well as to the objects within it. Its legacy and not recommened to use

**Block Public Access**

This was introduced to stop all public access ie users who are anonymous and do not have AWS identities. This is final fail safe security. There are different settings for block public access 

- Block all public access , no matter what resource policy says
- Blocks new access but allows any existing access which were allowed using ACLs before
- Blocks all public access given by ACLs whether before or after the block public was applied
- Allows access through bucket or access point polices for existing buckets but not for new ones
- Blocks all access through bucket and access point policies before or after the policy was applied

 

**Encryption:**

In S3 buckets are not encrypted but the objects are

**SSE:C Customer provided**. The customer provides the keys which are uploaded. S3 takes care of actual Enc/Decrypting the keys using your keys. The management of keys is entirely left up to customer. There is management overhead but customer has more control. When encrypting, one needs to pass the object to be encrypted and the key to S3. S3 discards the key once encryption is done, but stores the hash of the key with the encrypted object. To decrypt the same process but first S3 compares the stored hash with the hash of the passed key. This is usually used for regulation have env 

**SSE:S3** (S3 managed keys, unique using strong multi factor envy. The key is also encrypted using the master key. The en is AES-256). There is key generated for each object, which is used to encrypt the object and then an encrypted key is generated which is stored with the enc object, the original key is discarded. One has no control over the key generated but no admin overhead. Not suitable for compliance env where key management, key rotation is required. Also no separate role separation, so S3 does both key management and the enc/decrypt process

**SSE:KMS** (Key management Service) : similar like S3 with extra features like audit trail as to when the key was used, use your own CMK or AWS managed CMK. As seen on the KMS Section. The master key is in KMS but the actual enc/decrpt process is done in S3, where KMS generates a DEK (data encryption key) for every object which is used to encrypt the object in S3. The difference between SSE:S3 and SSE:KMS is where the Master key is (CMK in case of KMS) so there is role separation and also rotation policies can be set

Client side Encryption

**Static Website Hosting**

Usually S3 apis are used to access S3 object. S3 can be used for static page hosting which can be used using http. With S3 for static page, we need to point both index.html and error.html. though static page hosting is enabled we still need to set Resource policy and unblock public access

If Route 53 is used with S3 for creating web pages, the bucket name should be same as the domain name. For e.g. if by website is [tatha.com](http://tatha.com/) , then bucket name should be tatha.com. in R53 one needs to update the A record to point to alias instead of IP address

The website url (if not using Route 53) is [https://bucketname.s3-website-region.amazonaws.com](https://bucketname.s3-website-region.amazonaws.com/)
