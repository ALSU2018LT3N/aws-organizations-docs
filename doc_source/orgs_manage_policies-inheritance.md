# How policy inheritance works<a name="orgs_manage_policies-inheritance"></a>

You can attach policies to organization entities \(organization root, organizational unit, or account\):
+ When you attach a policy to the organization root, all OUs and accounts in the organization inherit that policy\. 
+ When you attach a policy to a specific OU, accounts that are directly under that OU or any child OU inherit the policy\.
+ When you attach a policy to a specific account, it affects only that account\. 

Because you can attach policies to multiple levels in the organization, accounts can inherit multiple policies\.

Exactly how policies affect the OUs and accounts that inherit them depends on the type of policy:
+ [Service control policies \(SCPs\)](#orgs_manage_policies-inheritance-scps)
+ [Tag policies](#orgs_manage_policies-inheritance-others)

## Inheritance for SCPs<a name="orgs_manage_policies-inheritance-scps"></a>

**Important**  
The information in this section does ***not*** apply to tag policies\. See the next section [Inheritance for tag policies](#orgs_manage_policies-inheritance-others)\.

Inheritance for service control policies behaves like a filter through which permissions flow to all parts of the tree below\. Imagine that the inverted tree structure of the organization is made of pipes that connect from the root to all of the OUs and end at the accounts\. All AWS permissions flow into the pipe at the root of the tree\. Those permissions must then flow past the SCPs attached to the root, OUs, and the account to get to the principal \(an IAM role or user\) making a request\. Each SCP can filter the permissions passing through to the levels below it\. If an action is blocked by a `Deny` statement then all OUs and accounts affected by that SCP are denied access to that action\. An SCP at a lower level can't add a permission back once it is blocked by an SCP at a higher level\. SCPs can ***only*** filter; they never add permissions back\.

SCPs do not support inheritance operators that alter how elements of the policy are inherited by child OUs and accounts\.

If you view the details for an account in the Organizations console, and choose Policies and then Service Control Policies in the right\-hand details pane, you can see a list of all policies applied to the account and where that policy comes from\. The same policy might apply to the account multiple times because the policy can be attached any or all of the parent containers of the account\. The effective policy that applies to the account is the intersection of allowed permissions of all applicable policies

For more information about how to use SCPs, see [Service control policies](orgs_manage_policies_scp.md)\.

## Inheritance for tag policies<a name="orgs_manage_policies-inheritance-others"></a>

Inheritance for tag policies behaves differently than SCPs\. The syntax for this policy type includes *inheritance operators*, which enable you to specify with fine granularity what elements from the parent policies are applied and what elements can be inherited by child OUs and accounts\.

The *effective policy* is the set of rules that are inherited from the organization root and OUs along with those directly attached to the account\. The effective policy specifies the rules that apply to the account\. You can view the effective tag policy for an account that includes the effect of all of the inheritance operators in the policies applied\. For more information, see [Viewing effective tag policies](orgs_manage_policies_tag-policies-effective.md)\.

This section explains how parent policies and child policies are processed into the effective policy for an account\. 

**Important**  
The information in this section does ***not*** apply to SCPs\. See the previous section [Inheritance for SCPs](#orgs_manage_policies-inheritance-scps)\.

### Terminology<a name="inheritance-terminology"></a>

The following table describes terms used in the following discussion about how policy inheritance works\.

Policy inheritance  
The interaction of policies at differing levels of an organization, moving from the top\-level root of the organization, down through the OU hierarchy to individual accounts\.  
You can attach policies to the organization root, organizational units \(OUs\), individual accounts, and to any combination of these organization entities\. Policy inheritance refers to policies that are attached to the organization root or to an OU\. All accounts that are members of the organization root or OU where a policy is attached *inherit* that policy\.  
For example, when policies are attached to the organization root, all accounts in the organization inherit that policy\. That's because all accounts in an organization are always under the organization root\. When you attach a policy to a specific OU, accounts that are directly under that OU or any child OU inherit that policy\. Because you can attach policies to multiple levels in the organization, accounts might inherit multiple policy documents for a single policy type\. 

Parent policies  
Policies that are attached higher in the organizational tree than policies that are attached to entities lower in the tree\.   
For example, if you attach policy A to the organization root, it's just a policy\. If you also attach policy B to an OU under that root, policy A is the parent policy of Policy B\. Policy B is the child policy of Policy A\. Policy A and policy B merge to create the effective tag policy for accounts in the OU\.

Child policies  
Policies that are attached at a lower level in the organization tree than the parent policy\. 

Effective policies  
The final, single policy document that specifies the rules that apply to an account\. The effective policy is the aggregation of any policies the account inherits, plus any policy that is directly attached to the account\. For example, tag policies enable you to view the effective tag policy that applies to any of your accounts\. For more information, see [Viewing effective tag policies](orgs_manage_policies_tag-policies-effective.md)\.

[Inheritance operators](#tag-policy-operators)  
Operators that control how inherited policies merge into a single effective policy\. These operators are considered an advanced feature\. Experienced tag policy authors can use them to limit what changes a child policy can make and how settings in policies merge\. 

### Inheritance operators<a name="tag-policy-operators"></a>

Inheritance operators control how inherited policies and account policies merge into the account's effective policy\. These operators include value\-setting operators and child control operators\. 

When you use the visual editor in the AWS Organizations console, you can use only the `@@assign` operator\. Other operators are considered an advanced feature\. To use the other operators, you must manually author the JSON policy\. Experienced policy authors can use them to control what values are applied to the effective policy and limit what changes child policies can make\. 

#### Value\-setting operators<a name="value-setting-operators"></a>

You can use the following value\-setting operators to control how your policy interacts with its parent policies\.
+ `@@assign` – **Overwrites** any inherited policy settings with the specified settings\. If the specified setting isn't inherited, this operator adds it to the effective policy\.
  + For tag policies, this removes any inherited tag keys and values and replaces them with the specified tag keys and values
+ `@@append` – **Adds** the specified settings \(without removing any\) to the inherited ones\. If the specified setting isn't inherited, this operator adds it to the effective policy\. 
  + For tag policies, this adds the specified tag keys and values to any inherited ones\.
+ `@@remove` – **Removes** the specified inherited settings from the effective policy, if they exist\.
  + For tag policies, this removes the specified tag keys and values from the effective policy, if they exist\. If this operator results in a tag policy key without any values, then accounts with that effective policy can use any value for that tag\.

#### Child control operators<a name="child-control-operators"></a>

You can use the `@@operators_allowed_for_child_policies` operator to control which value\-setting operators child policies can use\. Using child control operators is optional\. You can allow all operators, some specific operators, or no operators\. By default, all operators \(`@@all`\) are allowed\.
+ `"@@operators_allowed_for_child_policies"`:`["@@all"]` – Child OUs and accounts can use any operator in policies\. By default, all operators are allowed in child policies\.
+ `"@@operators_allowed_for_child_policies"`:`["@@assign", "@@append", "@@remove"]` – Child OUs and accounts can use only the specified operators in policies\. You can specify one or more value\-setting operators in this child control operator\.
+ `"@@operators_allowed_for_child_policies"`:`["@@none"]` – Child OUs and accounts can't use operators in policies\. You can use this operator to effectively lock down values that are defined in a parent policy so that child policies can't add, append, or remove values\.

**Note**  
If an inherited child control operator limits the use of an operator, you can't reverse that rule in a child policy\. If you include child control operators in a parent policy, they limit the value\-setting operators in all child policies\.

### Policy inheritance examples<a name="inheritance-examples"></a>

These examples show how policy inheritance works by showing how parent and child tag policies are merged into an effective tag policy for an account\.

The examples assume that you have the following organization structure\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/organizations/latest/userguide/images/org-structure-inheritance.png)

#### Example 1: Allow child policies to overwrite *only* tag values<a name="example-assign-operator"></a>

The following tag policy defines the `CostCenter` tag key and two acceptable values\. If you attach it to the organization root, the tag policy is in effect for all accounts in the organization\. 

**Policy A – Organization root tag policy**

```
{
    "tags": {
        "costcenter": {
            "tag_key": {
                "@@assign": "CostCenter"
            },
            "tag_value": {
                "@@assign": [
                    "Development",
                    "Support"
                ]
            }
        }
    }
}
```

Assume that you want users in OU1 to use different tag value for a key, and you want to enforce it for specific resource types\. Because policy A doesn't specify which child control operators are allowed, all operators are allowed\. You can use the `@@assign` operator and create a tag policy like the following to attach to OU1\. 

**Policy B – OU1 tag policy**

```
{
    "tags": {
        "costcenter": {
            "tag_key": {
                "@@assign": "CostCenter"
            },
            "tag_value": {
                "@@assign": [
                    "Sandbox"
                ]
            },
            "enforced_for": {
                "@@assign": [
                    "redshift:*",
                    "dynamodb:table"
                ]
            }
        }
    }
}
```

Specifying the `@@assign` operator for the tag does the following when policy A and policy B merge to form the *effective tag policy* for an account:
+ Policy B overwrites the two tag values that were specified in the parent policy, policy A\. The result is that `Sandbox` is the only compliant value for the `CostCenter` tag key\.
+ The addition of `enforced_for` specifies that the `CostCenter` tag *must* be the specified tag value on all Amazon Redshift resources and Amazon DynamoDB tables\.

As shown in the diagram, OU1 includes two accounts: 111111111111 and 222222222222\. 

**Resultant effective tag policy for accounts 111111111111 and 222222222222**

**Note**  
You can't directly use the contents of a displayed effective policy as the contents of a new policy\. The syntax doesn't include the operators needed to control merging with other child and parent policies\. The display of an effective policy is intended only for understanding the results of the merger\. 

```
{
    "tags": {
        "costcenter": {
            "tag_key": "CostCenter",
            "tag_value": [
                "Sandbox"
            ],
            "enforced_for": [
                "redshift:*",
                "dynamodb:table"
            ]
        }
    }
}
```

#### Example 2: Append new values to inherited tags<a name="example-append-operator"></a>

There may be cases where you want all accounts in your organization to specify a tag key with a short list of acceptable values\. For accounts in one OU, you may want to allow an additional value that only those accounts can specify when creating resources\. This example specifies how to do that by using the `@@append` operator\. The `@@append` operator is an advanced feature\. 

Like example 1, this example starts with policy A for the organization root tag policy\. 

**Policy A – Organization root tag policy**

```
{
    "tags": {
        "costcenter": {
            "tag_key": {
                "@@assign": "CostCenter"
            },
            "tag_value": {
                "@@assign": [
                    "Development",
                    "Support"
                ]
            }
        }
    }
}
```

For this example, attach policy C to OU2\. The difference in this example is that using the `@@append` operator in policy C *adds to*, rather than overwrites, the list of acceptable values and `enforced_for` rule\.

**Policy C – OU2 tag policy for appending values**

```
{
    "tags": {
        "costcenter": {
            "tag_key": {
                "@@assign": "CostCenter"
            },
            "tag_value": {
                "@@append": [
                    "Marketing"
                ]
            },
            "enforced_for": {
                "@@append": [
                    "redshift:*",
                    "dynamodb:table"
                ]
            }
        }
    }
}
```

Attaching policy C to OU2 has the following effects when policy A and policy C merge to form the effective tag policy for an account:
+ Because policy C includes the `@@append` operator, it allows for *adding to*, not overwriting, the list of acceptable tag values that are specified in Policy A\.
+ As in policy B, the addition of `enforced_for` specifies that the `CostCenter` tag must be used the specified tag value on all Amazon Redshift resources and Amazon DynamoDB tables\. Overwriting \(`@@assign`\) and adding \(`@@append`\) have the same effect if the parent policy doesn't include a child control operator that restricts what a child policy can specify\.

As shown in the diagram, OU2 includes one account: 999999999999\. Policy A and policy C merge to create the effective tag policy for account 999999999999\.

**Effective tag policy for account 999999999999**

**Note**  
You can't directly use the contents of a displayed effective policy as the contents of a new policy\. The syntax doesn't include the operators needed to control merging with other child and parent policies\. The display of an effective policy is intended only for understanding the results of the merger\. 

```
{
    "tags": {
        "costcenter": {
            "tag_key": "CostCenter",
            "tag_value": [
                "Development",
                "Support",
                "Marketing"
            ],
            "enforced_for": [
                "redshift:*",
                "dynamodb:table"
            ]
        }
    }
}
```

#### Example 3: Remove values from inherited tags<a name="example-remove-operator"></a>

There may be cases where the tag policy that is attached to the organization defines more tag values than you want an account to use\. This example explains how to do that using the `@@remove` operator\. The `@@remove` is an advanced feature\.

Like the other examples, this example starts with policy A for the organization root tag policy\.

**Policy A – Organization root tag policy**

```
{
    "tags": {
        "costcenter": {
            "tag_key": {
                "@@assign": "CostCenter"
            },
            "tag_value": {
                "@@assign": [
                    "Development",
                    "Support"
                ]
            }
        }
    }
}
```

For this example, attach policy D to account 999999999999\. 

**Policy D – Account 999999999999 tag policy for removing values**

```
{
    "tags": {
        "costcenter": {
            "tag_key": {
                "@@assign": "CostCenter"
            },
            "tag_value": {
                "@@remove": [
                    "Development",
                    "Marketing"
                ],
                "enforced_for": {
                    "@@remove": [
                        "redshift:*",
                        "dynamodb:table"
                    ]
                }
            }
        }
    }
}
```

Attaching policy D to account 999999999999 to has the following effects when policy A, policy C, and policy D merge to form the effective tag policy:
+ Assuming you performed all of the previous examples, policies B, C, and C are child policies of A\. Policy B is only attached to OU1, so it has no effect on account 9999999999999\.
+ For account 9999999999999, the only acceptable value for the `CostCenter` tag key is `Support`\.
+ Compliance is not enforced for the `CostCenter` tag key\.

**New effective tag policy for account 999999999999**

**Note**  
You can't directly use the contents of a displayed effective policy as the contents of a new policy\. The syntax doesn't include the operators needed to control merging with other child and parent policies\. The display of an effective policy is intended only for understanding the results of the merger\. 

```
{
    "tags": {
        "costcenter": {
            "tag_key": "CostCenter",
            "tag_value": [
                "Support"
            ]
        }
    }
}
```

If you later add more accounts to OU2, their effective tag policies would be different than for account 999999999999\. That's because the more restrictive policy D is only attached at the account level, and not to the OU\. 

#### Example 4: Restrict changes to child policies<a name="example-restrict-child"></a>

There may be cases where you want to restrict changes in child policies\. This example explains how to do that using child control operators\.

This example starts with a new organization root tag policy and assumes that tag policies aren't yet attached to organization entities\. 

**Policy E – Organization root tag policy for restricting changes in child policies **

```
{
    "tags": {
        "project": {
            "tag_key": {
                "@@operators_allowed_for_child_policies": ["@@none"],
                "@@assign": "Project"
            },
            "tag_value": {
                "@@operators_allowed_for_child_policies": ["@@append"],
                "@@assign": [
                    "Maintenance",
                    "Escalations"
                ]
            }
        }
    }
}
```

When you attach policy E to the organization root, prevents child policies from changing the `Project` tag key\. However, child policies can overwrite or append tag values\.

Assume you then attach the following policy to an OU\.

**Policy F – OU tag policy**

```
{
    "tags": {
        "project": {
            "tag_key": {
                "@@assign": "PROJECT"
            },
            "tag_value": {
                "@@append": [
                    "Escalations - research"
                ]
            }
        }
    }
}
```

Merging policy E and policy F have the following effects on the OU's accounts:
+ Policy F is a child policy to Policy E\.
+ Policy F attempts to change the case treatment, but it can't\. That's because policy E includes the `"@@operators_allowed_for_child_policies": ["@@none"]` operator for the tag key\.
+ However, policy F can append tag values for the key\. That's because policy E includes `"@@operators_allowed_for_child_policies": ["@@append"]` for the tag value\. 

**Effective policy for accounts in the OU**

**Note**  
You can't directly use the contents of a displayed effective policy as the contents of a new policy\. The syntax doesn't include the operators needed to control merging with other child and parent policies\. The display of an effective policy is intended only for understanding the results of the merger\. 

```
{
    "tags": {
        "project": {
            "tag_key": "project",
            "tag_value": [
                "Maintenance",
                "Escalations",
                "Escalations - research"
            ]
        }
    }
}
```

#### Example 5: Conflicts with child control operators<a name="multiple-same-node-operators"></a>

Child control operators can exist in tag policies that are attached at the same level in the organization hierarchy\. When that happens, the intersection of the allowed operators is used when the policies merge to form the effective policy for accounts\.

Assume policy G and policy H are attached to the organization root\. 

**Policy G – Organization root tag policy 1**

```
{
    "tags": {
        "project": {
            "tag_value": {
                "@@operators_allowed_for_child_policies": ["@@append"],
                "@@assign": [
                    "Maintenance"
                ]
            }
        }
    }
}
```

**Policy H – Organization root tag policy 2**

```
{
    "tags": {
        "project": {
            "tag_value": {
                "@@operators_allowed_for_child_policies": ["@@append", "@@remove"]
            }
        }
    }
}
```

In this example, one policy at the organization root defines that the values for the tag key can only be appended to\. The other policy attached to the organization root allows child policies to both append and remove values\. The intersection of these two permissions is used for child policies\. The result is that child policies can append values, but not remove values\. Therefore, the child policy can append a value to the list of tag values, but can't remove `Maintenance`\.

#### Example 6: Conflicts with appending values at same hierarchy level<a name="multiple-same-node-values"></a>

You can attach multiple tag policies to each organization entity\. When you do this, the tag policies that are attached to the same organization entity can include conflicting information\. These policies are evaluated based on the order in which they were attached to the organization entity\. To change which policy is evaluated first, you can detach a policy and then reattach it\.

Assume policy J is attached to the organization root first, and then policy K is attached to the organization root\. 

**Policy J – First tag policy attached to the organization root**

```
{
    "tags": {
        "project": {
            "tag_key": {
                "@@assign": "PROJECT"
            },
            "tag_value": {
                "@@append": ["Maintenance"]
            }
        }
    }
}
```

**Policy K – Second tag policy attached to the organization root**

```
{
    "tags": {
        "project": {
            "tag_key": {
                "@@assign": "project"
            }
        }
    }
}
```

In this example, the tag key `PROJECT` is used in the effective tag policy because the policy that defined it was attached to the organization root first\.

**Policy JK – Effective tag policy for account**

The effective policy for the account is as follows\.

**Note**  
You can't directly use the contents of a displayed effective policy as the contents of a new policy\. The syntax doesn't include the operators needed to control merging with other child and parent policies\. The display of an effective policy is intended only for understanding the results of the merger\. 

```
{
    "tags": {
        "project": {
            "tag_key": " PROJECT",
            "tag_value": [
                "Maintenance",
                "Escalations"
            ]
        }
    }
}
```