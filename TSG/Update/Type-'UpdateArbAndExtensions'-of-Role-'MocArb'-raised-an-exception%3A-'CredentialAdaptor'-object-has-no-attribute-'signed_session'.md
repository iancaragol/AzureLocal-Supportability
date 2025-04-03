# Type 'UpdateArbAndExtensions' of Role 'MocArb' raised an exception: 'CredentialAdaptor' object has no attribute 'signed_session'

Azure Local is configured with a specific version of Az CLI that is codesigned for each release, updating it out-of-band can cause unexpected issues. There is a known compatibility issue between the **Az CLI (2.70)** version (or higher, or different from what was shipped with Azure Local) and **customlocation (0.1.3)**. This might cause the following error message when running a customlocation cmdlet with this error:


```
Type 'UpdateArbAndExtensions' of Role 'MocArb' raised an exception:
[ERROR: The command failed with an unexpected error. Here is the traceback: ERROR: 'CredentialAdaptor' object has no attribute 'signed_session' Traceback (most recent call last):   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\knack/cli.py", line 233, in invoke   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\azure/cli/core/commands/__init__.py", line 666, in execute   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\azure/cli/core/commands/__init__.py", line 734, in _run_jobs_serially   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\azure/cli/core/commands/__init__.py", line 703, in _run_job   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\azure/cli/core/commands/__init__.py", line 336, in __call__   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\azure/cli/core/commands/command_operation.py", line 362, in handler   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\azure/cli/core/commands/arm.py", line 432, in show_exception_handler   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\azure/cli/core/commands/command_operation.py", line 360, in handler   File "C:\CloudContent\AzCliExtensions\customlocation\azext_customlocation\custom.py", line 142, in get_customlocation     return cl_client.get(resource_group_name=resource_group_name, resource_name=cl_name)            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   File "C:\CloudContent\AzCliExtensions\customlocation\azext_customlocation\vendored_sdks\azure\mgmt\extendedlocation\v2021_08_15\operations\_custom_locations_operations.py", line 282, in get     response = self._client.send(request, stream=False, **operation_config)                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\msrest/service_client.py", line 336, in send   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\msrest/pipeline/__init__.py", line 197, in run   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\msrest/pipeline/__init__.py", line 150, in send   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\msrest/pipeline/requests.py", line 65, in send AttributeError: 'CredentialAdaptor' object has no attribute 'signed_session' To check existing issues, please visit: https://github.com/Azure/azure-cli/issues]
```
**Azure Local is configured with a specific version of Az CLI that is codesigned for each release.**

# Symptoms
Running any customlocation cmdlet fails with the error message:
```PowerShell
# Example command:
# az customlocation show --subscription $subscription --resource-group $resourceGroup --name $customlocationName
[ERROR: The command failed with an unexpected error. Here is the traceback: ERROR: 'CredentialAdaptor' object has no attribute 'signed_session' Traceback (most recent call last):   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\knack/cli.py", line 233, in invoke   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\azure/cli/core/commands/__init__.py", line 666, in execute   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\azure/cli/core/commands/__init__.py", line 734, in _run_jobs_serially   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\azure/cli/core/commands/__init__.py", line 703, in _run_job   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\azure/cli/core/commands/__init__.py", line 336, in __call__   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\azure/cli/core/commands/command_operation.py", line 362, in handler   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\azure/cli/core/commands/arm.py", line 432, in show_exception_handler   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\azure/cli/core/commands/command_operation.py", line 360, in handler   File "C:\CloudContent\AzCliExtensions\customlocation\azext_customlocation\custom.py", line 142, in get_customlocation     return cl_client.get(resource_group_name=resource_group_name, resource_name=cl_name)            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   File "C:\CloudContent\AzCliExtensions\customlocation\azext_customlocation\vendored_sdks\azure\mgmt\extendedlocation\v2021_08_15\operations\_custom_locations_operations.py", line 282, in get     response = self._client.send(request, stream=False, **operation_config)                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\msrest/service_client.py", line 336, in send   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\msrest/pipeline/__init__.py", line 197, in run   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\msrest/pipeline/__init__.py", line 150, in send   File "D:\a\_work\1\s\build_scripts\windows\artifacts\cli\Lib\site-packages\msrest/pipeline/requests.py", line 65, in send AttributeError: 'CredentialAdaptor' object has no attribute 'signed_session' To check existing issues, please visit: https://github.com/Azure/azure-cli/issues]
```

# Issue Validation
Run `az version` and confirm the following:
1. "azure-cli" version is 2.70 or higher (or different from what was shipped with Azure Local)
2. "customlocation" version is 0.1.3

