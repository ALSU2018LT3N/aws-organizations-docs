# About Service Control Policies<a name="orgs_manage_policies_about-scps"></a>

A service control policy \(SCP\) determines what services and actions can be delegated by administrators to the users and roles in the accounts that the SCP is applied to\. An SCP ***does not*** grant any permissions\. Instead, think of it as a filter\. If the SCP allows the actions for a service, then the administrator of the account can grant permissions for those actions to the users and roles in that account, and the users and roles can use perform the actions if the administrators grant those permissions\. If the SCP denies actions for a service, then the administrators in that account cannot effectively grant permissions for those actions, and the users and roles in the account cannot perform the actions even if granted by an administrator\. For example, you can specify that the administrators of account 111122223333 can grant permissions only to Amazon S3 and Amazon EC2, while the administrators of account 777788889999 can grant permissions only to DynamoDB\.

**Important**  
SCPs ***do not*** affect the master account no matter where the account is in the root/OU hierarchy\.
SCPs ***do*** affect the root user along with all IAM users and standard IAM roles in any affected account\.
SCPs ***do not*** affect any service\-linked role in an account\. These roles exist to support integration with other AWS services and can't be restricted by SCPs\.
SCPs are available only in organizations that enable all features\. SCPs are not available if your organization has enabled only the consolidated billing features\.

The following illustration shows how SCPs work\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/organizations/latest/userguide/images/How_SCP_Permissions_Work.jpg)

In this illustration, the root has an SCP attached that allows permissions A, B, and C\. An OU in that root has an SCP that allows C, D, and E\. Because the root's OU does not allow D or E, then nothing in the root or any of its children can use them, including the parent OU\. Even though the parent OU explicitly allows them, they end up blocked because they are blocked by the root\. Also, because the OU's SCP does not allow A or B, those permissions are blocked for the parent OU and any of its children\. However, other OUs under the root that are peers to the parent OU could allow A and B\.

Users and roles must still be granted permissions using IAM permission policies attached to them or to groups\. The permissions granted by such policies are filtered by the SCPs, and any actions that are not permitted by the applicable SCPs cannot be performed by the user\. Actions allowed by the SCPs can be used if they are granted to the user or role by one or more IAM permission policies\.

When you attach SCPs to the root, OUs, or directly to accounts, all policies that affect a given account are evaluated together using the same rules that govern IAM permission policies:

+ Any action that has an explicit `Deny` in an SCP cannot be delegated to users or roles in the affected accounts\. An explicit `Deny` statement overrides any `Allow` that might be granted by other SCPs\.

+ Any action that has an explicit `Allow` in an SCP \(such as the default "\*" SCP or by any other SCP that calls out a specific service or action\) can be delegated to users and roles in the affected accounts\.

+ Any action that is not explicitly allowed by an SCP is implicitly denied and cannot be delegated to users or roles in the affected accounts\.

By default, an SCP named `FullAWSAccess` is attached to every root, OU, and account\. This default SCP allows all actions and all services\. So in a new organization, until you start creating or manipulating the SCPs, all of your existing IAM permissions continue to operate as they did\. As soon as you apply a new or modified SCP to a root or OU that contains an account, the permissions that your users have in that account become filtered by the SCP\. Permissions that used to work might now be denied if they are not allowed by the SCP at every level of the hierarchy down to the specified account\.

If you disable the SCP policy type in a root, all SCPs are automatically detached from all entities in that root\. If you re\-enable SCPs in that root, all the original attachments are lost, and all entities are reset to being attached to only the default `FullAWSAccess` SCP\.

For details about the syntax of SCPs, see [Service Control Policy Syntax](orgs_reference_scp-syntax.md) in the Reference section of this guide\.

## Strategies for Using SCPs<a name="SCP_strategies"></a>

You can configure the SCPs in your organization to work as either of the following:

+ A blacklist \- actions are allowed by default, and you specify what services and actions are prohibited\.

+ A whitelist \- actions are prohibited by default, and you specify what services and actions are allowed\.

### Using SCPs as a Blacklist<a name="orgs_policies_blacklist"></a>

The default configuration of AWS Organizations supports using SCPs as blacklists\. Account administrators can delegate all services and actions until you create and attach a policy that denies \(blacklists\) a specific service or set of actions\.

To support this, AWS Organizations attaches an AWS managed SCP named [https://console.aws.amazon.com/organizations/?#/policies/p-FullAWSAccess](https://console.aws.amazon.com/organizations/?#/policies/p-FullAWSAccess) to every root and OU when it is created\. This policy allows all services and actions\. Because the policy is an AWS managed SCP, it cannot be modified or deleted\. It is always available for you to attach or detach from the entities in your organization as needed\. The policy looks like this:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        }
    ]
}
```

This enables account administrators to delegate permissions for any service or action until you create and attach an SCP that explicitly prohibits those actions that you don't want users and roles in certain accounts to perform\.

Such a policy might look like the following example, which prevents users in the affected accounts from performing any actions for the DynamoDB service\. The organization administrator can detach the `FullAWSAccess` policy and attach this one instead\. Note that this SCP still allows all other services and their actions:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowsAllActions",
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        },
        {
            "Sid": "Deny 
            "Effect": "Deny",
            "Action": "dynamodb:*",
            "Resource": "*"
        }
    ]
}
```

The users in the affected accounts cannot perform DynamoDB actions because the explicit `Deny` element in the second statement overrides the explicit `Allow` in the first\. You could also configure this by leaving the `FullAWSAccess` policy in place, and then attaching a second policy that has only the `Deny` statement in it, as shown here:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": "dynamodb:*",
            "Resource": "*"
        }
    ]
}
```

The combination of the `FullAWSAccess` policy and the `Deny` statement in the preceding DynamoDB policy that is applied to a root or OU has the exact same effect as the single policy that contains both statements\. All policies that apply at a specified level are combined together, and each statement, no matter which policy originated it, gets evaluated according to the rules discussed earlier \(that is, an ***explicit*** `Deny` overrides an ***explicit ***`Allow`, which overrides the default ***implicit ***`Deny`\)\.

### Using SCPs as a Whitelist<a name="orgs_policies_whitelist"></a>

To use SCPs as a whitelist, you must replace the AWS managed `FullAWSAccess` SCP with an SCP that explicitly permits only those services and actions that you want to allow\. By removing the default `FullAWSAccess` SCP, all actions for all services are now implicitly denied\. Your custom SCP then overrides the implicit `Deny` with an explicit `Allow` for only those actions that you want to permit\. Note that for a permission to be enabled for a specified account, every SCP from the root through each OU in the direct path to the account, and even attached to the account itself, must allow that permission\.

Such a whitelist policy might look like the following example, which enables account users to perform operations for Amazon EC2 and Amazon CloudWatch, but no other service\. All SCPs in parent OUs and the root also must explicitly allow these permissions:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:*",
                "cloudwatch:*"
            ],
            "Resource": "*"
        }
    ]
}
```