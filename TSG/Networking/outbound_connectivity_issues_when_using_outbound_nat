# Symptoms
You have deployed Virtual Machines into a Virtual Network and have created an outbound NAT pool to provide native outbound access to the internet. 
In some situations, you may notice that traffic is not working properly when making outbound calls.

# Issue Validation
To confirm the scenario that you are encountering is the issue documented in this article, you can follow the validation steps below:
1. You have workloads deployed on a Virtual Network.
2. You have workloads added to an backend pool that have outbound NAT rules configured.
3. One of the following scenarios is applicable:
    - Scenario 1: You have deployed a Virtual Network with a local DNS server deployed that has external DNS Forwarders configured to point to your datacenter or public DNS endpoint. VMs deployed to the Virtual Network that 
leverage your DNS server. There is no Gateway Connection that traffic is flowing through, or the DNS endpoint is not included in the routes defined within the Gateway Connection.
    - Scenario 2: You have deployed VMs into the Virtual Network that are using a public DNS endpoint. There is no Gateway Connection that traffic is flowing through, or the DNS endpoint is not included in the routes defined within the Gateway Connection.

# Cause
In this situation, you may not have the proper protocols defined for the outbound NAT rule. If you have a rule configured for TCP, any UDP related traffic will not be NATed properly, resulting in packet being dropped. 

> NOTE: There is a known issue with NetworkController that results in only the first Outbound NAT rule working. Any additional Outbound NAT rules configured will not take effect.

# Mitigation Details
Ensure that your Outbound NAT rule has `Protocol = 'All'` defined. You must configure this manually using powershell. 

Install or update SdnDiagnostics module. Refer to [Install the SDN diagnostics PowerShell module on the client computer](https://learn.microsoft.com/en-us/azure/azure-local/manage/sdn-log-collection#install-the-sdn-diagnostics-powershell-module-on-the-client-computer) for instructions.
```powershell
$ncUri = 'https://nc.contoso.com'
$loadBalancer = 'loadbalancer1'
$outboundNatRule = 'outboundnatrule1'

$object = Get-SdnResource -NcUri $ncUri -ResourceRef "/loadBalancers/$loadBalancer/outboundNatRules/$outboundNatRule"
$object.properties.protocol = "All"

Set-SdnResource -NcUri $ncUri -ResourceRef $object.resourceRef -Object $object
```

> NOTE: WAC does not currently expose the ability to configure `All` for protocol and only allows you to define `TCP` or `UDP`. 