# Cause
There was a breaking change introduced in Azure CLI 2.70.0, released on 03/03/2025. [Release notes: Release Azure CLI 2.70.0 · Azure/azure-cli](https://github.com/Azure/azure-cli/releases/tag/azure-cli-2.70.0)

 As part of this release, the Track 1 SDK has been removed from Azure CLI, including the removal of the resource parameter from the get_login_credentials() fuction. More details on the PR that introduced this breaking change - https://github.com/Azure/azure-cli/pull/29631#discussion_r1947487038

# Mitigation Details

To mitigate we need to downgrade the Azure CLI version. To do this, we will remove the current Az CLI and install it back using the specific version supported for the release:

1.  **Confirm the version of Az CLI installed**:
- We can confirm the version installed on the machine using this command:
```Powershell
Get-WmiObject -Class Win32_Product | Where-Object { ( $_.Name -Like "*Azure Cli*" )  }
```
- Az CLI installed should be the 32-bits version, if the 64 bits version is present, we will need to remove it.

2. **Finding the packaged version of the Azure CLI in nugetstore**
- A copy of the msi for the valid Az CLI version should be available on each node in c:\nugetstore. Find the full path for the valid msi file and copy the path below:
```Powershell
$azcliPath = "c:\NugetStore\Microsoft.AzureStack.AzureCLI.<<version>>\content\azure-cli-<<version>>.msi"
# For example:
# $azcliPath = "c:\NugetStore\Microsoft.AzureStack.AzureCLI.2.67.0.1"
```
- During Update process, you might find multiple versions of this nuget. In case of multiple nugets for Azure CLI, pick the latest.
- Take note of the <\<version\>> for the nuget, since this is the supported release of the Az CLI for your version of Azure Local.

```Powershell
$azSupportedVersion = "<<version>>"
# For example:
# $azSupportedVersion = "2.67"
```

3. **Uninstall any Azure CLI with an unsupported version**
```Powershell
Get-WmiObject -Class Win32_Product | Where-Object { ( $_.Name -Like "*Azure Cli*" ) -and ( $_.Version -NotLike "$($azSupportedVersion)*" ) } | ForEach-Object { $_.Uninstall() }
```

4. **Uninstall Azure CLI 64-bits if present**
- If the 64-bits version of the Azure CLI is present, independent of the version, we will need to remove it since it is not currently supported in Azure Local.
```Powershell
$prod = Get-WmiObject -Class Win32_Product | Where-Object { ( $_.Name -Like "*Microsoft Azure CLI (64-bit)*" ) }
$prod.uninstall()
```

5. **Install Az CLI using the msi from Step 2**
```Powershell
$argStr = "/I $azcliPath /quiet"
Start-Process msiexec.exe -Wait -ArgumentList $argStr
```

6. **Add the Az CLI path if not already added**
- The following scriptblock will add the Az CLI path to the Environment Variables at Machine and Proces level if not already added:

```Powershell
$NewPath = "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\wbin" # Az CLI path
```
- Add to Environment Variable at Machine level:
```Powershell
$path = [Environment]::GetEnvironmentVariable('PATH', 'Machine')
if ( -not( $path -like "*$NewPath*") ) {
    $path += ";$NewPath;"
    [Environment]::SetEnvironmentVariable('PATH', $path, 'Machine')
    Write-Host "Path [$NewPath] set to Machine level environment Path variable" -Verbose
}
else {
    Write-Host "Path [$NewPath] already set at Machine Level" -Verbose
}
```

- Add to Environment Variable at Process level:
```Powershell
$path = [Environment]::GetEnvironmentVariable('PATH', 'Process')  
if ( -not( $path -like "*$NewPath*") ) {  
    $path += ";$NewPath;"  
    [Environment]::SetEnvironmentVariable('PATH', $path, 'Process')  
    Write-Host "Path [$NewPath]  set to Process level environment Path variable" -Verbose  
}  
else {  
    Write-Host "Path [$NewPath] already set at Process Level" -Verbose  
}  
```

- Confirm that the path has been successfully added at Machine and Process level:
```Powershell
$updatedMachineLevelPath = [Environment]::GetEnvironmentVariable('PATH', 'Machine')
Write-Host "Updated Machine Level Path = $updatedMachineLevelPath" -Verbose
  
$updatedProcessLevelPath = [Environment]::GetEnvironmentVariable('PATH', 'Process')  
Write-Host "Updated Process Level Path  = $updatedProcessLevelPath" -Verbose
```

7. **Set the Az CLI to not auto-update**

```Powershell
az config set auto-upgrade.all=no # Will disable auto-upgrade for extensions
az config set auto-upgrade.enable=no # Will disable auto-upgrade for Az CLI
az config set auto-upgrade.prompt=no # Will disable update prompt
```

8. **Validate installation of Az CLI**
- Run the following commands and confirm that the Az CLI isntalled is the expected version from the nugetstore, and that only the 32-bits version is installed.
```Powershell
az version 
Get-WmiObject -Class Win32_Product | Where-Object { ( $_.Name -Like "*Azure Cli*" )  }
```


### **Additional Notes**

-  These steps need to be run on every node in the cluster

### **References**
- [Release notes: Release Azure CLI 2.70.0 · Azure/azure-cli](https://github.com/Azure/azure-cli/releases/tag/azure-cli-2.70.0)
- [How to update the Azure CLI - Automatic Update](https://learn.microsoft.com/en-us/cli/azure/update-azure-cli#:~:text=installed%20extensions.-,Automatic%20Update,-By%20default%2C%20autoupgrade)