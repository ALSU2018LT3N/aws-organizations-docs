# Authentication and Access Control for AWS Organizations<a name="orgs_permissions"></a>

Access to AWS Organizations requires credentials\. Those credentials must have permissions to access AWS resources, such as an Amazon S3 bucket, an Amazon Elastic Compute Cloud \(Amazon EC2\) instance, or an Organizations organizational unit \(OU\)\. The following sections provide details on how you can use AWS Identity and Access Management \(IAM\) to help secure access to your organization and control who can administer it\.

To determine who can administer which parts of your organization, AWS Organizations uses the same IAM\-based permissions model as other AWS services\. As an admin in the master account of an organization, you can grant IAM\-based permissions to perform Organizations tasks by attaching policies to users, groups, and roles in the master account\. Those policies specify the actions that those principals can perform\. You attach an IAM permissions policy to a group that the user is a member of, or directly to a user or role\. [As a best practice, we recommend that you attach policies to groups instead of users](http://alpha-docs-aws.amazon.com/IAM/latest/UserGuide/best-practices.html#use-groups-for-permissions)\. You also have the option to grant full admin permissions to others\.

For most admin operations for Organizations, you need to attach permissions to users or groups in the master account\. If a user in a member account needs to perform admin operations for your organization, then you need to grant the Organizations permissions to an *IAM role* in the master account and enable the user in the member account to assume the role\. For general information about IAM permissions policies, see [Overview of IAM Policies](http://alpha-docs-aws.amazon.com/IAM/latest/UserGuide/access_policies.html) in the *IAM User Guide*\.


+ [Authentication](#orgs_permissions_authentication)
+ [Access Control](#orgs-access-control)
+ [Overview of Managing Access Permissions to Your AWS Organization](orgs_permissions_overview.md)
+ [Using Identity\-Based Policies \(IAM Policies\) for AWS Organizations](orgs_permissions_iam-policies.md)

## Authentication<a name="orgs_permissions_authentication"></a>

You can access AWS as any of the following types of identities:

+ **AWS account root user** \- When you sign up for AWS, you provide an email address and password that is associated with your AWS account\. These are your *root credentials*, and they provide complete access to all of your AWS resources\.
**Important**  
For security reasons, we recommend that you use the root credentials only to create an *administrator user*, which is an *IAM user* with full permissions to your AWS account\. Then, you can use this administrator user to create other IAM users and roles with limited permissions\. For more information, see [IAM Best Practices](http://alpha-docs-aws.amazon.com/IAM/latest/UserGuide/best-practices.html#create-iam-users) and [Creating an Admin User and Group](http://alpha-docs-aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html) in the *IAM User Guide*\.

+ **IAM user** \- An [IAM user](http://alpha-docs-aws.amazon.com/IAM/latest/UserGuide/id_users.html) is simply an identity within your AWS account that has specific custom permissions \(for example, permissions to create a file system in Amazon EFS\)\. You can use an IAM user name and password to sign in to secure AWS webpages like the [AWS Management Console](https://console.aws.amazon.com/), [AWS Discussion Forums](https://forums.aws.amazon.com/), or the [AWS Support Center](https://console.aws.amazon.com/support/home#/)\.

  In addition to a user name and password, you can also generate [access keys](http://alpha-docs-aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html) for each user\. You can use these keys when you access AWS services programmatically, either through [one of the several SDKs](https://aws.amazon.com/tools/) or by using the [AWS Command Line Interface \(CLI\)](https://aws.amazon.com/cli/)\. The SDK and CLI tools use the access keys to cryptographically sign your request\. If you don’t use the AWS tools, you must sign the request yourself\. AWS Organizations supports *Signature Version 4*, a protocol for authenticating inbound API requests\. For more information about authenticating requests, see [Signature Version 4 Signing Process](http://alpha-docs-aws.amazon.com/general/latest/gr/signature-version-4.html) in the *AWS General Reference*\.

+ **IAM role** \- An IAM role is another IAM identity you can create in your account that has specific permissions\. It is similar to an IAM user, but it is not associated with a specific person\. An IAM role enables you to obtain temporary access keys that can be used to access AWS services and resources\. IAM roles with temporary credentials are useful in the following situations:

  + **Federated user access** – Instead of creating an IAM user, you can use preexisting user identities from AWS Directory Service, your enterprise user directory, or a web identity provider\. These are known as *federated users*\. AWS assigns a role to a federated user when access is requested through an [identity provider](http://alpha-docs-aws.amazon.com/IAM/latest/UserGuide/id_roles_providers.html)\. For more information about federated users, see [Federated Users and Roles](http://alpha-docs-aws.amazon.com/IAM/latest/UserGuide/introduction_access-management.html#intro-access-roles) in the *IAM User Guide*\.

  + **Cross\-account access** – You can use an IAM role in your account to grant another AWS account permissions to access your account’s resources\. For an example, see [Tutorial: Delegate Access Across AWS Accounts Using IAM Roles](http://alpha-docs-aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html) in the *IAM User Guide*\.

  + **AWS service access** – You can use an IAM role in your account to grant an AWS service permissions to access your account’s resources\. For example, you can create a role that allows Amazon Redshift to access an Amazon S3 bucket on your behalf and then load data stored in the bucket into an Amazon Redshift cluster\. For more information, see [Creating a Role to Delegate Permissions to an AWS Service](http://alpha-docs-aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) in the *IAM User Guide*\.

  + **Applications running on Amazon EC2** – Instead of storing access keys within the EC2 instance for use by applications running on the instance and making AWS API requests, you can use an IAM role to manage temporary credentials for these applications\. To assign an AWS role to an EC2 instance and make it available to all of its applications, you can create an instance profile that is attached to the instance\. An instance profile contains the role and enables programs running on the EC2 instance to get temporary credentials\. For more information, see [Using Roles for Applications on Amazon EC2](http://alpha-docs-aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) in the *IAM User Guide*\.

## Access Control<a name="orgs-access-control"></a>

You can have valid credentials to authenticate your requests, but unless you have permissions you cannot administer or access AWS Organizations resources\. For example, you must have permissions to create an organizational unit \(OU\) or to attach a service control policy \(SCP\) to an account\.

The following sections describe how to manage permissions for AWS Organizations\. We recommend that you read the overview first\.

+ [Overview of Managing Access Permissions to Your AWS Organization](orgs_permissions_overview.md)

+ [Using Identity\-Based Policies \(IAM Policies\) for AWS Organizations](orgs_permissions_iam-policies.md)