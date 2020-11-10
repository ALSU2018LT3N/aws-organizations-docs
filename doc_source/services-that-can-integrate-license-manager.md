# AWS License Manager and AWS Organizations<a name="services-that-can-integrate-license-manager"></a>

AWS License Manager streamlines the process of bringing software vendor licenses to the cloud\. As you build out cloud infrastructure on AWS, you can save costs by using bring\-your\-own\-license \(BYOL\) opportunities—that is, by repurposing your existing license inventory for use with cloud resources\. With rule\-based controls on the consumption of licenses, administrators can set hard or soft limits on new and existing cloud deployments, stopping noncompliant server usage before it happens\.

By linking License Manager with AWS Organizations, you can enable cross\-account discovery of computing resources throughout your organization\.

For more information about License Manager, see the [License Manager User Guide](https://docs.aws.amazon.com/license-manger/latest/userguide/)\.

Use the following information to help you to help you integrate AWS License Manager with AWS Organizations\.

**Topics**
+ [Service\-linked roles created when you enable integration](#integrate-enable-slr-license-manager)
+ [Service principals used by the service\-linked roles](#integrate-enable-svcprin-license-manager)
+ [Enabling trusted access with License Manager](#integrate-enable-ta-license-manager)

## Service\-linked roles created when you enable integration<a name="integrate-enable-slr-license-manager"></a>

The following [service\-linked roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) are automatically created in your organization's accounts when you enable trusted access\. These roles allow License Manager to perform supported operations within the accounts in your organization\.

You can delete or modify these roles only if you disable trusted access between License Manager and Organizations or if the account is removed from the organization\.
+ `AWSLicenseManagerMasterAccountRole`
+ `AWSLicenseManagerMemberAccountRole`
+ `AWSServiceRoleForAWSLicenseManagerRole`

For more information, see [Using the License Manager–management account role](https://docs.aws.amazon.com/license-manger/latest/userguide/master-role.html) and [Using the License Manager–member account role](https://docs.aws.amazon.com/license-manger/latest/userguide/member-role.html)\.

## Service principals used by the service\-linked roles<a name="integrate-enable-svcprin-license-manager"></a>

The service\-linked roles in the previous section can be assumed only by the service principals authorized by the trust relationships defined for the role\. The service\-linked roles used by License Manager grant access to the following service principals:
+ `license-manager.amazonaws.com`
+ `license-manager.member-account.amazonaws.com`

## Enabling trusted access with License Manager<a name="integrate-enable-ta-license-manager"></a>

For information about the permissions needed to enable trusted access, see [Permissions required to enable trusted access](orgs_integrate_services.md#orgs_trusted_access_perms)\.

**To enable trusted access with License Manager**  
You must sign in to the License Manager console using your AWS Organizations management account and associate it with your License Manager account\. Then you can configure your License Manager settings\. For information, see [Configuring AWS License Manager Guide Settings](https://docs.aws.amazon.com/license-manger/latest/userguide/settings.html)\.