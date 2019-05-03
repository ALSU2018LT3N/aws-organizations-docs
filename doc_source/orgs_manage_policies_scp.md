# Service Control Policies<a name="orgs_manage_policies_scp"></a>

Service control policies \(SCPs\) are one type of policy that you can use to manage your organization\. SCPs offer central control over the maximum available permissions for all accounts in your organization, allowing you to ensure your accounts stay within your organization’s access control guidelines\. SCPs are available only in an organization that has [all features enabled](orgs_manage_org_support-all-features.md)\. SCPs aren't available if your organization has enabled only the consolidated billing features\. For instructions on enabling SCPs, see [Enabling and Disabling a Policy Type on a Root](orgs_manage_policies.md#enable_policies_on_root)\.

SCPs are necessary but not sufficient for granting access in the accounts in your organization\. Attaching an SCP to the organization root or an organizational unit \(OU\) defines a guardrail for what actions accounts within the organization root or OU can do\. You still need to attach [IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html) to users and roles in your organization's accounts to actually grant permissions to them\. With an SCP attached to those accounts, identity\-based and resource\-based policies grant permissions to entities only if those policies and the SCP allow the action\. If both a permissions boundary \(an advanced IAM feature\) and an SCP are present, then the boundary, the SCP, and the identity\-based policy must all allow the action\. For more information, see [Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html) in the IAM User Guide\. 

**Topics**
+ [Testing Effects of SCPs](#scp-warning-testing-effect)
+ [SCP Size Limit](#scp-size-limit)
+ [Effects on Permissions](#scp-effects-on-permissions)
+ [Tasks and Entities Not Restricted by SCPs](#not-restricted-by-scp)
+ [Amazon EC2 Actions for the Root User](#ec2-actions-root)
+ [How SCPs Work](orgs_manage_policies_about-scps.md)
+ [Strategies for Using SCPs](SCP_strategies.md)
+ [Creating and Updating SCPs](create-policy.md)
+ [Example Service Control Policies](orgs_manage_policies_example-scps.md)
+ [SCP Syntax](orgs_reference_scp-syntax.md)

## Testing Effects of SCPs<a name="scp-warning-testing-effect"></a>

AWS strongly recommends that you don't attach SCPs to the root of your organization without thoroughly testing the impact that the policy has on accounts\. Instead, create an OU that you can move your accounts into one at a time, or at least in small numbers, to ensure that you don't inadvertently lock users out of key services\. One way to determine whether a service is used by an account is to examine the [service last accessed data in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_access-advisor.html)\. Another way is to [use AWS CloudTrail to log service usage at the API level](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/how-cloudtrail-works.html)\.

## SCP Size Limit<a name="scp-size-limit"></a>

All characters in your SCP count against its [size limit](orgs_reference_limits.md#min-max-values)\. The examples in this guide show the SCPs formatted with extra white space to improve their readability\. However, to save space if your policy size approaches the limit, you can delete any white space, such as space characters and line breaks that are outside quotation marks\.

**Tip**  
Use the visual editor to build your SCP\. It automatically removes extra white space\.

## Effects on Permissions<a name="scp-effects-on-permissions"></a>

SCPs are similar to IAM permission policies and use almost the same syntax\. However, an SCP never grants permissions\. Instead, SCPs are JSON policies that specify the maximum permissions for an organization or organizational unit \(OU\)\. Note the following:
+ An SCP limits permissions for entities in member accounts, including each AWS account root user\. Any account has only those permissions permitted by ***every*** parent above it\. If a permission is blocked at any level above the account, either implicitly \(by not being included in an `Allow` policy statement\) or explicitly \(by being included in a `Deny` policy statement\), a user or role in the affected account can't use that permission, even if the account administrator attaches the `AdministratorAccess` IAM policy with \*/\* permissions to the user\.
+ Users and roles must still be granted permissions with appropriate IAM permission policies\. A user without any IAM permission policies has no access at all, even if the applicable SCPs allow all services and all actions\.
+ If a user or role has an IAM permission policy that grants access to an action that is also allowed by the applicable SCPs, the user or role can perform that action\.
+ If a user or role has an IAM permission policy that grants access to an action that is either not allowed or explicitly denied by the applicable SCPs, the user or role can't perform that action\.
+ SCPs affect all users and roles in attached accounts, ***including the root user***\. The only exceptions are those described in the following list of tasks that aren't affected and can't be restricted by using SCPs\.
+ SCPs ***do not ***affect any service\-linked role\. Service\-linked roles enable other AWS services to integrate with AWS Organizations and can't be restricted by SCPs\.
+ SCPs ***affect only principals*** that are managed by accounts that are part of the organization\. They don't affect users or roles from accounts outside the organization\. For example, consider an Amazon S3 bucket that's owned by account A in an organization\. The bucket policy grants access to users from accounts outside the organization\. Account A has an SCP attached\. That SCP doesn't apply to those outside users\. It applies only to users that are managed by account A in the organization\. 
+ When you disable the SCP policy type in a root, all SCPs are automatically detached from all entities in that root\. If you reenable SCPs in a root, that root reverts to only the default `FullAWSAccess` policy automatically attached to all entities in the root\. Any attachments of SCPs to entities from before SCPs were disabled are lost and aren't automatically recoverable, although you can manually reattach them\.

## Tasks and Entities Not Restricted by SCPs<a name="not-restricted-by-scp"></a>

The following tasks and entities are not restricted by SCPs:
+ Any action performed using permissions that are attached to a service\-linked role\.
+ Managing root credentials\. No matter what SCPs are attached, the root user in an account can always do the following:
  + Changing the root user's password
  + Creating, updating, or deleting root access keys
  + Enabling or disabling multi\-factor authentication on the root user
  + Creating, updating, or deleting x\.509 keys for the root user
+ Registering for the Enterprise support plan as the root user
+ Changing the AWS support level as the root user
+ Managing Amazon CloudFront keys
+ Trusted signer functionality for CloudFront private content
+ Modifying AWS account email allowance/reverse DNS
+ Performing tasks on some AWS\-related services:
  + Alexa Top Sites
  + Alexa Web Information Service
  + Amazon Mechanical Turk
  + Amazon Product Marketing API

## Amazon EC2 Actions for the Root User<a name="ec2-actions-root"></a>

Currently, when using [root user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html) credentials to make Amazon EC2 requests, the resource and condition policy elements don't function as expected in the following ways:
+ **Resource ARNs** – If you specify an AWS resource ARN in an SCP's resource element, it won't match the resource the root user performs the Amazon EC2 action on during policy evaluation\. This means that the specified restrictions for this action don't apply to the root user\. Deny statements that specify all resources \(`"Resource": "*"`\) for Amazon EC2 actions are correctly evaluated for root users\. 
+ **Amazon EC2 condition keys** – All condition keys that start with `"ec2"` aren't evaluated when using root credentials\. Because policy conditions aren't correctly evaluated for root users, users with root credentials might have unintended access to Amazon EC2 actions\. 

This issue doesn't affect IAM users and roles or any AWS service except Amazon EC2\. Only the root user is subject to this issue\. If you don’t want your root user to have access to Amazon EC2 actions, attach an SCP like the [Restrict Access to Amazon EC2 for Root User](orgs_manage_policies_example-scps.md#example-ec2-root-user) example to your organization root\.