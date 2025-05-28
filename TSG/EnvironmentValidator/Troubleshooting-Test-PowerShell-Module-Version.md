# Overview

Environment Validator runs a Validated Recipe validator prior to Update. The test detects issues with install components that have drifted from the validated recipe.

# Symptoms

During pre-update validation, the Validated Recipe tests fail on Test PowerShell Module version.

Inspecting the Pre-Update health check results (the example is PowerShell but these results are available in the console)
```
Get-SolutionUpdateEnvironment | Select-Object -ExpandProperty HealthCheckResult | Where-Object Status -ne 'Success'
```

Reveals the following failure:

```
Name               : AzStackHci_ValidatedRecipe_PowerShellModule_Version
DisplayName        : Test PowerShell Module Version
Tags               : {}
Title              : Test PowerShell Module Version
Status             : FAILURE
Severity           : CRITICAL
Description        : Validating that the PS modules installed on the host are the same versions defined in the
                     validated recipe.
Remediation        : https://learn.microsoft.com/en-us/azure/azure-local/update/update-troubleshooting-23h2
TargetResourceID   : NODE01
TargetResourceName : NODE01
TargetResourceType : ValidatedRecipe
Timestamp          : 5/27/2025 2:59:37 PM
AdditionalData     : {}
HealthCheckSource  : Manual\Standard\Medium\ValidatedRecipe\6f49f155
```

This result means the validation found PowerShell Modules it was not expecting but it is lacking the detail on which modules are wrong (this issue is being addressed). 


# Issue Validation

The validation can be run manually to verify and give us more information. Note: you can pass PsSession array to this cmdlet, the example just focusses on running locally.

```
$result = Invoke-AzStackHciValidatedRecipeValidation -PassThru -Include Test-PSModules
```

The output can be viewed to confirm a failure state:

```
PS C:\> $result

HealthCheckSource  : Manual\Standard\Medium\ValidatedRecipe\166243fa
Name               : AzStackHci_ValidatedRecipe_PowerShellModule_Version
DisplayName        : Test PowerShell Module Version
Tags               : {}
Title              : Test PowerShell Module Version
Status             : FAILURE
Severity           : CRITICAL
Description        : Validating that the PS modules installed on the host are the same versions defined in the
                     validated recipe.
Remediation        : https://learn.microsoft.com/en-us/azure/azure-local/update/update-troubleshooting-23h2
TargetResourceID   : NODE01
TargetResourceName : NODE01
TargetResourceType : ValidatedRecipe
Timestamp          : 5/28/2025 11:59:26 AM
AdditionalData     : {[Detail, 'PackageManagement':'1.4.8.1' found;  'PowershellGet':'2.2.5' found;  Checking version
                     of PS module 'Az.Accounts' on host 'NODE01'. Got '5.0.1' but expected less-than or equal to
                     '4.0.2' so it is invalid.;  'Az.Resources':'7.8.0' found;  'Az.ConnectedMachine':'1.1.1' found;
                     'Az.Storage':'8.1.0' found;  'Az.Attestation':'2.1.0' found;  'Az.AksArc':'0.1.3' found;
                     'TraceProvider':'1.0.28' found;  'DownloadSdk':'1.1.20' found;  'moc':'1.2.27' found;
                     'ArcHci':'1.2.68' found;  'Az.StackHCI':'2.5.0' found;  'SdnDiagnostics':'4.2504.1.2119' found],
                     [Status, FAILURE], [TimeStamp, 05/28/2025 11:59:26], [Resource, Validated Assembly Recipe]...}
```

Inspect the AdditionalData > detail property for a breakdown of what needs remediating:

```
PS C:\> ($result | ? Name -eq AzStackHci_ValidatedRecipe_PowerShellModule_Version).AdditionalData.Detail -split ';'
'PackageManagement':'1.4.8.1' found
  'PowershellGet':'2.2.5' found
  Checking version of PS module 'Az.Accounts' on host 'NODE01'. Got '5.0.1' but expected less-than or equal to '4.0.2' so it is invalid.
  'Az.Resources':'7.8.0' found
  'Az.ConnectedMachine':'1.1.1' found
  'Az.Storage':'8.1.0' found
  'Az.Attestation':'2.1.0' found
  'Az.AksArc':'0.1.3' found
  'TraceProvider':'1.0.28' found
  'DownloadSdk':'1.1.20' found
  'moc':'1.2.27' found
  'ArcHci':'1.2.68' found
  'Az.StackHCI':'2.5.0' found
  'SdnDiagnostics':'4.2504.1.2119' found
```

# Cause

The pertinent information in the result above is **Checking version of PS module 'Az.Accounts' on host 'NODE01'. Got '5.0.1' but expected less-than or equal to '4.0.2' so it is invalid.**

# Mitigation Details

On each node we should remove any versions of modules that the validator found to be wrong. 

```
$RequiredModuleName = 'Az.Accounts'
$RequiredModuleVersion = '4.0.2'

# Get all versions of the 
Get-InstalledModule -Name $RequiredModuleName -AllVersions

#Uninstall all but the required version
Get-InstalledModule -Name $RequiredModuleName -AllVersions | Where-Object { $_.Version -ne $RequiredModuleVersion } | ForEach-Object { Uninstall-Module -Name $RequiredModuleName -RequiredVersion $_.Version -Force }

# if the required version was not present in the first command's output, install it.
Install-Module -Name $RequiredModuleName -RequiredVersion $RequiredModuleVersion -Force

# Verify only the required version is installed
Get-InstalledModule -Name $RequiredModuleName -AllVersions

# Verify validator now passes
Invoke-AzStackHciValidatedRecipeValidation -PassThru -Include Test-PSModules | Format-List Name, Status
```
