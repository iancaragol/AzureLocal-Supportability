# Symptoms
Azure Local cluster deployment fails with below error message:
```
Type 'CleanUpDeploymentPackageAndNugetStoreOnNonSeedNodes' of Role 'BareMetal' raised an exception:\n\nThe specified module 'CloudCommon' was not loaded because no valid module file was found in any module directory.\nat CleanUpDeploymentPackageAndNugetStoreOnNonSeedNodes, C:\\CloudDeployment\\Classes\\BareMetal\\BareMetal.psm1: line 557\nat <ScriptBlock>, C:\\CloudDeployment\\ECEngine\\InvokeInterfaceInternal.psm1: line 139\nat Invoke-EceInterfaceInternal, C:\\CloudDeployment\\ECEngine\\InvokeInterfaceInternal.psm1: line 134\
```
# Issue Validation
Azure Local Cluster Deployment of 2411.3 (or earlier) fails after the Azure Local AzureEdgeLifecyleManager Extension was installed or updated to Version 30.2503.0.881.

Verify if the **CloudCommon** folder exists in ```C:\Program Files\WindowsPowerShell\Modules\``` on non-seed nodes. If it does not exist, apply the below the mitigation.
<br />*"The seed node is usually the first node in the cluster during deployment. All other nodes are non-seed nodes*

# Cause
The missing CloudCommon folder on non-seed nodes is causing the deployment error when the latest (30.2503.0.881 AzureEdgeLifecyleManager Extension attempts deployment of 2411.3 (or earlier) builds of Azure Local.

_**Note:** While the AzureEdgeLifecyleManager Extension Version 30.2503.0.881 has been rolled-back (latest version once again is 30.2411.2.789), there are still clusters that will experience this issue and need to perform the manual mitigation list below._

# Mitigation Details
- Manually copy the ```C:\Program Files\WindowsPowerShell\Modules\CloudCommon``` folder from seed node to all other nodes in the cluster to ```C:\Program Files\WindowsPowerShell\Modules\```