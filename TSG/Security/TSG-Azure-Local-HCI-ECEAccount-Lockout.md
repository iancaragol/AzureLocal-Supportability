ECEAgent or "Azure Stack HCI Orchestrator Service" is unable to start because ECEAgentService or HCIOrchestrator local account is locked out.

# Symptoms
An error like the following in the System Event Log:
`The ECEAgent service was unable to log on as .\ECEAgentService with the currently configured password due to the following error: 
The referenced account is currently locked out and may not be logged on to.`

Or action plan log:
`Type 'SelfUpdate' of Role 'UpdateBootstrap' raised an exception: The referenced account is currently locked out and may not be logged on to. at Install-Nuget`

# Issue Validation
ECEAgent or "Azure Stack HCI Orchestrator Service" service is unable to start, ore remoting fails. To determine which account is locked, connect to the nodes and run:
```
Get-WmiObject -Query "SELECT * FROM Win32_UserAccount Where LocalAccount = true" -computerName localhost | Format-Table Domain, Name, lockOut, Disabled
```

# Cause
ECEAgent and "Azure Stack HCI Orchestrator Service" services run under local admin accounts that are configured for PKU2U security protocol. PKU2U security protocol is needed for remote node authentication, because those services run action plans on remote nodes. PKU2U security protocol requires a local user account.

Therefore, two local user accounts were introduced: ECEAgentService (to run ECEAgent) and HCIOrchestrator (to run "Azure Stack HCI Orchestrator Service"). Sometimes an error happens such that the user's password is new, but the service is still trying to log on with the old password. After a few unsuccessful login attempts, the user's account is locked out.

# Mitigation Details
   > :exclamation: **IMPORTANT**
   > These mitigation steps are only needed on node(s) where ECEAgentService or HCIOrchestrator user accounts are locked out. They are not needed on nodes where ECEAgent and "Azure Stack HCI Orchestrator Service" are not locked and running without issues.

Mitigation involves generating a new password and re-syncing the user account and the corresponding service to use the new password. Use an elevated PowerShell and enter an acceptably strong password after the first (Read-Host) command.

### To re-sync the password and unlock the ECEAgentService account:
Connect to the node(s) where ECEAgentService is locked out, and run the following commands:
```Powershell
$Password = Read-Host -prompt "Enter Password" -AsSecureString
$UserAccount = Get-LocalUser -Name "ECEAgentService"
$UserAccount | Set-LocalUser -Password $Password
net user "ECEAgentService" /active:yes

sc.exe config "ECEAgent" obj= ".\ECEAgentService" password= "<password from the Read-Host command>"
```
### To re-sync the password and unlock the HCIOrchestrator account.
On one node, run the following to stop the ECE Service cluster group:
```
Stop-ClusterGroup "Azure Stack HCI Orchestrator Service Cluster Group"
```
On ALL nodes, run the below commands. Note that the passwords can differ on each node.
```Powershell
$Password = Read-Host -prompt "Enter Password" -AsSecureString
$UserAccount = Get-LocalUser -Name "HCIOrchestrator"
$UserAccount | Set-LocalUser -Password $Password
net user "HCIOrchestrator" /active:yes

sc.exe config "Azure Stack HCI Orchestrator Service" obj= ".\HCIOrchestrator" password= "<password from the Read-Host command>"
```

On One node, run the following:
```
Start-ClusterGroup "Azure Stack HCI Orchestrator Service Cluster Group"
```
### Remediation Confirmation
Confirm that the accounts are unlocked
```
Get-WmiObject -Query "SELECT * FROM Win32_UserAccount Where LocalAccount = true" -computerName localhost | Format-Table Domain, Name, lockOut, Disabled
```
and that ECEAgent or Azure Stack HCI Orchestrator Service Cluster Group is now running by using the respective command
```
Get-ClusterGroup "Azure Stack HCI Orchestrator Service Cluster Group"

Get-Service "ECEAgent"
```