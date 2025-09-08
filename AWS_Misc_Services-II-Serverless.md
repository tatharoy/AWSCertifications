# AWS Misc Services - II - Serverless

**Lambda�**�

AWS Lambda is AWS introduction to ‘serverless’ architecture. It is Function as a Service offering,

-  it is only billed for the duration of the function running time . 
- When a lambda function is created , it needs a runtime (Java, Python, Node etc) and a memory is allocated. The more a memory is allocated , more CPU is allocated and it costs more to run the lambda function. 
- An Execution role is also assigned to the function , this role has the permissions to access other services as needed. 
- A Lambda function may be invoked via other AWS services like EC2 etc , event driven , scheduled or directly from customers via API gateway. 
- A lambda func by default runs outside a VPC which has access to internet. A lambda function if configured inside a VPC , the VPC needs to be configured to access internet if lambda function needs internet access  
- A L function has a max runtime limit of **15 minutes.** It cannot run more than that
- Lambda comes under free tier with **1M free requests per month and 400000 GB** secs per month** , i.e one can run 1 GB memory for 400000 secs per month for free**
- Check EventBridge in CLoudWatch section. The Lambda function can be triggered by EventBridge rules 

![Screen_Shot_2020-10-23_at_8-58-30_PM.png](image/Screen_Shot_2020-10-23_at_8-58-30_PM.png)

**SQS  Simple queue service**

- The oldest aws managed service. a distributed Message queue system. Its in public space so internet applications with proper permissions can access it. Queues are used for decoupling 
- SQS is a pull based system, poll based. messages in sqs can contain max of **256 KB**. (Have to confirm on the max size of messages)
- There are two kind of queues
    - Standard : there is no guarantee of order of messages and they guarantee **atleast one time** delivery. Standard queue tries to maintain best effort ordering but there is no guarantee of them during high load. This is used for high volume
    - FIFO : they guarantee first in first out hence order is maintained and message is delivered **exactly once** . It limits to only **300 messages per sec, or 3000 messages per batch** . They also support message groups in which each group has separate order based on the same queue

- Supports Dead Letter Queue 
- Auto Scaling group can be used to scale SQS tased on number of messages
- Messages once consumed by clients are not deleted but stays in the queue hidden which is called **visibility timeout.�**�During the visibility timeout no other clients can view the messages. If the consumer does not explicitly deletes the message within the timeout its available to other consumers. This is similar to kafka’s timeout after which the same message is available to other consumers
- Messages in the queue can stay from **1 to max 14 days, default is 4 days.**
- Encryption at REST using KMS
- Access is via Identity Policy and Queue Policy (Resource Policy) which can be used for other accounts
- There is short polling and long polling for the consumers. For the short polling the consumers will continuously poll the queue for messages until one comes. This may increase costs since its billed based on the calls
- Billed on number of requests and on one request **1 to 10** messages will be received with max **64 KB**
- Since SQS is more expensive on the number of calls made, there **2 polling technique, Short and Long (waitTimeSecsPolling, upto 20 secs).**  So if there are less messages Long polling is preferred 

**SNS : Simple notification service**

- SNS runs in the public space so if internet users have the right permissions they can access the sns service. Also cAN BE ACCESSED FROM PRIVATE VPC IF vac has. Rights to access public end points
- Setup,operate and send notifications from the cloud. SNS can deliver messages to https, phone OS, sms , email , to SQS queue and also trigger lambda functions. Messages are **pushed** 
- SNS uses the concept of topics, where each subscriber can dynamically subscribe with the SNS topic and get notified. SNS formats the message based on the subscriber
- Supports message delivery status, delivery retry, supports SSE, cross account access via Topic policy similar to source policy
- All messages in SNS are available in multiple AZ.
- Messages in SNS should be **<= 256KB**
- When a SNS topic is created a URN is created
- 50 cents per first 1 million notifications
- Messages are not persisted 

**SNS and SQS Fan Out**

Fan out is used to replicate the same message in a topic to multiple dedicated queue

![Screen_Shot_2020-10-24_at_12-30-25_PM.png](image/Screen_Shot_2020-10-24_at_12-30-25_PM.png)

In the above case when a object is uploaded in S3 bucket and event is triggered which is sent to SNS. ;To transcode the uploaded video to multiple video size, the message is copied to separate queues for each type of size. This is called Fan out 

**SWS  : simple workflow service**

A service to coordinate work between different applications . The tasks in SWS can stay for 1 year. SWS delivers tasks only once. There are 3 actors for SWF

Starters: the apps or human who started the workflow

Deciders: who controller the flow. They decide when the flow should do after the one task is complete

Activity worker: who does the actual work

**Step Functions**

For executions which follows state , state functions can be used. This can be used for serverless long running workflows. Lambda functions are stateless and can only run for 15 mins max. A State workflow has START - STATES - END. Workflows in Step Function can be standard and express, Standard is used when each state can last time 1 day max. For Express each state can last for max 5 minutes. States can be triggered via API Gateway , IOT Rules, EventBridge , Lambda. Each state can be given a role for permissions . A state can take in input and there can be output for next state

The difference states can be 

- SUCCEED OR FAILED
- WAIT - will wait for some time or a date of time. This is input to a state node and 
- CHOICE - a choice based on state , something like if else. Different set of behavior based on the input
- PARALLEL - parallel branches in a state machine
- MAP - accepts a list of input. For each input , does the action on each input of the map
- TASK -  represents a single unit of work on a state machine. It is the task state that we connect with other AWS services 

**A sample architecture using step functions , lambda,SNS�**�

**![Screen_Shot_2020-10-24_at_12-04-32_PM.png](image/Screen_Shot_2020-10-24_at_12-04-32_PM.png)**

Elastic Transcoder

Converts media file from their original format to the format which will play in media devices like mobile etc. it is charged based on the time taken to transcode and also the format on which it is charged

**API Gateway**

- Scalable and highly available managed api endpoints service
- AWS API Gateway enables API caching which caches the output for a particular response  for a TTL
- Supports HTTP, WebSockets
- Also one can enable CORS in API Gateway, API monetization, API documentation using OpenAPI 3.1
- Billed on the API calls, data transfer and features like caching
- Log the api calls in cloudwatch

**Kinesis**

Kinesis is scalable, public streaming service similar to Kafka streams. All data is stored for 24 hours there are 3 types of kinesis

**Kinesis Streams :** Its consists of shards (just like Kafka partitions) , So source producers can push to kinesis shards which goes to one of the shards. There can be lambda functions or EC2 instances which listens to these shards and does some processing on that

Shards allow **1000 records per second for writes with max data is 1 MB write per second**. **5 transactions** per second reads unto **2M reads per second**

Default retention of data is **24 hours** and can be extended to **7 days**

**Kinesis Firehose:** Firehose is similar to streams, just that the user need not worry about setting up shards. All producers send to the hose which takes care of auto scaling.

The firehose has optional  lambda functions inbuilt but configurable to analyze the data and put in S3 . So basically one need not worry about data consumers in firehose

**Kinesis Analytics:** allow to write sql queries on data in firehose or streams and then store the data in S3 or redshift
