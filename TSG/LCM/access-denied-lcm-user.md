# Symptoms
During deployment, update, or add-node you may encounter 'Access Denied'. Some examples include:

During deployment you might see the following error message:

```
raised an exception: Connecting to remote server xxx failed with the following error message : Access is denied
```

During update download stage, you may see:

```
update package download failure with details similar to "Action plan GetCauDeviceInfo ID xxx failed with state: Failed".
```


# Known Causes
1. LCM (deployment user) credentials were not updated properly using the Set-AzureStackLCMUserPassword cmdlet.
2. LCM (deployment user) does not have sufficient permissions in Active Directory.

# Issue Validation
If you encounter an "Access Denied" error during deployment, update, or node addition, first verify that you are able to connect to a node with the LCM (deployment user) credentials. If you do not know your LCM username, please view the [Retrieving your LCM (user deployment) username section](#retrieving-your-lcm-deployment-user-username)

```PowerShell
$credential = Get-Credential # Input your LCM (deployment user) credentials
$targetHostname = "<target_hostname>" # This is your target host
Invoke-Command -computername $targetHostname -credential $credential -scriptblock {hostname}
```

If you receive an error stating the connection failed, you should verify that the credentials in the ECE store match the LCM user credentials in Active Directory. To do this, run the validation and mitigation script in the [Validating LCM (user deployment) credentials match the credentials ECE Store section](#validating-lcm-user-deployment-credentials-match-the-credentials-ece-store). 

If there is no error with connecting to the node using the credentials, verify the Active Directory permissions are set properly for the LCM user. Log into the domain controller node using the LCM user credentials and run the following to get the Access Control List (ACL) for the user on the OU. Please replace LCM username accordingly.

```Powershell
$ou = "OU=MyOU,DC=domain,DC=com"
$acl = Get-Acl "AD:$ou"

$lcmUsername = "<lcm username>"
$domain = (Get-WmiObject -Class Win32_ComputerSystem).Domain
$shortDomain = $domain.Split('.')[0]
$user = "$shortDomain\$lcmUsername"
$userAcl = $acl.Access | Where-Object {$_.IdentityReference -eq $user}

# Show permissions for that user
$userACL | Select-Object IdentityReference, ActiveDirectoryRights, AccessControlType
```

You should see output like the following:
```
IdentityReference           ActiveDirectoryRights AccessControlType
-----------------           --------------------- -----------------
<domain>\<LCM Username>              ReadProperty             Allow
<domain>\<LCM Username>                GenericAll             Allow
<domain>\<LCM Username>  CreateChild, DeleteChild             Allow
```

The user should have ReadProperty, GenericAll, CreateChild, and DeleteChild permissions on the OU. If you do not see all of these ActiveDirectoryRights listed for this user, please follow [instructions to prepare active directory](https://learn.microsoft.com/en-us/azure/azure-local/deploy/deployment-prep-active-directory?view=azloc-2503), including running the AsHciADArtifactsPreCreationTool listed in the wiki to ensure all permissions are set appropriately for the user.

# Scripts
### Validating LCM (user deployment) credentials match the credentials ECE Store

### Prerequisites
Ensure the certificate with subject name CN=RuntimeParameterEncryptionCert is not missing or expired. If so, please run [Start-SecretRotation](https://learn.microsoft.com/en-us/azure/azure-local/manage/manage-secrets-rotation?view=azloc-24113) to rotate certificates.

Please input your LCM user credentials when prompted.

```PowerShell
# Import necessary modules
Import-Module "ECEClient" 3>$null 4>$null
Import-Module "C:\Program Files\WindowsPowerShell\Modules\Microsoft.AS.Infra.Security.SecretRotation\Microsoft.AS.Infra.Security.ActionPlanExecution.psm1" -DisableNameChecking
Import-Module "C:\Program Files\WindowsPowerShell\Modules\Microsoft.AS.Infra.Security.SecretRotation\PasswordUtilities.psm1" -DisableNameChecking

# Validate that the username provided by customer is of the correct format. Username should be provided without domain and not contain any special characters.
ValidateIdentity -Username $credential.UserName

# Convert the SecureString password to an encrypted standard string
$encryptedPassword = $credential.GetNetworkCredential().Password | Protect-CmsMessage -To "CN=RuntimeParameterEncryptionCert"

# Validate credentials in ECE
$ValidateParams = @{
    TimeoutInSecs = 10 * 60
    RetryCount = "2"
    ExclusiveLock = $true
    RolePath = "SecretRotation"
    ActionType = "ValidateCredentials"
    ActionPlanInstanceId = [Guid]::NewGuid()
}
$ValidateParams['RuntimeParameters'] = @{
    UserName = $credential.GetNetworkCredential().UserName
    Password = $encryptedPassword
}

Write-AzsSecurityVerbose -Message "Validating credentials in ECE.`r`nStarting action plan with Instance ID: $($ValidateParams.ActionPlanInstanceId)" -Verbose
$ValidateActionPlanInstance = Start-ActionPlan @ValidateParams 3>$null 4>$null

if ($ValidateActionPlanInstance -eq $null)
{
    Write-AzsSecurityWarning -Message "There was an issue running the action plan. Please reach out to Microsoft support for help" -Verbose
}
elseif ($ValidateActionPlanInstance.Status -eq 'Failed')
{
    Write-AzsSecurityWarning -Message "Could not find matching credentials in ECE store." -Verbose
}
elseif ($ValidateActionPlanInstance.Status -eq 'Completed')
{
    Write-AzsSecurityVerbose -Message "Found matching credentials in ECE store." -Verbose
}
```

### Mitigation
Please input your LCM user credentials when prompted.

```PowerShell
# Prompt for credentials
$credential = Get-Credential

# Import the necessary module
Import-Module "C:\Program Files\WindowsPowerShell\Modules\Microsoft.AS.Infra.Security.SecretRotation\PasswordUtilities.psm1" -DisableNameChecking

# Validate that the username provided by customer is of the correct format. Username should be provided without domain and not contain any special characters.
ValidateIdentity -Username $credential.UserName

# Check the status of the ECE cluster group
$eceClusterGroup = Get-ClusterGroup | Where-Object { $_.Name -eq "Azure Stack HCI Orchestrator Service Cluster Group" }
if ($eceClusterGroup.State -ne "Online") {
    Write-AzsSecurityError -Message "ECE cluster group is not in an Online state. Cannot continue with password rotation." -ErrRecord $_
}

# Update ECE with the new password
Write-AzsSecurityVerbose -Message "Updating password in ECE" -Verbose

$ECEContainersToUpdate = @(
    "DomainAdmin",
    "DeploymentDomainAdmin",
    "SecondaryDomainAdmin",
    "TemporaryDomainAdmin",
    "BareMetalAdmin",
    "FabricAdmin",
    "SecondaryFabric",
    "CloudAdmin"
)

foreach ($containerName in $ECEContainersToUpdate) {
    Set-ECEServiceSecret -ContainerName $containerName -Credential $credential 3>$null 4>$null
}

Write-AzsSecurityVerbose -Message "Finished updating credentials in ECE." -Verbose
```

### Retrieving Your LCM (deployment user) Username
Run the following script on your HCI node to retrieve the LCM username:

```Powershell
$eceWinServiceVersion = Get-ChildItem "C:\Agents" -Directory |
    Where-Object { $_.Name -match 'Microsoft\.AzureStack\.Solution\.ECEWinService\.(\d+\.\d+\.\d+\.\d+)' } |
    ForEach-Object { $matches[1] }

[System.Reflection.Assembly]::LoadFile("C:\Agents\Microsoft.AzureStack.Solution.ECEWinService.$eceWinServiceVersion\content\ECEWinService\CloudEngine.dll") | Out-Null
[System.Reflection.Assembly]::LoadFile("c:\Agents\Microsoft.AzureStack.Solution.ECEWinService.$eceWinServiceVersion\content\ECEWinService\Microsoft.AzureStack.Orchestration.Common.Packaging.Contract.dll") | Out-Null
[System.Reflection.Assembly]::LoadFile("c:\Agents\Microsoft.AzureStack.Solution.ECEWinService.$eceWinServiceVersion\content\ECEWinService\Microsoft.AzureStack.Orchestration.Common.Packaging.dll") | Out-Null
[System.Reflection.Assembly]::LoadFile("c:\Agents\Microsoft.AzureStack.Solution.ECEWinService.$eceWinServiceVersion\content\ECEWinService\Microsoft.Diagnostics.Tracing.EventSource.dll") | Out-Null
[System.Reflection.Assembly]::LoadFile("c:\Agents\Microsoft.AzureStack.Solution.ECEWinService.$eceWinServiceVersion\content\ECEWinService\Microsoft.AzureStack.Solution.MetricTelemetry.dll") | Out-Null
 
Import-Module ECEClient 3>$null 4>$null
$eceClient = Create-ECEClusterServiceClient
$cloudDefinitionAsXmlString = (Get-CloudDefinition -EceClient $eceClient).CloudDefinitionAsXmlString
$cloudDefElements = [System.Xml.Linq.XElement]::Parse($cloudDefinitionAsXmlString)
 
$customerConfigurationObject = New-Object -TypeName 'CloudEngine.Configurations.CustomerConfiguration' -ArgumentList $cloudDefElements
$cloudRoleObject = [CloudEngine.Configurations.ConfigurationPathExtensions]::Find($customerConfigurationObject, 'Cloud')
[CloudEngine.Configurations.IInterface] $interface = $cloudRoleObject.Interface('Build')
$eceParams = $interface.GetInterfaceParameters()
 
$securityInfo = $ECEParams.Roles["Cloud"].PublicConfiguration.PublicInfo.SecurityInfo
$DAdmin = $securityInfo.DomainUsers.User | Where Role -eq "DomainAdmin"
Write-Output "Your LCM username is: $($DAdmin.Credential.Credential.UserName)"
```