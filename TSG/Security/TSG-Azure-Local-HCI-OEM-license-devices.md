
# [2411.2][2411.3]  AzureStackOSConfig ConfigureSecurityBaseline failure during deployment on Azure Stack HCI OEM license devices due to license corruption on nodes.


# Symptoms

AzureLocal 2411.2 or 2411.3 deployment failed at `Apply security settings on servers` step with the following error message:
```
Type 'ConfigureSecurityBaseline' of Role 'AzureStackOSConfig' raised an exception: [ConfigureSecurityBaseline] ConfigureSecurityBaseline failed on xxx with exception: -> Failed to apply OSConfiguration enforcement for ASHCIApplianceSecurityBaselineConfig on xxx after 5 attempts
```
# Issue Validation


First confirm this is **AzureLocal 2411.2** or **AzureLocal 2411.3** deployment on **Azure Stack HCI OEM license** devices.

If the above condition matches, please go to one of the nodes. Go to `D:\CloudContent\MASLogs\ASSecurityOSConfigLogs` (Or for some environments: `C:\CloudContent\MASLogs\ASSecurityOSConfigLogs`) and check the latest `ASOSConfig_SetASOSConfigDocInternal_*.log` file. There should be failures like 
```
...
WARNING:     Setting: InteractiveLogon_SmartCardRemovalBehavior. State: failed. Error: 0xc004f012.
WARNING:     Setting: WinlogonCachedLogonsCount. State: failed. Error: 0xc004f012.
WARNING:     Setting: Audit_AuditTheUseOfBackupAndRestoreprivilege. State: failed. Error: 0xc004f012.
...
```

If you encounter the error code **0xc004f012** and observe at least 80 total policy failures (as indicated in the failure report summary within the log file), the issue is confirmed.

# Mitigation Details

All following mitigation steps are applicable to both AzureLocal 2411.2 and 2411.3 releases. **However, there is no mitigation plan available for the continued deployment of AzureLocal 2411.2.** With the implementation of this mitigation, the environment will be deployed using AzureLocal 2411.3.

1. Please **reimage** all of the hosts that are going to be part of the cluster **with most-recent AzureLocal 2411.3 image**. The following steps need to be applied directly **after all hosts are reimaged** and **before ARC registration**. 

2. Install driver after reimaging the servers. Install the driver package provided by your server vendor.

3. Once **reimaging and driver installation** are done, execute the below commands on **all** of the hosts.
``` Powershell
$downloadLocationOfZip  = "C:\DownloadKIRTMP"
mkdir $downloadLocationOfZip -Force
```

4. Download the remediation package [here](https://azurestackreleases.download.prss.microsoft.com/dbazure/AzureStackHCI/Update/10.2411.3.2/DeployKIRNuget.zip) and copy to `C:\DownloadsKIRTMP` on **all** hosts.

5. Execute the below commands on **all** of the hosts.
``` Powershell
$downloadLocationOfZip  = "C:\DownloadKIRTMP"
mkdir C:\KIRPackage -Force
Expand-Archive $downloadLocationOfZip\DeployKIRNuget.zip C:\KIRPackage -f
```
6. Run the DeployKIRNuget.ps1 file in the downloaded package on **all** hosts.
``` Powershell
C:\KIRPackage\DeployKIRNuget.ps1
```
7. Check that on **all** hosts, under `C:\NugetStore`, there is a folder called `Microsoft.AS.KIR.10.2411.3.3`.
8. Proceed with the normal deployment workflow, including ARC registration.