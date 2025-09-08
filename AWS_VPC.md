# AWS VPC

**Virtual Private Cloud**

VPCs are like virtual data centers in the cloud. VPCs lets you provision logically isolated section of AWS where you can launch resources in the virtual network . One has complete control over virtual network environment , selecting IP ranges , subnets and route tables and network gateways. One can create separate public or private subnets for different applications and also provide security groups or Network ACLs for each resource in the subnets. Additionally one can create Hardware Virtual private network (VPN) to connect from company data center to VPC in  AWS public cloud

**VPC Architecture** 

An internet or VPN gateway —> Router —> Route tables —> Network ACL —> public/private subnets. Each subnet has a range of ip addresses and a security groups. Amazon has a set of ip ranges that can be used for subnets 10.0.0.0 - 10.255.255.255 , 172.16.0.0 - 172.31.255.255, 192.168.0.0 - 192.168.255.255. These are all ranges for private network. 

![853a0c8ca23b86b2731b1250b3d01c6e.png](image/853a0c8ca23b86b2731b1250b3d01c6e.png)

- A VPC is for one account and one region, a VPC does not span regions. There is only one default VPC per region, which can be deleted and also recreated. Its not necessary to have a default VPC in a region
- The default VPC is always configured with 172.31.0.0/16. It always has one subnet in each AZ
- In a region one can have 5 VPCs created.It can be increased by mailing AWS.
- There can be only one internet gateway per vpc.
-  A subnet can only exist in one AZ
- Security groups do not span across VPCs. 
- Since security groups span across AZs that means one SG can be applied to multiple subnets  
- Route tables has the configurations to allow one subnet to communicate with another subnet. We can block one subnet to talk to another subnet using route tables.

**Difference between default VPC and custom VPC**

Default are user friendly , they allow to deploy apps immediately. All subnets in default are public have internet facing routes. EC2 instances in default have public and private ip address. In custom the default addresses are private ip addresses

**Peering:**

.

- One can peer one VPC with another using direct network route.
- This can be done with VPCs in one account or with other accounts.
- VPC peering can be done in multiple region (it was only in one region before).
- VPCs do not allow transitive peering I.e if VPC A is peering with both B and C, it does not mean that B and C are peering transitively.
- Peering also doesn’t work if two peers share or has overlaying cidr blocks

**Lab:�**�

- VPC can be created  in dedicated hardware or multi tenancy
- When we create a custom VPC, default route, security groups and Network ACLs are created.No subnets and internet gateways are created . When a VPC is created , this is created

![af9c70d7f65663d4e695d80b6be51c0d.png](image/af9c70d7f65663d4e695d80b6be51c0d.png)

- The subnets and internet gateway need to be created separately.
- While creating subnets we can choose the AZ and also the cdir ip block. It has to be less than or equal to the block assigned to the VPC
- The default route table created for the vpc is associated with the subnets by default. Hence we do not want the main route table to be connected with any internet gateway, otherwise all subnets will be internet accessible . So to connect one or more subnets to the internet, those subnets need to be connected to a custom route table which in turn needs to have inbound routes added with 0.0.0.0./0 and connected to the internet gateway
- There should be one route for one type of subnet, i.e all public facing subnet it should map to the same route

**NAT :�**�

NAT (Network Access Translation) helps to connect instances in private subnet to connect to the internet or other services but does not allow internet to access those machines.  Nat does not provide IPV6 addresses so for that egress-only internet gateway should be used. Very good video on NAT [https://www.youtube.com/watch?v=QBqPzHEDzvo](https://www.youtube.com/watch?v=QBqPzHEDzvo)

There are two kinds of NAT , Instances and Gateways

Instances:

Instances are like any other EC2 instance which acts as a NAT. There are Amazon AMIs which are available to be created in the public subnet. From the private subnet, the main default route table needs to be configured to route 0.0.0.0/0 to the NAT instance instead of the internet gateway. The NAT instance will then route traffic to internet . All EC2 instances by default have source/destination checks which need to be disabled in case of Nat instances . Nat instances have the disadvantage of low availability since it is only in on AZ, it can be configured to be part of a auto scaling group in different AZs but then everything has to be managed etc. it also has low bandwidth

Gateway

Gateways are like any other AWS services which are managed. It just need to be created in the public subnet and no need to configure security groups for it. It can auto scale and also supports 10GBPs bandwidth. They are automatically assigned public ips. Similar to nat instances , in the private subnet the route table need to be routed to the gateway

**Network ACL:�**�

- A custom NACL is by default deny all. Each subnet can be connected to only one NACL.
- A NACL also can only be associated with only one VPC like the internet gateway. A NACL can span AZs .
- The NACL has to be configured for both inbound and outbound routes unlike security groups which are statelfull, NACLs are stateless. Using NACL one can block specific ip addresses . For outbound NACL routes, the ephemeral ports need to be opened.   
- NACLs processes rules based on order , recommended is multiples of 100. For e.g. you can have a rule to allow all inbound calls from the internet 0.0.0.0/0 ALLOW which is rule number 100, if I want to block only from certain ip, it needs to be rule 99 and not 101. i.e rules with lower number are given preference if two rules  collide 

Load balancer in VPC is similar to normal default load balancers just while configuring we need to point to custom VPC and select ATLEAST two subnets. So to make load balancer work , there should be atleast 2 subnets with public ips and connected to internet gateways

**VPC Flow Logs:**

Flow logs enables one to capture all network traffic in your vpc and push that info to cloud watch logs.    One can retrieve the traffic logs from cloud watch logs . The flow logs can be created in vpc level (all network traffic will be captured) , at the subnet level (only logs for the subnet). While configuring flow logs, first the log group needs to be provided for cloud watch and also a role needs to be created for flow log to access cloud watch. Few limitations of flow logs are

It cannot log for peers if they are not in the same account

Once the configuration of a flow log is done it cannot be changed

It does not log traffic from the metadata ip 169.254.169.254

**Bastion vs NAT (Bastion a.k.a Jump Boxes)**

Bastion instances are used to access instances in private subnet for ssh/rdp purposes. We call Bastion as jump boxes. NAT is meant for outgoing traffic for EC2 instances to access internet.

**VPC Endpoints (also called DirectLink)�**� 

VPC endpoints are used for EC2 instances to directly access other aws resources without going out to the internet. It internally uses AWS network.

First is to create a role to access say S3 and assign the role to the instance in private or public subnet. Second step is to create a endpoint for say S3 and assign it to route table for the private subnet. Alternatively in the route table just like we added NAT gateway as destination for 0.0.0.0/0 we can add the Endpoint.

**HA and Best Practices:**

[https://docs.aws.amazon.com/quickstart/latest/vpc/architecture.html](https://docs.aws.amazon.com/quickstart/latest/vpc/architecture.html)

- For each Availability Zone, this Quick Start provisions one public subnet and one private subnet by default.
- ACLs as firewalls to control inbound and outbound traffic at the subnet level. This Quick Start provides an option to create a network ACL protected subnet in each Availability Zone.
- Independent routing tables configured for every private subnet to control the flow of traffic within and outside the Amazon VPC. The public subnets share a single routing table, because they all use the same Internet gateway as the sole route to communicate with the Internet.
