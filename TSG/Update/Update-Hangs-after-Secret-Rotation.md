# Description
After Secret Rotation Update will stall and the HCI Orchestration service is crashing about every minute.

# Symptoms
During Update the progress will stall after secret rotation.

# Issue Validation
1. Update has not made progress after Secret Rotation
   * Update is usually trying to update the Lifecycle Agents which is the step after secret rotation.
2. ECE Service (HCI Orchestrator) is crashing.

You can use the following Cmdlet to check to see if the service is crashing.

```PowerShell
Function Get-OrchestrationLastCrash {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$false)]
        [ValidateNotNullOrEmpty()]
        [string]$Executable = "HciOrchestratorService.WinSvcHost.exe"
    )
    $startTime = (Get-Date).AddDays(-1)
    # We want to check all nodes
    # Get the node that Orchestration Service is running on
    $node = (Get-ClusterGroup *Orc*).OwnerNode

    # Just grab the exceptions from that node
    $error_messages = Invoke-Command -ComputerName $node -ScriptBlock {
        Get-WinEvent -FilterHashtable @{LogName = "Application"; StartTime = $($startTime); ID = 1000 } -ErrorAction SilentlyContinue
    } 

    $sorted_error_messages = $error_messages | Sort-Object TimeCreated -Descending
    $orchestation_errors = $sorted_error_messages | Where-Object { $_.Message -match $Executable }
    $last_error = $orchestation_errors | Select-Object -First 1 | Select-Object -Property TimeCreated, MachineName, Message
    $last_error | Format-List * -Force
}
```

If this is the same issue for this TSG you should see the following stack trace. We are throwing from MtlsCertificateException
```
TimeCreated : SomeDate
MachineName : TheMachineName
Message     : Microsoft.AzureStack.HciOrchestratorService.WinSvcHost.exe  
                       Framework Version: v4.0.30319  
                       Description: The application requested process termination through  
                       System.Environment.FailFast(string message).  
                       Message: Unhandled exception encountered; terminating. Exception:  
                       System.InvalidOperationException: Nullable object must have a value.
                           at System.ThrowHelper.ThrowInvalidOperationException(ExceptionResource resource)
                           at System.Nullable`1.get_Value()
                           at Microsoft.AzureStack.Common.Infrastructure.WebLayer.SecurityAuditLogger.MtlsCertificateException(Exception exception, String subjectName, String thumbprint)
```


# Cause
ECE Service uses common-infra to handle Auth for HTTP calls. When the Auth fails, we will attempt to log information about the request such as Correlation ID, Correlation Vector, Certificate Subject, Certificate Thumbprint. The issue is that the correlation ID and Correlation Vector can be null which causes a null reference exception.

We have seen that there are two callers that caused this to happen: Start-MonitoringActionplanInstanceToComplete and LCMController. 

We believe there could be other services or scripts that are using the old ECE service client certificate but have not seen any evidence.

## Start-MonitoringActionplanInstanceToComplete
A secondary issue is that Start-MonitoringActionplanInstanceToComplete loads the ECE service client certificate and then polls ECE service for an action plan. Once secret rotation happens the certificate is now invalid which then causes ECE service to crash.

## LCMController
This is an Arc extension and is not a part of the secret rotation process. They took a dependency on our certificates but since they are not part of secret rotation they will continue to use the old certificate after we have performed rotation.


# Mitigation Details

There are multiple mitigations, start from the first and stop when you see the issue go away.

## **(No Production Impact)** Terminate Start-MonitoringActionplanInstanceToComplete

As state above this cmdlet has an issue with handling secret rotation. If someone is using the cmdlet, just cancel and reload it.

## **(No Production Impact)** Restart LCM Extension on all Nodes

We have found that LCM Controller will continue to use a certificate after we have performed secret rotation. This will restart LCM Controller on the node in which it is running.

```powershell
Function Restart-LCM {
    [CmdletBinding()]
    param()

    $nodes = (Get-ClusterNode).Name

    Invoke-Command -ComputerName $nodes -ScriptBlock {
        Write-Host "[$env:COMPUTERNAME]: Getting LCMController service"
        $service = Get-Service 'LcmController' -ErrorAction SilentlyContinue
        
        if ($service -and ($service.Status -eq 'Running')) {
            Write-Host "[$env:COMPUTERNAME]: Restaring LCMController"

            $service | Stop-Service -Force
            $service | Start-Service
            
            Write-Host "[$env:COMPUTERNAME]: LCMController Restarted"
        }
        else {
            Write-Host "[$env:COMPUTERNAME]: LCMController is not running."
        }

    }
}
```

## **(No Production Impact)** Restart all local Azure Local Services and Move all Clustered HCI Services

This script moves all Azure Local Clustered Services to a new node and then restarts all local Azure Local services. The output is somewhat verbose, but it indicates when all services are being moved or restarted.
```powershell

<#
    Restarts all windows services, move the cluster grouped services to other nodes.
#>
Function Restart-AzureLocalServices {
    [CmdletBinding()]
    param()

    Function Write-Local($Message) {
        Write-Host "[$($env:COMPUTERNAME)] : $Message"
    }

    $nodes = (Get-ClusterNode).Name

    # This will move the cluster service to another node.
    Function Move-ClusterService($Name) {
        Write-Local "$Name"

        $group = Get-ClusterGroup $Name
        $sourceNode = $group.OwnerNode
        [string[]]$targetNodes = ($nodes | Where-Object { !$_.Equals($sourceNode) })
        $targetNode = $targetNodes[0]

        Write-Local "Moving Cluster Group '$Name' from '$($sourceNode)' to '$($targetNode)' "

        Move-ClusterGroup -Name $Name -Node $targetNode
    }

    Move-ClusterService "Azure Stack HCI Orchestrator Service Cluster Group"
    Move-ClusterService "Azure Stack HCI Update Service Cluster Group"
    Move-ClusterService "Azure Stack HCI Download Service Cluster Group"
    Move-ClusterService "Azure Stack HCI Health Service Cluster Group"

    # This restarts the Azure Local non clustered services
    Invoke-Command -ComputerName $nodes -ScriptBlock {
        $services = @('Azure Stack HCI Download Standalone Tool Agent', 'ECEAgent')
        foreach ($service in $services)
        {
            Write-Host "[$env:COMPUTERNAME] Restart $service"
            $service = Get-Service $service
            $service | Stop-Service -Force
            $service | Start-Service
        }
    }
}
```

## **(Impacts Production)** Restart each node one at a time
There could be some unknown service or script running which is using the old client certificate. This is the option of last resort and will ensure that the client certificate is not being used anymore. Only do this if you have no production workloads running.

# Validation
After each mitigation you can check to see if ECE has stopped crashing and update has progressed. If so the issue for this TSG was resolved.