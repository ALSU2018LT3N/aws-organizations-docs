# Remove the Master Account and Delete the Organization<a name="orgs_manage_org_delete"></a>

When you no longer need your organization, you can delete it\. This removes the master account from the organization and deletes the organization itself\. The former master account becomes a standalone AWS account\. You then have three options: You can continue to use it as a standalone account, you can use it to create a different organization, or you can accept an invitation from another organization to add the account to that organization as a member account\. 

**Important**  
If you delete an organization, you will not be able to recover it\. If you created any policies inside of the organization, they will also be deleted\.
You can delete an organization only after you remove all member accounts from the organization\. If you created some of your member accounts using AWS Organizations, you might be blocked from removing those accounts\. You can remove a member account only if it has all the information that is required to operate as a standalone AWS account\. For more information about how to provide that information and remove the account, see [Leaving an Organization as a Member Account](orgs_manage_accounts_remove.md#orgs_manage_accounts_leave-as-member)\.

When you remove the master account from an organization by deleting the organization, the account is affected in the following ways:
+ The account is responsible for paying only its own charges and is no longer responsible for the charges incurred by any other account\.
+ Integration with other services might be disabled\. For example, AWS Single Sign\-On requires an organization to operate, so if you remove an account from an organization that supports AWS SSO, the users in that account can no longer use that service\.

The master account of an organization is never affected by service control policies \(SCPs\), so there is no change in permissions after SCPs are no longer available\.

**To remove the master account from an organization and delete the organization \(console\)**
**Minimum permissions**  
To delete an organization, you must sign in as an IAM user or role in the master account, and you must have the following permissions:  
`organizations:DeleteOrganization`
`organizations:DescribeOrganization` \(console only\)

1. Sign in to the Organizations console at [https://console\.aws\.amazon\.com/organizations/](https://console.aws.amazon.com/organizations/)\. You must sign in as an IAM user, assume an IAM role, or sign in as the root user \([not recommended](http://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#lock-away-credentials)\) in the organization's master account\.

1. Before you can delete the organization, you must first remove all accounts from the organization\. For more information, see [Removing a Member Account from Your Organization](orgs_manage_accounts_remove.md)\.

1. On the **Settings** tab, choose **Delete organization**\.

1. In the **Delete organization** confirmation dialog box, choose **Delete organization**\.

1. \(Optional\) If you also want to close this account, you can follow the steps at [Closing an AWS Account](orgs_manage_accounts_close.md)\.

**To delete an organization \(AWS CLI, AWS API\)**  
You can use one of the following commands to delete an organization: 
+ AWS CLI: [aws organizations delete\-organization](http://docs.aws.amazon.com/cli/latest/reference/organizations/delete-organization.html)
+ AWS API: [DeleteOrganization](http://docs.aws.amazon.com/organizations/latest/APIReference/API_DeleteOrganization.html)