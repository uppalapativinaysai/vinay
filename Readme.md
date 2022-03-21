
# Azure - Create New VMs

### Overview 
This automation script create/s new VM/s from an input file (CSV) using the offering templates.
The following procedures are included:
- Checks for validity of required parameters (from an input file)
- Creates the ARM parameter template file based on the input file
- Deploys a VM from an ARM template and parameter file.

### Prerequisites
This section lists the requirements to run the script successfully.
1. Access to subscription and role-based access with Contributor privileges
2. PowerShellCore version 7.0 or higher
3. PowerShell Az module version 5.0 or higher
4. Ensure the following secrets are onboarded to the specified key vault:
    | Secret name | Desription |
    |:--- |:---
    |csUninstallPW|Password to prevent unauthorized uninstall of Crowdstrike.|
    |domainUsername|Administrator of the account on the domain (used for Windows builds).|
    |domainPassword|Password of the account on the domain (used for Windows builds).|

### Parameter and syntax
Use the following command to run the script:

```powershell
.\LUXG-Create-New-VMs.ps1 -tenantId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" -vmList "<FILE_PATH>\vmList.csv"
```

   | Parameter | Type |Mandatory| Description |
   |:--- |:--- |:---|:---
   |tenantId|string|Yes| Azure tenant Id. |
   |vmList|string|Yes| Absolute file path of the file which contains the VM build configuration. |

