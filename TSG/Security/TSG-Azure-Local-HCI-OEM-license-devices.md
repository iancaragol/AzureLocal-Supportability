
# [2411.2][2411.3]  AzureStackOSConfig ConfigureSecurityBaseline failure during deployment on Azure Stack HCI OEM license devices due to license corruption on nodes.


# Symptoms

AzureLocal 2411.2 or 2411.3 deployment failed at `Apply security settings on servers` step with the following error message:
```
Type 'ConfigureSecurityBaseline' of Role 'AzureStackOSConfig' raised an exception: [ConfigureSecurityBaseline] ConfigureSecurityBaseline failed on xxx with exception: -> Failed to apply OSConfiguration enforcement for ASHCIApplianceSecurityBaselineConfig on xxx after 5 attempts
```
# Validation


First confirm this is **AzureLocal 2411.2** or **AzureLocal 2411.3** deployment on **Azure Stack HCI OEM license** devices.

If the above condition matches, please go to one of the nodes. Go to `D:\CloudContent\MASLogs\ASSecurityOSConfigLogs` (Or for some environments: `C:\CloudContent\MASLogs\ASSecurityOSConfigLogs`) and check the latest `ASOSConfig_SetASOSConfigDocInternal_*.log` file. There should be failures like 
```
...
WARNING:     Setting: InteractiveLogon_SmartCardRemovalBehavior. State: failed. Error: 0xc004f012.
WARNING:     Setting: WinlogonCachedLogonsCount. State: failed. Error: 0xc004f012.
WARNING:     Setting: Audit_AuditTheUseOfBackupAndRestoreprivilege. State: failed. Error: 0xc004f012.
...
```

If you find the error code **0xc004f012** with 80+ (a number greater than 80, sometimes can be 90+ or more) total policy failures (the number can be found in failure report summary in the log file), the issue is confirmed.

# Mitigation Details

**WARNING for AzureLocal 2411.2 deployment:** Please follow the below mitigation steps to reimage and deploy **AzureLocal 2411.3** instead. **There is no mitigation steps provided for AzureLocal 2411.2 deployment.** 

1. Please **reimage** all of the hosts that are going to be added to the cluster **with available AzureLocal 2411.3 image**. The following steps need to be applied directly **after all hosts are reimaged** and **_before_ ARC registration**

2. Once **reimage** is done, execute the below commands on **all** of the hosts.
``` Powershell
$downloadLocationOfZip  = "C:\DownloadKIRTMP"
mkdir $downloadLocationOfZip -Force
```

3. Download the remediation package [here](https://azurestackreleases.download.prss.microsoft.com/dbazure/AzureStackHCI/Update/10.2411.3.2/DeployKIRNuget.zip) and copy to `C:\DownloadsKIRTMP` on **all** hosts.

4. Execute the below commands on **all** of the hosts.
``` Powershell
$downloadLocationOfZip  = "C:\DownloadKIRTMP"
mkdir C:\KIRPackage -Force
Expand-Archive $downloadLocationOfZip\DeployKIRNuget.zip C:\KIRPackage -f
```
5. Run the DeployKIRNuget.ps1 file in the downloaded package on **all** hosts.
``` Powershell
C:\KIRPackage\DeployKIRNuget.ps1
```
6. Check that on **all** hosts, under `C:\NugetStore`, there is a folder called `Microsoft.AS.KIR.10.2411.3.3`.
7. Proceed with the normal deployment workflow, including ARC registration.