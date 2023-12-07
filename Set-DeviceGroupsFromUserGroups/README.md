# Microsoft Entra ID copy group membership

## Synopsys
Update membership of Azure Active Directory (Entra ID) device groups based on user groups.

# Description
Source (user) groups and destination (device) groups are defined in the configuration file (JSON).  
The script gets all the devices for each member of the source group (including transitive members) and updates the membership of the destination group.  
**OperatingSystem**, **JoinType**,**Compliance** & **DeviceEnabled** filters are supported.

## Required Permissions
Microsoft Graph (2)  
    GroupMember.ReadWrite.All  
    User.Read.All  

## Components
Automation Account  
Runbook  
App registration  
Storage container  

## Data Flow Diagram
![alt text](https://github.com/AdrianbCojocaru/Entra-ID/raw/main/Set-DeviceGroupsFromUserGroups/Diagram.drawio.png "Set-DeviceGroupsFromUserGroups")

## Data Flow Description
| Step name     | Associated description |
| ------------- | ---------------------- |
| 1. Certificate-based authentication | A certificate is added to the automation account certificate store. This certificate is used to authenticate with Microsoft Graph on the behalf of the  application. |
| 2. Access Tokens | The OAuth 2.0 tokens will be received for Microsoft Graph. |
| 3. REST API call to blob storage | A SAS token, stored as and encrypted variable insinde an automation account, is used to retrieve the config file. This file is *.json based and contains the configuration for the AAD groups that will be updated. |
| 4. Get Configuration file | The configuration file is being returned and processed. |
| 5. Rest API calls to Microsoft Entra ID | Multiple REST API calls using the Microsoft Graph token will be performed to validate the groups defined in the configuration file, get devices for each user part of the source (user) groups and update the membership of the destination (device) groups. |

## Config.json

```xml
[
    {
        "Creator":"Adrian Cojocaru",
        "Description":"Updating [Devices] [Windows] Adrian Cojocaru device group with devices of users from Adrian Cojocaru - All user group.",
        "UserAzureADGroupId":"51782d82-f172-4b5b-8273-c7276262c44a",
        "UserAzureADGroupName":"Adrian Cojocaru - All",
        "DeviceAzureADGroupId":"75152bae-b436-4c81-8d9c-69c9e2afad5a",
        "DeviceAzureADGroupName":"[Devices] [Windows] Adrian Cojocaru",
        "OSList":"Windows",
        "TrustTypeList":"",
        "isCompliant":"Yes,No",
        "accountEnabled":"Yes"
    },
    {
        "Creator":"Adrian Cojocaru",
        "Description":"Updating [Devices] [MacOS] [Windows] DWC Team device group with devices belonging to users from [Users] DWC Team user group.",
        "UserAzureADGroupId":"e5186146-1669-4311-83cf-2b3f0c79dc99",
        "UserAzureADGroupName":"[Devices] DWC Team",
        "DeviceAzureADGroupId":"7f1401f3-5f01-4991-a5cf-6a303404b1f6",
        "DeviceAzureADGroupName":"[MacOS] [Windows] DWC Team",
        "OSList":"Windows,MacOS",
        "TrustTypeList":"AzureAd,ServerAd,Workplace",
        "isCompliant":"",
        "accountEnabled":""
    },
    {
        "Creator":"Adrian Cojocaru",
        "Description":"Updating [EPM] [Devices] TECH USER ADMIN PRD IGA device group with devices belonging to users from TECH USER ADMIN PRD IGA user group.",
        "UserAzureADGroupId":"9c09caa4-8696-4eab-8b05-d56845152381",
        "UserAzureADGroupName":"TECH USER ADMIN PRD IGA",
        "DeviceAzureADGroupId":"8e365573-f7ec-4336-bf7f-57e4a443c015",
        "DeviceAzureADGroupName":"[EPM] [Devices] TECH USER ADMIN PRD IGA",
        "OSList":"Windows",
        "TrustTypeList":"",
        "isCompliant":"",
        "accountEnabled":""
    },
    {
        "Creator":"Adrian Cojocaru",
        "Description":"Updating [EPM] [Devices] [Windows] FUNC USER ADMIN PRD IGA device group with devices belonging to users from FUNC USER ADMIN PRD IGA user group.",
        "UserAzureADGroupId":"53b94e63-36f7-4109-afda-204e4ccd4e89",
        "UserAzureADGroupName":"FUNC USER ADMIN PRD IGA",
        "DeviceAzureADGroupId":"4b8dc31c-f6d6-4c7a-b43b-cea4e550bc2b",
        "DeviceAzureADGroupName":"[EPM] [Devices] [Windows] FUNC USER ADMIN PRD IGA",
        "OSList":"Windows",
        "TrustTypeList":"",
        "isCompliant":"Yes,No",
        "accountEnabled":"Yes,No"
    },
    {
        "Creator":"Fraser Menzies",
        "Description":"Updating [Devices] [Windows] Digital Workplace M365 Copilot Devices device group with devices belonging to users from UR Digital Workplace M365 Copilot Users user group.",
        "UserAzureADGroupId":"9cd02489-b2b6-47bf-ae67-4fc6a6365a2b",
        "UserAzureADGroupName":"UR Digital Workplace M365 Copilot Users",
        "DeviceAzureADGroupId":"d87a874a-9d37-4468-ade9-545d24a54af5",
        "DeviceAzureADGroupName":"[Devices] [Windows] Digital Workplace M365 Copilot Devices",
        "OSList":"Windows",
        "TrustTypeList":"AzureAd,ServerAd",
        "isCompliant":"Yes",
        "accountEnabled":"Yes"
    }
     
]
```

## Config.json notes
**UserAzureADGroupId** has to match the **UserAzureADGroupName** and **DeviceAzureADGroupId** has to match the **DeviceAzureADGroupName**. If they don't match, the current section in the *.json file will be skipped.  

**OSList** is used to select only devices with a certain OS. Accepts multiple values separated by comma.  
Valid values: Windows,MacOS,IPhone,IPad,Android.  
If this property is left empty, then it will default to all the valid values.  

**TrustTypeList** is the equivalent of the **Join type** property from the UI for a certain device. Accepts multiple values separated by comma.  
Valid values: AzureAd,ServerAd,Workplace.  
If this property is left empty, then it will default to all the valid values.  

| TrustTypeList | Join Type |
| ------------- | ---------------------- |
| AzureAd | Microsoft Entra joined |
| ServerAd | Microsoft Entra hybrid joined |
| Workplace | Microsoft Entra registered |

**isCompliant** is equivalent to the **Compliant** property for a device in AzureAD.  
Valid values: Yes,No  
"isCompliant":"Yes,No" is the same as "isCompliant":""  
If this property is left empty, then it will get both Compliant and Noncompliant devices.  

**accountEnabled** is equivalent to the **Enabled** property for a device in AzureAD.  
Valid values: Yes,No  
"accountEnabled":"Yes,No" is the same as "accountEnabled":""  
If this property is left empty, then it will get devices that are enabled and devices that are disabled.  