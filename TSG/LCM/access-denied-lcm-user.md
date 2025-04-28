# Symptoms
During deployment, update, or add-node you may encounter 'Access Denied'. Some examples include:

During deployment you might see the following error message(s)

```
Exception: Connecting to remote server <hostname/IP> failed with the following error message: Access is denied
```

```
Exception : Type 'ValidateConnectivity' of Role 'EnvironmentValidator' raised an exception: Unable to create a valid session to <ip>: [<ip>] 
Connecting to remote server <ip> failed with the following error message : Access is denied. 
```

During update download stage, you may see:

```
update package download failure with details similar to "Action plan GetCauDeviceInfo ID xxx failed with state: Failed".
```

# Known Causes
1. The LCM (deployment user) credentials were not updated properly using the [Set-AzureStackLCMUserPassword](https://learn.microsoft.com/en-us/azure/azure-local/manage/manage-secrets-rotation?view=azloc-24112#run-set-azurestacklcmuserpassword-cmdlet) cmdlet. This cmdlet is responsible for updating the password in Active Directory as well as updating it in the ECE Store. Both of these should be in sync, otherwise there will be access denied issues.
2. The LCM user does not have sufficient permissions on the Organization Unit (OU) in Active Directory (AD).
3. The cluster node's NTLM policy, configured via Group Policy, may be blocking remote operations such as Invoke-Command. This can occur if NTLM is restricted either at the OU level or on the individual nodes via applied Group Policy Objects (GPOs).
4. The WinRM trusted hosts configuration is set up incorrectly.

# Issue Validation

Before running the following steps, please log into a node with the LCM user credential to ensure the credential works. If you do not know your LCM username, please view the [Retrieving your LCM (user deployment) username section](#retrieving-your-lcm-deployment-user-username). If you do not know your LCM user credentials, then you will need to recreate the LCM user and follow this document to ensure the LCM user is set up properly.

### Step 1: Check if LCM user is in the Local Administrators Group on all cluster nodes
Run the script below, replacing the `lcmUser` variable with your LCM (deployment) user on all cluster nodes. 

```Powershell
# Set your LCM user and the list of groups that contain the LCM user
$lcmUser = 'DOMAIN\LCMUser'
$groupsContainingLCMUser = @(
    'DOMAIN\Group1',
    'DOMAIN\Group2',
    'DOMAIN\Group3'
)

Write-Output "Checking if any of the specified groups are in the local Administrators group..."

try {
    $adminGroupMembers = Get-LocalGroupMember -Group 'Administrators'
} catch {
    Write-Error "Failed to retrieve Administrators group members. Error: $_"
    return
}

$found = $false

foreach ($group in $groupsContainingLCMUser) {
    if ($adminGroupMembers.Name -contains $group) {
        Write-Output "Group '$group' is in the Administrators group."
        $found = $true
        break
    }
}

if ($found) {
    Write-Output "$lcmUser has administrative rights via one of the specified groups."
} else {
    Write-Output "$lcmUser does NOT have administrative rights via the specified groups."
}
```

If the script shows that the `lcmUser` is not in the local Administrators group, then run the mitigation below:

```Powershell
# Replace 'DOMAIN\LCMUser' with your actual LCM username
$lcmUser = 'DOMAIN\LCMUser'

Write-Output "Attempting to add $lcmUser to the local Administrators group..."
try {
    Add-LocalGroupMember -Group 'Administrators' -Member $lcmUser
    Write-Output "$lcmUser was successfully added to the local Administrators group."
} catch {
    Write-Error "Failed to add $lcmUser to the Administrators group. Error: $_"
}
```

### Step 2: Check the LCM user credentials work on all nodes
Verify the LCM user credentials work with all iterations of Invoke-Command with and without CredSSP and using the hostname or the IP as the target:

```PowerShell
$credential = Get-Credential # Input your LCM (deployment user) credentials

$targetHostname = "<target_hostname>" # Replace with your target host
$targetIp = "<target_ip>" # Replace with your target IP address

Invoke-Command -computername $targetHostname -credential $credential -scriptblock {hostname}
Invoke-Command -ComputerName $targetHostname -Credential $credential -Authentication Credssp -ScriptBlock {whoami}

Invoke-Command -computername $targetIp -credential $credential -scriptblock {hostname}
Invoke-Command -ComputerName $targetIp -Credential $credential -Authentication Credssp -ScriptBlock {whoami}
```

If there are no issues running the previous script, then proceed to **Step 3**, otherwise:

1. If no Invoke-Command works (with or without CredSSP), double check you have entered the right LCM user credentials. If you are certain they are correct, then check the [WinRM Trusted Hosts configuration](#winrm-trusted-hosts-configuration) is set up properly to include all hostnames and IPs of all nodes in your cluster.
2. If Invoke-Command without CredSSP works, but CredSSP does not work, try [resetting the CredSSP registry keys](#reset-credssp-registry-keys).
3. If Invoke-Command with hostname works, but using the IP does not work, ensure the [WinRM Trusted Hosts configuration](#winrm-trusted-hosts-configuration) is set up properly to include all IPs of all nodes in your cluster.
 
### Step 3: Verify the LCM credential in AD matches ECE Store
If Invoke-Command works, then validate the credential exists in ECE store by running the script in the [Validating LCM (user deployment) credentials match the ECE Store](#validating-lcm-user-deployment-credentials-match-the-ece-store) section. If this script returns that they do not match, run the mitigation script from the same section.

### Step 4: Check LCM user permissions on the OU
Verify the Active Directory permissions are set properly for the LCM user. Log into the domain controller node using the LCM user credentials and run the following to get the Access Control List (ACL) for the user on the OU. Replace the LCM username in the script with your LCM username and the sample OU with your environment OU.

```Powershell
# Define the target OU (update to match your environment)
$ou = "OU=MyOU,DC=domain,DC=com"

# Get the ACL of the OU
$acl = Get-Acl "AD:$ou"

# Define the LCM username (without domain prefix)
$lcmUsername = "<lcm_username>"

# Get the domain name in short form
$domain = (Get-WmiObject -Class Win32_ComputerSystem).Domain
$shortDomain = $domain.Split('.')[0]
$user = "$shortDomain\$lcmUsername"

# Get all groups the LCM user is a member of
$userGroups = Get-ADUser $lcmUsername -Properties MemberOf |
    Select-Object -ExpandProperty MemberOf |
    ForEach-Object {
        ($_ -split ',')[0] -replace '^CN=', "$shortDomain\"
    }

# Combine user identity and groups
$identityRefs = @($user) + $userGroups

# Filter ACL for entries that match the user or any of their groups
$userAcl = $acl.Access | Where-Object { $identityRefs -contains $_.IdentityReference.Value }

# Display results
$userAcl | Select-Object IdentityReference, ActiveDirectoryRights, AccessControlType
```

You should see output like the following:
```
IdentityReference           ActiveDirectoryRights AccessControlType
-----------------           --------------------- -----------------
<domain>\<LCM Username>              ReadProperty             Allow
<domain>\<LCM Username>                GenericAll             Allow
<domain>\<LCM Username>  CreateChild, DeleteChild             Allow
```

The user should have the following permissions on the OU:
* ReadProperty 
* GenericAll 
* CreateChild 
* DeleteChild 
 
If you do not see all of these ActiveDirectoryRights listed for this user, please follow [instructions to prepare active directory](https://learn.microsoft.com/en-us/azure/azure-local/deploy/deployment-prep-active-directory?view=azloc-2503), including running the `AsHciADArtifactsPreCreationTool` listed in the wiki to ensure all permissions are set appropriately for the user. Ensure these same permissions are applied to all OU objects and their descendents.

### WinRM Trusted Hosts Configuration

```Powershell
Get-Item WSMan:\localhost\Client\TrustedHosts
```

This file should contain the ips or hostnames of each of the nodes in your cluster. To add a host, use this command:

```Powershell
Set-Item WSMan:\localhost\Client\TrustedHosts -Value <ip address>
```

# Scripts
### Validating LCM (User Deployment) Credentials Match the ECE Store

### Prerequisites
Ensure the certificate with subject name CN=RuntimeParameterEncryptionCert, or CN=DscEncryptionCert is not missing or expired. If so, please run [Start-SecretRotation](https://learn.microsoft.com/en-us/azure/azure-local/manage/manage-secrets-rotation?view=azloc-24113) to rotate certificates.

Please input your LCM user credentials when prompted. **DO NOT include the domain as part of the username in the credential**.

```PowerShell
$credential = Get-Credential

# Import necessary modules
Import-Module "ECEClient" 3>$null 4>$null
Import-Module "C:\Program Files\WindowsPowerShell\Modules\Microsoft.AS.Infra.Security.SecretRotation\Microsoft.AS.Infra.Security.ActionPlanExecution.psm1" -DisableNameChecking
Import-Module "C:\Program Files\WindowsPowerShell\Modules\Microsoft.AS.Infra.Security.SecretRotation\PasswordUtilities.psm1" -DisableNameChecking

# Validate that the username provided is of the correct format. Username should be provided without domain and not contain any special characters.
if ($credential.UserName -match '^[^\\]+(?=\\)|(?<=@).+$') {
    throw "Please provide user Identity without domain."
}

# Create ECE client and get the stamp version
$eceClient = Create-ECEClusterServiceClient
$stampVersion = $eceClient.GetStampVersion().GetAwaiter().GetResult()

# Define certificate path
$certPath = "Cert:\LocalMachine\My"  # Change this path if the cert is located elsewhere

# Check if RuntimeParameterEncryptionCert exists
$certName = "RuntimeParameterEncryptionCert"
$certificate = Get-ChildItem -Path $certPath | Where-Object { $_.Subject -like "*$certName*" }

# If RuntimeParameterEncryptionCert is not found, check for DscEncryptionCert
if (-not $certificate) {
    Write-AzsSecurityVerbose -Message "RuntimeParameterEncryptionCert not found. Checking for DscEncryptionCert." -Verbose
    $certName = "DscEncryptionCert"
    $certificate = Get-ChildItem -Path $certPath | Where-Object { $_.Subject -like "*$certName*" }

    # If DscEncryptionCert is also not found, throw an error
    if (-not $certificate) {
        throw "Neither RuntimeParameterEncryptionCert nor DscEncryptionCert found in the certificate store."
    }
} else {
    Write-AzsSecurityVerbose -Message "Found RuntimeParameterEncryptionCert." -Verbose
}

# Convert the SecureString password to an encrypted standard string using the selected certificate
$encryptedPassword = $credential.GetNetworkCredential().Password | Protect-CmsMessage -To "CN=$certName"

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

if ($ValidateActionPlanInstance -eq $null) {
    Write-AzsSecurityWarning -Message "There was an issue running the action plan. Please reach out to Microsoft support for help" -Verbose
}
elseif ($ValidateActionPlanInstance.Status -eq 'Failed') {
    Write-AzsSecurityWarning -Message "Could not find matching credentials in ECE store." -Verbose
}
elseif ($ValidateActionPlanInstance.Status -eq 'Completed') {
    Write-AzsSecurityVerbose -Message "Found matching credentials in ECE store." -Verbose
}
```

### Mitigation
Please input your LCM user credentials when prompted. **DO NOT include the domain as part of the username in the credential**.

```PowerShell
# Prompt for credentials
$credential = Get-Credential

# Import the necessary module
Import-Module "C:\Program Files\WindowsPowerShell\Modules\Microsoft.AS.Infra.Security.SecretRotation\PasswordUtilities.psm1" -DisableNameChecking

# Validate that the username provided is of the correct format. Username should be provided without domain and not contain any special characters.
if($credential.UserName -match '^[^\\]+(?=\\)|(?<=@).+$')
{
    throw "Please provide user Identity without domain."
}

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
#Retrieve ECEWinService NuGet Version
$eceWinServiceVersion = Get-ChildItem "C:\Agents" -Directory |
    Where-Object { $_.Name -match 'Microsoft\.AzureStack\.Solution\.ECEWinService\.(\d+\.\d+\.\d+\.\d+)' } |
    ForEach-Object { $matches[1] }

#Load Assemblies
[System.Reflection.Assembly]::LoadFile("C:\Agents\Microsoft.AzureStack.Solution.ECEWinService.$eceWinServiceVersion\content\ECEWinService\CloudEngine.dll") | Out-Null
[System.Reflection.Assembly]::LoadFile("c:\Agents\Microsoft.AzureStack.Solution.ECEWinService.$eceWinServiceVersion\content\ECEWinService\Microsoft.AzureStack.Orchestration.Common.Packaging.Contract.dll") | Out-Null
[System.Reflection.Assembly]::LoadFile("c:\Agents\Microsoft.AzureStack.Solution.ECEWinService.$eceWinServiceVersion\content\ECEWinService\Microsoft.AzureStack.Orchestration.Common.Packaging.dll") | Out-Null
[System.Reflection.Assembly]::LoadFile("c:\Agents\Microsoft.AzureStack.Solution.ECEWinService.$eceWinServiceVersion\content\ECEWinService\Microsoft.Diagnostics.Tracing.EventSource.dll") | Out-Null
[System.Reflection.Assembly]::LoadFile("c:\Agents\Microsoft.AzureStack.Solution.ECEWinService.$eceWinServiceVersion\content\ECEWinService\Microsoft.AzureStack.Solution.MetricTelemetry.dll") | Out-Null

#Retrieve LCM User Username
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

### Reset CredSSP Registry Keys
```Powershell
# Set properties of CredentialsDelegation key
Set-ItemProperty -Path HKLM:\SOFTWARE\Policies\Microsoft\Windows\CredentialsDelegation -Name AllowFreshCredentials -Value 1 -Type DWORD -Force
Set-ItemProperty -Path HKLM:\SOFTWARE\Policies\Microsoft\Windows\CredentialsDelegation -Name AllowFreshCredentialsWhenNTLMOnly -Value 1 -Type DWORD -Force
Set-ItemProperty -Path HKLM:\SOFTWARE\Policies\Microsoft\Windows\CredentialsDelegation -Name ConcatenateDefaults_AllowFresh -Value 1 -Type DWORD -Force
Set-ItemProperty -Path HKLM:\SOFTWARE\Policies\Microsoft\Windows\CredentialsDelegation -Name ConcatenateDefaults_AllowFreshNTLMOnly -Value 1 -Type DWORD -Force

# Create CredentialsDelegation sub-keys, if needed
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\CredentialsDelegation\AllowFreshCredentials" -ItemType Directory -Force
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\CredentialsDelegation\AllowFreshCredentialsWhenNTLMOnly" -ItemType Directory -Force
# Set properties of CredentialsDelegation sub-keys
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\CredentialsDelegation\AllowFreshCredentials" -Name "1" -Value "wsman/*" -Type String -Force
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\CredentialsDelegation\AllowFreshCredentialsWhenNTLMOnly" -Name "1" -Value "wsman/*" -Type String -Force
```