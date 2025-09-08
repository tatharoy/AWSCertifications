# AWS IAM & Security

**AWS Accounts**

One can create multiple root accounts for the same email id by adding ‘+’ to your existing email id like tatharoy1+generalaccnt@gmail.com, where [tatharoy1@gmail.com](mailto:tatharoy1@gmail.com) is my email id with one root account. When an account is created by default a role is added, OrganizationAccountAccessRole. When the root user has to login , the url will be [https://console.aws.amazon.com/](https://console.aws.amazon.com/) whereas when an IAM account is created the user need to access the url of the account which is [https://accountid.signin.aws.amazon.com/console](https://accountid.signin.aws.amazon.com/console) , the url can be customized to update the numeric account id with an alias. The alias need to be unique globally

**IAM�**�

Principal : its the physical person, application , device which wants to authenticate with AWS

User :  An IAM [user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html) is an entity that you create in AWS. The IAM user represents the person or service who uses the IAM user to interact with AWS. A primary use for IAM users is to give people the ability to sign in to the AWS Management Console for interactive tasks and to make programmatic requests to AWS services using the API or CLI. An IAM user is a single Principal 

Role :  An IAM [role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) is very similar to a user, in that it is an identity with permission policies that determine what the identity can and cannot do in AWS. However, a role does not have any credentials (password or access keys) associated with it. Instead of being uniquely associated with one person, a role is intended to be assumable by anyone who needs it, i.e multiple principals . An IAM user can assume a role to temporarily take on different permissions for a specific task. A role can be assigned to a [federated user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers.html) who signs in by using an external identity provider instead of IAM

Users are used when we know the actual user or application which will use the AWS services. Role is used when we do not know who individually will access the service but give it any user or service in general will use it 

Group : Group of users

- IAM is a global service
- IAM is free but has some limits
- IAM is globally resilient i.e can sustain failure in multiple regions
- IAM users can only control users within the account and cannot be configured for external users like federated users and users in other accounts
- An IAM user has only one username and password (password is optional for IAM users with key) but can have at most 2 key/secrets
- Keys can be created, deleted , inactivated etc.
- Access secret can be downloaded only once
- There can be max of **5000** users per account. If there are scenarios to have more than 5000 users than IAM roles or federation can be used. For exams if questions has more cases for 5000 users or internet users than IAM users is not the solution
- An IAM user can be added to max of **10** groups
- There are a limit of **300** groups per account but it can be increased
- no limits to max num of users in a group, a group will have max of 5k users since that’s the limit for account n not group
- There are no default group created when a account is created. Also groups do not support heirarchy 
- Groups do not have login credentials like users and also cannot be referenced from resource policy using arns
- Roles and users are identities and can be used from Resource based policies

**Amazon Resource Name** : every resource in AWS has a unique resource name called arn, the format of an arn starts with 

arn:partition:service:region:accountid . The rest is clear but partition is diffrent , the default partition is called aws, but say in china , partition will be aws-cn

resources will end with the following 

resource_type:resource:qualifier

For e,.g

arn:aws:iam::1234556977:user/tatha2  - this uniquely identifies IAM user tatha2 in account 1234556977. since IAM is a global service, so no region 

arn:aws:s3:::bucketname/file - files n buckets do not need 

arn:aws:ec2:us-east-1:123456789:instance/id - EC2 instance

arn:aws:s3::mybucket -  this identifies the bucket “mybocket”

arn:aws:s3::mybucket/* - this signifies all the files within the bucket but not the bucket itself

**AWS Policy:**  

AWS policy are permissions . They are in the form of a JSON document. A policy document is just a list of statement, which mentions which action is allow/deny on which resource

   

-  IdentityPolicy : permissions given to a given user/group/role. Attached to an identity and defines what that identity can do. Can be used to give access to only identities for that account 
-  ResourcePolicy : permissions given to a given Resource, attached to a resource and defines who can access the resources and what they can do, e.g S3 bucket policy. This policy can be used to give rights to resources to identities within or outside the account. IT can also be used to give access to anonymous users. A resource policy has the Principal in the policy document

Policies can also be identified as

- Inline Policy : is assigned individually to identities. Fo if one policy is assigned to 3 identities , 3 separate objects are created and if it has to be changed, 3 policies has to be changed, i.e high management overhead. Should only be used in case special cases, giving social right to only one person or deny only one person etc
- Managed Policy : common reusable policy , only one is used and assigned to identities. Low management overhead

 

Policies can be classified as 

-     AWS Policies : created by aws and non editable
-    Customer managed policies : 

Role Policies : 

Roles have special kind of policies 

- Trust Policy: This identifies which identities can assume the particular role . Assume because roles are ideally used for temporary. Once the identity (external users, systems) assumes the role, they are given temporary security credentials i.e token which is generated by STS service. Using the tokens, the identity can access resources based on the permission policy   
- Permission Policy : an typical resource policy 

A sample trust policy json

{

  "Version": "2012-10-17",

  "Statement": [

    {

      "Effect": "Allow",

      "Principal": {

        "AWS": "arn:aws:iam::417788663857:root” // allowing account no 

      },

      "Action": "sts:AssumeRole”, //sts token to assume role

      "Condition": {}

    }

  ]

}

The above is the Resource policy because it has the **Principal** field  

**Role Switching :** one can do role switching to access resources of one account in another account. To do that the destination account should create a role for the master account, giving the account id of the master., The permissions can based on what resources it wants to give to the master account, in most cases all Access. A master account user (not the root user of an account) can now switch role to access the resources of destination account 

A sample json for AWSS3ReadOnlyAccess

{

    "Version": "2012-10-17",

    "Statement": [

        {

            “Sid” : someid,

            "Effect": "Allow",

            "Action": [

                "s3:Get*",

                "s3:List*"

            ],

            "Resource": "*"

        }

    ]

}

**Effect** :  can be only Allow or Deny

**Action** : A list of all actions that can be done. It should be prefixed with the services name, like S3. Followed by wildcard like * , or Get, Save, List etc with wild card too

**Resource�**�: this also can be wildcard or a list of ARNs

**Rules for overlapping policies on the same Resource**

**1**. Explicit Deny : if there is an effect Deny, that takes highest precedence 

1. Explicit Allow: Effect is Allow 
2. Impicit Deny. If nothing applied, then its deny 

A sample Customer Manager Policy i created for bucket named Tatha

{

    "Version": "2012-10-17",

    "Statement": [

        {

            "Sid": "VisualEditor0",

            "Effect": "Allow",

            "Action": [

                "s3:GetLifecycleConfiguration",

                "s3:GetBucketTagging",

                "s3:GetBucketPolicyStatus",

**Omitted a lot of actions**

            ],

            "Resource": [

                "arn:aws:s3:::tatha/*",

                "arn:aws:s3:::tatha",

                "arn:aws:s3:*:417788663857:job/*",

                "arn:aws:s3:us-east-1:417788663857:accesspoint/local"

            ]

        },

        {

            "Sid": "VisualEditor1",

            "Effect": "Allow",

            "Action": [

                "s3:GetAccessPoint",

                "s3:GetAccountPublicAccessBlock",

                "s3:ListAllMyBuckets",

                "s3:ListAccessPoints",

                "s3:ListJobs",

                "s3:HeadBucket"

            ],

            "Resource": "*"

        }

    ]

}

With customer managed policy one can also add checks like source IPs and MFA accounts etc

**Service Control Policies :**

SCPs are like permission policies but it can be applied to only orgs, org units or accounts, basically account permission boundaries. 

- It identifies what each account can do. 
- It cannot be applied to root or master account of org. 
- It follows hierarchy so scp applied to org applies to all ou and accounts within the org 
- SCPs do not grant permissions , it only limits which services can be used, or which region can be used in that account, or was type of ec2 instances can be spun up. 
- Using SCP does not grant access to identities within the account. For e.g even if S3 is allowed in SCP , individual identity policy should be set to for a given user/role to access S3

- If SCP mentions that S3 cannot be used , then no identities within the account including root account user can use S3
- SCP polices comes in 2 types
    - Allow List : in this all services to be allowed need to be explicitly set to allow. This is more secure but more management overhead
    - Deny List : this is default, it means all services are allowed and we have to explicitly set deny for particular service. If S3 need to be disabled action : “S3:*” , with effect : Deny need to be set. It is less management overhead. When a deny SCP screamed by default a permission policy of Effect : Allow, Action : “*”, resource : “*” is set

**Resource Access Management** :

RAM is helpful when in an organization there are multiple accounts but all or few accounts need to share few resources like databases/VPC/S3/EC2 instances etc. The Source account needs to createn RAM and give the destination account and the resource. The destination account root user need to accept that RAM request

**SSO :**

This can be achieved through AWS SSO Service. It can be integrated with on premise Enterpise AD, or SAML enabled applications. All login related events are recorded in CloudTrail   

![9f208f25af13506dc044400f611d6d93.png](image/9f208f25af13506dc044400f611d6d93.png)
