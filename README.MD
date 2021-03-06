# PSAutomator - PowerShell Module

Overview of this module is available at: https://evotec.xyz/hub/scripts/psautomator-powershell-module/

### Description

This PowerShell Module is new approach to onboarding, offboarding and business as usual processes running in companies infrastructure. Usually each company has different rules, different approaches on how processes should look like and this module takes an easy approach that's similar to what you can find in services like **IFTTT** or **Microsoft Flow**. Those services work in known schema such as Services, Triggers, Ingredients and Applets. For this PowerShell Module I've taken similar approach which is described below.

While it's encouraged to keep correct order Service, Trigger, Ignore, Condition, Action it actually only requires Service, Trigger to be in correct order.

- [x] **Service** – is kind of a wrapper for other blocks above. It has also ability to load configuration from file. Currently loading file doesn't do anything.
- [x] **Trigger** – is first block in Service. Before an Action can be executed it needs a **Trigger**. **Trigger** can be membership in Group, Organizational Unit etc
    - [ ] User Based Triggers
        - [ ] Always,
        - [x] OrganizationalUnit,
        - [x] GroupMembership,
        - [ ] Filter
- [x] **Ignore** – But Trigger can also have things that need to be ignored. For example lack of email address field.
    - [ ] User Based Ignore
        - [x] MatchingEmptyOrNull,
        - [ ] MatchingObjects,
        - [ ] MatchingFields
- [x] **Condition** – It can also be conditioned for example Last User Modification Date should be more then 30 days.
    - [ ] User Based Conditions
        - [ ] RequireGroupMembership,
        - [ ] RequireLastModified
- [x] **Action** – are essentially Tasks that are about to be executed. This can be adding a user to a group, disabling user etc. This is also final step to close Service.
    - [ ] User Based Actions
        - [x] AccountAddGroupsSpecific
        - [x] AccountDisable
        - [x] AccountEnable
        - [x] AccountHideInGAL
        - [x] AccountShowInGAL
        - [x] AccountRemoveGroupsAll
        - [x] AccountRemoveGroupsSecurity
        - [x] AccountRemoveGroupsDistribution
        - [x] AccountRemoveGroupsSpecific
        - [x] AccountRemoveGroupsDomainLocal
        - [x] AccountRemoveGroupsGlobal
        - [x] AccountRemoveGroupsUniversal
        - [ ] AccountRename,
        - [x] AccountSnapshot

Keep in mind that following is true for Service:
- [x] Can hold only 1 trigger
- [x] Can have multiple Ignore blocks
- [x] Can have multiple Condition blocks
- [x] Can have multiple Action blocks

**This is proof-of-concept. Heavy work in progres... Please take your time and leave feedback!**

## DO NOT USE IN **PRODUCTION** YET - UNLESS YOU LIKE FIRES!

### Sample offboarding

![image](https://evotec.xyz/wp-content/uploads/2018/10/img_5bce250f1fe42.png)

```powershell
Clear-Host
Import-Module PSAutomator -Force #-Verbose
Import-Module PSSharedGoods -Force
Service -Name 'Active Directory Offboarding' -ConfigurationPath 'C:\Support\GitHub\PSAutomator\Examples\MyConfiguration.xml' {
    Trigger -Name 'OU Offboarded Users' -User OrganizationalUnit -Value 'OU=Users-Offboarded,OU=Production,DC=ad,DC=evotec,DC=xyz' |
        Condition -Name 'No conditions' |
        Ignore -Name 'Ignore Windows Email Address if Empty or null' -Ignore MatchingEmptyOrNull -Value EmailAddress |
        Action -Name 'Make User Snapshot' -ActiveDirectory AccountSnapshot -Value 'C:\Users\pklys\Desktop\MyExport' |
        Action -Name 'Disable AD Account' -ActiveDirectory AccountDisable |
        Action -Name 'Hide account in GAL' -ActiveDirectory AccountHideInGAL |
        Action -Name 'Remove all security groups' -ActiveDirectory AccountRemoveGroupsSecurity |
        Action -Name 'Rename Account' -ActiveDirectory AccountRename -Value @{ Action = 'AddText'; Where = 'After'; Text = ' (offboarded)'; }
}
```

### Sample onboarding

![image](https://evotec.xyz/wp-content/uploads/2018/10/img_5bce267af35e6.png)

```powershell
Clear-Host
Import-Module PSAutomator -Force #-Verbose
Import-Module PSSharedGoods -Force
Service -Name 'Active Directory Enable Users in OU' {
    Trigger -Name 'Find Offboarded Users' -User OrganizationalUnit -Value 'OU=Users-Offboarded,OU=Production,DC=ad,DC=evotec,DC=xyz' |
        Ignore |
        Action -Name 'Enable Offboarded Users' -ActiveDirectory AccountEnable |
        Action -Name 'Add to group GDS-TestGroup5' -ActiveDirectory AccountAddGroupsSpecific -Value 'GDS-TestGroup5' |
        Action -Name 'Add to group GDS-TestGroup4' -ActiveDirectory AccountAddGroupsSpecific -Value 'GDS-TestGroup4' |
        Action -Name 'Remove Offboarded Tag' -ActiveDirectory AccountRename -Value @{ Action = 'RemoveText'; Fields = 'DisplayName', 'Name' ; Text = ' (offboarded)'; }
}
```

## DO NOT USE IN **PRODUCTION** YET - UNLESS YOU LIKE FIRES!