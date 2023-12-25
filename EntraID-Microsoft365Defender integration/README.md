# Microsoft Entra ID copy group membership

## Synopsys
Update Azure Active Directory (Entra ID) group membership automatically based on Microsoft 365 Defender (Defender XDR) advanced hunting queries.

# Description
The Advanced Hunting Query needs to return the AadDeviceId property for each device. The Azure AD Device Id is used to identify the device in Microsoft Graph.  
An external *.json file stored on blob storage. This file is used to specify the group that is being updated and the coresponding Advanced Hunting for Microsoft 365 Defender.  
This script reads the Json file and for each section it will update the AzureAD group membershipp with devices returned by the query.
This script is designed to run in a PowerShell Runbook using either certificate or App Id & Secret authentication. 

## Required Permissions
Microsoft Graph (2)  
    GroupMember.ReadWrite.All  
    Device.Read.All  
Microsoft Threat Protection (1)  
    AdvancedHunting.Read.All  

## Components
Automation Account  
Runbook  
App registration  
Storage container  

## Data Flow Diagram
![alt text](https://github.com/AdrianbCojocaru/Entra-ID/raw/main/EntraID-Microsoft365Defender%20integration/Diagram.drawio.png "Set-AADGroupsFromAdvancedHunting")

## Data Flow Description
| Step name     | Associated description |
| ------------- | ---------------------- |
| 1. Certificate-based authentication | A certificate is added to the automation account certificate store. This certificate is used to authenticate with Microsoft Graph and Microsoft Threat Protection on the behalf of the  application. |
| 2. Access Tokens | The OAuth 2.0 tokens will be received for Microsoft Graph and Threat Protection. |
| 3. REST API call to blob storage | A SAS token, stored as and encrypted variable insinde an automation account, is used to retrieve the config file. This file is *.json based and contains the configuration for the AAD groups that will be updated. |
| 4. Get Configuration file | The configuration file is being returned and processed. |
| 5. Rest API calls to Microsoft Entra ID | A REST API call using the Microsoft Threat Protection obtained at step 2 is performed. The request body included the Advanced Hunting query defined in the configuration file. |
| 6. Get device list | The previous call returns a list of devices that will update the membership for the AAD group defined in the configuration file. |
| 7. Rest API calls to Microsoft Entra ID | Multiple REST API calls using the Microsoft Graph token will be performed to validate the group and devices that Microsoft Threat Protection returned. The membership of an AAD group will be updated with the valid devices as long as it won't lose more than 99% of its members. This is a safeguard, and it is subject to change. |

## Config.json

```xml
[
    {
        "Creator": "Adrian Cojocaru",
        "Description": "Returns all Windows devices that have Adobe Shocwave installed.",
        "AzureADGroupId": "c8993ca6-28ed-4c09-9909-732a5f9d40bf",
        "AzureADGroupName": "[Global] CP Adobe Shockwave Removal",
        "AdvancedHuntingQuery": "DeviceInfo | where MachineGroup in~ (\"WS Users & Workstations\" , \"WS Unrestricted Internet Access\") | project DeviceId, DeviceName, OSArchitecture, AadDeviceId | join kind=innerunique (DeviceTvmSoftwareEvidenceBeta | where SoftwareName == \"shockwave_player\" | project DeviceId, DiskPaths, RegistryPaths, SoftwareVersion) on DeviceId"
    },
    {
        "Creator": "Adrian Cojocaru",
        "Description": "Returns all Windows devices that have Git installed.",
        "AzureADGroupId": "1210937d-4c25-4939-95ea-7621fe1d8d70",
        "AzureADGroupName": "[VulnMgmt] [Test] Git update",
        "AdvancedHuntingQuery": "DeviceTvmSoftwareEvidenceBeta | where SoftwareVendor == \"git-scm\"  | join (DeviceInfo | where MachineGroup in~ (\"WS Users & Workstations\" , \"WS Unrestricted Internet Access\") and DeviceType == \"Workstation\") on DeviceId | summarize any(DeviceId, DeviceName, LoggedOnUsers, SensorHealthState, RegistryDeviceTag, DeviceManualTags, DeviceDynamicTags) by AadDeviceId"
    },
    {
        "Creator": "Adrian Cojocaru",
        "Description": "Returns all Windows devices that have Python installed.",
        "AzureADGroupId": "0733f249-6335-419f-ab16-78abb9d5623b",
        "AzureADGroupName": "[VulnMgmt] Global Python update",
        "AdvancedHuntingQuery": "DeviceInfo | where MachineGroup in~ (\"WS Users & Workstations\", \"WS Unrestricted Internet Access\") | project DeviceId, DeviceName, OSArchitecture, AadDeviceId | join kind=innerunique ( DeviceTvmSoftwareEvidenceBeta | where SoftwareName == \"python\" and SoftwareName != \"python_launcher\" and (DiskPaths contains \"C:\\\\Program Files\\\\Python\" and DiskPaths !contains \"anaconda\" ) | project DeviceId, DiskPaths, RegistryPaths, SoftwareVersion,SoftwareName) on DeviceId | project DeviceId, DeviceName, OSArchitecture, AadDeviceId, DiskPaths, RegistryPaths, SoftwareVersion, SoftwareName"
    }
]
```

## Config.json notes
**AzureADGroupId** has to match the **AzureADGroupName**. If they don't match, the current section in the *.json file will be skipped.  
The **AdvancedHuntingQuery** needs to return the **AadDeviceId**. The Azure AD Device Id is used to identify the device in Microsoft Graph.