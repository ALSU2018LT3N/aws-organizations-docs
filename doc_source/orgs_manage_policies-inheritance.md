# How Policy Inheritance Works<a name="orgs_manage_policies-inheritance"></a>

You can attach tag policies to any organization entity \(organization root, organizational unit, or account\):
+ When you attach a tag policy to the organization root, all accounts in the organization inherit that policy\. 
+ When you attach a tag policy to a specific OU, accounts that are directly under that OU or any child OU inherit the policy\.
+ When you attach tag policies to an account, they affect only that account\. 

Because you can attach policies to multiple levels in the organization, accounts can inherit multiple tag policies\.

The *effective tag policy* is the set of tagging rules that are inherited from the organization root and OUs, plus the attached account tag policies\. The effective tag policy specifies the tagging rules that apply to the account\. To learn how to view the effective tag policy for an account, see [Viewing Effective Tag Policies](orgs_manage_policies_tag-policies-effective.md)\.

This page describes how parent policies and child policies are aggregated into the effective policy for an account\.

## Terminology<a name="inheritance-terminology"></a>

The following table describes common terms used in defining how policy inheritance works\.


| Term | Definition | 
| --- | --- | 
| Policy inheritance |  The interaction of tag policies at differing levels of an organization\. You can attach tag policies to the organization root, organizational units \(OUs\), individual accounts, and to a combination of these organization entities\. Policy inheritance refers to policies that are attached to the organization root or to an OU\. All accounts that are members of the organization root or OU where a tag policy is attached *inherit* that tag policy\. For example, when policies are attached to the organization root, all accounts in the organization inherit that policy\. That's because all accounts in an organization are always under the organization root\. When policies are attached to a specific OU, accounts that are directly under that OU or any child OU inherit the policy\. Because you can attach policies to multiple levels in the organization, accounts might inherit multiple policy documents for a single policy type\.   | 
| Parent policies | Policies that are attached higher in the organizational tree than policies that are attached to entities lower in the tree\.  For example, if you attach policy A to the organization root, it's just a policy\. If you also attach policy B to an OU, policy A is the parent policy of Policy B\. Policy B is the child policy of Policy A\. Policy A and policy B merge to create the effective tag policy for accounts in the OU\.  | 
| Child policies | Policies that are attached at a lower level in the organization tree than the parent policy\.  | 
| [Effective policy](orgs_manage_policies_tag-policies-effective.md) | A single policy document that specifies the tagging rules that apply to an account\. The effective policy is the aggregation of any tag policies the account inherits, plus any tag policy that is directly attached to the account\.  | 
| [Inheritance operators](#tag-policy-operators) | Operators that control how inherited policies merge into a single effective policy\. These operators are considered an advanced feature\. Experienced tag policy authors can use them to limit what changes a child policy can make and how settings in policies merge\.   | 

## Inheritance Operators<a name="tag-policy-operators"></a>

Inheritance operators control how inherited tag policies and account tag policies merge into the account's effective tag policy\. These operators include value\-setting operators and child control operators\. 

When you use the visual editor in the AWS Organizations console, you can use only the `@@assign` operator\. Other operators are considered an advanced feature\. To use the other operators, you must manually author the JSON policy\. Experienced tag policy authors can use them to control what tag values are applied to the effective tag policy and limit what changes child policies can make\. 

### Value\-Setting Operators<a name="value-setting-operators"></a>

You can use the value\-setting operators to control how your tag policy interacts with its parent policies\.
+ `@@assign` – Overwrites inherited tag keys and values with the specified tag keys and values\. If the specified tag isn't inherited, this operator adds the key and value to the effective policy\.
+ `@@append` – Adds the specified tag keys and values to the inherited tag keys and values\. If the specified tag isn't inherited, this operator adds the keys and values to the effective policy\. 
+ `@@remove` – Removes specific inherited tag keys and values from the effective policy, if they exist\. If this operator removes all values from a tag policy key, accounts that are directly under that OU or any child OU can use any value for that tag\.

### Child Control Operators<a name="child-control-operators"></a>

You can use the `@@operators_allowed_for_child_policies` operator to control which value\-setting operators child tag policies can use\. Using child control operators is optional\. You can allow all operators, some specific operators, or no operators\. By default, all operators \(`@@all`\) are allowed\.
+ `"@@operators_allowed_for_child_policies"`:`["@@all"]` – Child OUs and accounts can use any operator in tag policies\. By default, all operators are allowed in child policies\.
+ `"@@operators_allowed_for_child_policies"`:`["@@assign", "@@append", "@@remove"]` – Child OUs and accounts can use only the specified operators in tag policies\. You can specify one or more value\-setting operators in this child control operator\.
+ `"@@operators_allowed_for_child_policies"`:`["@@none"]` – Child OUs and accounts can't use operators in tag policies\. To effectively lock down values that are defined in a parent policy so that child policies can't add, append, or remove values, use this operator\.

**Note**  
If an inherited child control operator limits the use of an operator, you can't reverse that rule in a child tag policy\. If you include child control operators in a parent policy, they limit the value\-setting operators in all child policies\.

## Tag Policy Inheritance Examples<a name="inheritance-examples"></a>

These examples show how policy inheritance works by showing parent and child tag policies are merged into an effective tag policy for an account\.

The examples assume that you have the following organization structure\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/organizations/latest/userguide/images/org-structure-inheritance.png)

