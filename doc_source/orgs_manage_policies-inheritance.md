# How policy inheritance works<a name="orgs_manage_policies-inheritance"></a>

You can attach policies to organization entities \(organization root, organizational unit \(OU\), or account\):
+ When you attach a policy to the organization root, all OUs and accounts in the organization inherit that policy\. 
+ When you attach a policy to a specific OU, accounts that are directly under that OU or any child OU inherit the policy\.
+ When you attach a policy to a specific account, it affects only that account\. 

Because you can attach policies to multiple levels in the organization, accounts can inherit multiple policies\.

Exactly how policies affect the OUs and accounts that inherit them depends on the type of policy:
+ [Authorization policy types](orgs_manage_policies_inheritance_auth.md) – Service control policies \(SCPs\)
+ [Management policy types](orgs_manage_policies_inheritance_mgmt.md) – Tag policies