The input file "vmList.csv" is included in the package script. The following input are required:
  
   | Column name | Type | Mandatory | Default/allowed values | Description |
   |:--- |:--- |:---|:---|:---
   |subscriptionId|string|Yes|None|Azure subscription Id.|
   |dxcManaged|boolean|Yes|TRUE, FALSE|Required tag to indicate whether VM is managed by DXC.|
   |dxcMonitored|boolean|Yes|TRUE, FALSE|Required tag to indicate whether VM is monitored by DXC.|
   |dxcBackup|boolean|Yes|TRUE, FALSE|Required tag to indicate whether VM is backed up by DXC.|
   |dxcPatchGroup|boolean|Yes|TRUE, FALSE|Required tag to indicate whether VM is patched up by DXC. Format as follows if TRUE is "{ \"Required\": \"True\",\"PatchGroupName\": \"WindowsMonthly\"}" |
   |application|string|Yes|None|Required tag to indicate the name of the hosted application.|
   |departmentName|string|Yes|None|Required tag to indicate the department owning the server.|
   |project|string|Yes|None|Required tag to indicate the name of the project name.|
   |Username|string|Yes|None|Local administrator account|
   |Password|securestring|Yes|None|Local administrator password. Must contain at least 12 characters, 1 upper case letter, 1 lower case letter, 1 number, and 1 special character.|
   |vmName|string|Yes|None|Name of the Virtual Machine. Maximum of 15 characters.|
   |vmRG|string|Yes|None|Resource Group of the Virtual Machine|
   |osName|string|Yes|Windows, Centos, Ubuntu, Suse, RHEL, Oracle|Operating system type of the Virtual Machine|
   |osVersion|string|Yes|Refer to [Supported OS Versions](https://confluence.dxc.com/display/CSA/Supported+OS+Versions) for the list of supported versions.|Operating system version of the Virtual Machine|
   |vmSize|string|Yes|List of Azure VM sizes are [here](https://docs.microsoft.com/en-us/azure/virtual-machines/sizes). Note: Availability of VM sizes are governed per region. |Size (T-shirt) of the Virtual Machine|
   |vmLocation|string|Yes|None|Region the Virtual Machine will be placed|
   |OSDiskType|string|Yes|Standard_LRS, StandardSSD_LRS, Premium_LRS|The storage type of the OS disk. Standard_LRS is standard HDD, StandardSSD_LRS is standard SSD, Premium_LRS is premium SSD. Not all VM Sizes support premium SSD, refer to [Azure VM Sizes](https://docs.microsoft.com/en-us/azure/virtual-machines/sizes) for more details.|
   |numberOfDataDisks|integer|Yes|0-64 (range of min/max values)|Number of data disks to create.|
   |dataDiskType|string|Yes|Standard_LRS, StandardSSD_LRS, Premium_LRS|The storage type of the OS disk. Standard_LRS is standard HDD, StandardSSD_LRS is standard SSD, Premium_LRS is premium SSD. Not all VM Sizes support premium SSD, refer to [Azure VM Sizes](https://docs.microsoft.com/en-us/azure/virtual-machines/sizes) for more details.|
   |dataDiskSize|integer|Yes|1-32767 (range of min/max values). Set at 128 by default.|Size of each data disk in GB|
   |dataDiskHostCaching|string|Yes|None, ReadOnly, ReadWrite. Set to "None" by default.|Specifies the caching mechanism for data disks.|
   |existingvNetName|string|Yes|None|Existing Virtual Network that the VM NIC will be attached to. It should be at the same location as the Virtual Machine|
   |existingvNetResourceGroup|string|Yes|None|Resource Group of the Existing Virtual Network.|
   |subnetName|string|Yes|None|The Subnet for the Virtual Machine.|
   |publicIPRequired|string|Yes|yes, no|Does this Virtual Machine need a public IP? (yes/no)|
   |KeyVaultName|string|Yes|None|Name of the Key Vault|
   |KeyVaultRG|string|Yes|None|Resource Group of the Key Vault|
   |existingBootDiagStorageName|string|Yes|None|Existing Storage Account where boot diagnostics will be stored.|
   |existingBootDiagStorageResourceGroup|string|Yes|None|Resource group of the existing Storage Account where boot diagnostics will be stored.|
   |workspaceName|string|Yes|None|Existing Log Analytics Workspace where VM agent sends log and metric data.|
   |workspaceRG|string|Yes|None|Resource group of the existing Log Analytics Workspace where VM agent sends log and metric data.|
   |dxcEPAgent|string|Yes|crowdstrike, azureDefender, false|Workload/endpoint security agent.|
   |existingRecoveryServicesVault|string|Yes|None|Recovery Services Vault name where the VMs will be backed up to. If 'dxcBackup' is FALSE, set value to "None".|
   |existingRecoveryServicesVaultRG|string|Yes|None|Resource group of the existing Recovery Services Vault name where the VMs will be backed up to. If 'dxcBackup' is FALSE, set value to "None" by default.|
   |existingBackupPolicy|string|Yes|DefaultPolicy, DXC30day, DXC60day, DXC90day|Backup policy to be used to backup the Virtual Machine. If 'dxcBackup' is FALSE, set value to "None".|
   |Availability|string|Yes|No Availability Required, AvailabilitySet, AvailabilityZone. Set to "No Availability Required" by default.|Resourcegroup of the virtual machine|
   |existingAvailabilitySetName|string|Yes|None|Existing Availability Set Name located under the same resource group and location as the Virtual Machine. If 'Availability' is set to "No Availability Required", set value to "None".|
   |zone|integer|Yes|1, 2, 3|The number for the availability zone; will only apply if 'Availability' is set to "AvailabilityZone", set value to 1 by default.|
   |domainToJoin|string|Yes|None|The FQDN of the AD domain.|
   |ouPath|string|Yes|None|Specifies an organizational unit (OU) for the domain account.|

### Execution

1. Download the script package from the GitHub repository ***cloud\client-luxottica*** to your preferred directory.

2. Open PowerShellCore console (run as Administrator) and switch to script directory ***.\cloud\client-luxottica\VM-Build-Automation***.

3. Modify the input file ***vmList.csv*** with your build specifications. Copy over/place the script ***LUXG-Create-New-VMs.ps1*** to the same directory where the offering templates are stored.

4. Execute the following command

    ```powershell
    .\LUXG-Create-New-VMs.ps1 -tenantId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" -vmList "<FILE_PATH>\vmList.csv"
    ```

5. Wait until script execution completes.
