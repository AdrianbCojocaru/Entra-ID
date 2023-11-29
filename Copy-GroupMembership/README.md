# Microsoft Entra ID copy group membership

## Synopsys
Copy the group membership from one or more Azure Active Directory (Entra ID) groups to a single destination group.

# Description
The new members will always be added to the destionation group (if they don't already exists).  
Members will never be removed from the destination group even if they are removed from the source group(s).  
Uses a JSON configuration file stored on blob storage that defines the source & target groups.    
When new groups are added, only this file will change.  
Initially developed for Intune's EPM component removal.

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


## Data Flow Description
| Step name     | Associated description |
| ------------- | ---------------------- |
| 1. Certificate-based authentication | A certificate is added to the automation account certificate store. This certificate is used to authenticate with Microsoft Graph and Microsoft Threat Protection on the behalf of the  application. |
| 2. Access Tokens | The OAuth 2.0 tokens will be received for Microsoft Graph and Threat Protection. |
| 3. REST API call to blob storage | A SAS token, stored as and encrypted variable insinde an automation account, is used to retrieve the config file. This file is *.json based and contains the configuration for the AAD groups that will be updated. |
| 4. Get Configuration file | The configuration file is being returned and processed. |
| 5. Rest API calls to Microsoft Entra ID | Multiple REST API calls using the Microsoft Graph token will be performed to validate the groups defined in the configuration file, get devices for each user part of the source (user) groups and update the membership of the destination (device) groups. |

## Config.json

```xml
[
    {
        "Creator":"Adrian Cojocaru",
        "Description":"Copy group members from groups GroupName1 to group DestinationGroupName.",
        "SourceAzureADGroupIds":"4be1e852-f6bb-4810-beb8-d96978516928",
        "SourceAzureADGroupNames":"GroupName1",
        "DestinationAzureADGroupId":"3ab6e913-478a-41e3-89b1-3e3c9930935c",
        "DestinationAzureADGroupName":"DestinationGroupName"
    },
    {
        "Creator":"Adrian Cojocaru",
        "Description":"Copy group members from groups SourceGroupName1 and SourceGroupName2 to group DestinationGroupName.",
        "SourceAzureADGroupIds":"62e3b5f7-c994-4916-b5d4-e84f50dcd3b0,a4a749b1-a962-477a-b360-6cb19f55dbae",
        "SourceAzureADGroupNames":"SourceGroupName1,SourceGroupName2",
        "DestinationAzureADGroupId":"89a9c485-de45-4567-9f07-df36e640d9fe",
        "DestinationAzureADGroupName":"DestinationGroupName"
    }
]
```

## Config.json notes
Multiple source groups must be separated by comma for both Name and ID. Please see the example above.  
**SourceAzureADGrouplds** have to match the **SourceAzureADGroupNames** for each of the groups. If they don't match, the current section in the *.json file will be skipped.