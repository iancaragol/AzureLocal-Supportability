# Overview

Environment Validator runs physical disk checks during Deployment, Update and ScaleOut actions in AzureLocal. The tool detects issues with data disk requirements for Storage Spaces.

# Symptoms

During validation, the Environment Validator for Hardware has a failed critical result and deployment is unable to continue:

```
2024-01-10 11:43:11 Warning  [EnvironmentValidator:ValidateHardware] Critical (blocking) Hardware failures found in validation. 8 Rules failed: 
2024-01-10 11:43:11 Verbose  [EnvironmentValidator:ValidateHardware] Validator failed. Hardware requirements not met. Review output and remediate: 
Rule: 

HealthCheckSource  : Deployment\Hardware\fd4a63f1
Name               : AzStackHci_Hardware_Test_PhysicalDisk
DisplayName        : Test PhysicalDisk API Node01
Tags               : {}
Title              : Test PhysicalDisk API
Status             : FAILURE
Severity           : CRITICAL
Description        : Checking PhysicalDisk has CIM data
Remediation        : https://learn.microsoft.com/en-us/azure-stack/hci/deploy/deployment-tool-prerequisites
TargetResourceID   : Machine: Node01, Class: PhysicalDisk
TargetResourceName : Machine: Node01, Class: PhysicalDisk
TargetResourceType : PhysicalDisk
Timestamp          : 1/10/2024 7:43:03 PM



 AdditionalData: 

Key   : Detail
Value : Unable to retrieve data for PhysicalDisk on Node01

Key   : Status
Value : FAILURE

Key   : TimeStamp
Value : 01/10/2024 19:43:03

Key   : Resource
Value : Null

Key   : Source
Value : Node01
```

This result means the validation did not find any disks suitable to use as data disks in Storage Spaces. 

# Issue Validation

If impacted, when the following PowerShell is run on a node, it returns no result.

```
$allowedBusTypes = @('SATA', 'SAS', 'NVMe', 'SCM')
$allowedMediaTypes = @('HDD', 'SSD', 'SCM')
$bootPhysicalDisk = Get-Disk | Where-Object {$_.IsBoot -or $_.IsSystem} | Get-PhysicalDisk
Get-StorageNode -Name $env:COMPUTERNAME* | `
    Get-PhysicalDisk -PhysicallyConnected | `
    Where-Object { `
        $_.BusType -in $allowedBusTypes -and `
        $_.MediaType -in $allowedMediaTypes -and `
        $_.DeviceId -notin $bootPhysicalDisk.DeviceId -and `
        $_.CanPool -eq $true
    }
```

# Cause

The cause can be one or more of the following:
- Data Disks must be the right BusType: (SATA, SAS, NVMe or SCM).
- Data Disks must be the right MediaType: (HDD, SSD, SCM).
- Data Disks must not a boot device.
- Data Disks CanPool property must be true.
- Data Disk must be consistent across all Azure Local nodes.

# Mitigation Details

To verify which of the requirements are not met, run the following on all nodes.

```
Get-StorageNode -Name $env:COMPUTERNAME* | Get-PhysicalDisk -PhysicallyConnected | Format-List PhysicalLocation, UniqueId, SerialNumber, CanPool, CannotPoolReason, BusType, MediaType, Size
```

Common issues:  
- Data Disks CanPool property is false, from the output above there will be a CannotPoolReason property indicating why the disk cannot be pooled. Check the following article to help with remediation https://learn.microsoft.com/en-us/windows-server/storage/storage-spaces/storage-spaces-states
- Data Disk BusType is RAID. For Azure Local RAID controller cards or SAN (Fibre Channel, iSCSI, FCoE) storage, shared SAS enclosures connected to multiple machines, or any form of multi-path IO (MPIO) where drives are accessible by multiple paths, aren't supported. https://learn.microsoft.com/en-us/azure/azure-local/concepts/system-requirements-23h2?view=azloc-24113#machine-and-storage-requirements