### Example 1: Allow Child Policies to Overwrite Tag Values<a name="example-assign-operator"></a>

The following tag policy defines the `CostCenter` tag key and acceptable values\. If you attach it to the organization root, the tag policy is in effect for all accounts in the organization\. 

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
                    "ec2:*",
                    "dynamodb:table"
                ]
            }
        }
    }
}
```

Specifying the `@@assign` operator for the tag does the following when policy A and policy B merge to form the *effective tag policy* for an account:
+ Policy B overwrites the two tag values that were specified in the parent policy, policy A\. `Sandbox` is only compliant value for the `CostCenter` tag key\.
+ The addition of `enforced_for` specifies that the `CostCenter` tag must be used the specified tag value on all Amazon EC2 resources and Amazon DynamoDB tables\.

As shown in the diagram, OU1 includes two accounts: 111111111111 and 222222222222\. 

**Effective tag policy for accounts 111111111111 and 222222222222**

```
{
    "tags": {
        "costcenter": {
            "tag_key": "CostCenter",
            "tag_value": [
                "Sandbox"
            ],
            "enforced_for": [
                "ec2:*",
                "dynamodb:table"
            ]
        }
    }
}
```

### Example 2: Append New Values to Inherited Tags<a name="example-append-operator"></a>

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
                    "ec2:*",
                    "dynamodb:table"
                ]
            }
        }
    }
}
```

Attaching policy C to OU2 has the following effects when policy A and policy C merge to form the effective tag policy for an account:
+ Because policy C includes the `@@append` operator, it allows for *adding to*, not overwriting, the list of acceptable tag values that are specified in Policy A\.
+ As in policy B, the addition of `enforced_for` specifies that the `CostCenter` tag must be used the specified tag value on all Amazon EC2 resources and Amazon DynamoDB tables\. Overwriting \(`@@assign`\) and adding \(`@@append`\) have the same effect if the parent policy doesn't include a child control operator that restricts what a child policy can specify\.

As shown in the diagram, OU2 includes one account: 999999999999\. Policy A and policy C merge to create the effective tag policy for account 999999999999\.

**Effective tag policy for account 999999999999**

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
                "ec2:*",
                "dynamodb:table"
            ]
        }
    }
}
```

### Example 3: Remove Values from Inherited Tags<a name="example-remove-operator"></a>

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
                        "ec2:*",
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

### Example 4: Restrict Changes to Child Policies<a name="example-restrict-child"></a>

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

### Example 5: Conflicts with Child Control Operators<a name="multiple-same-node-operators"></a>

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

### Example 6: Conflicts with Appending Values at Same Hierarchy Level<a name="multiple-same-node-values"></a>

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