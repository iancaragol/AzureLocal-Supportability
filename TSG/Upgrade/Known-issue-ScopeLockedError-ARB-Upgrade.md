During Azure Local solution update, the overall upgrade operation will fail - the ARB cluster upgrade succeeds, but a cleanup setup during the ARB upgrade operation fails as ARB RP is unable to delete the dependent cluster extension used to trigger ARB cluster upgrade.

ARB RP deploys the Appliance Management Cluster Extension/KVA Management Operator (AMCE/KVAMO) to trigger the cluster upgrade. This means ARB expects to be able to create and delete this cluster extension as part of the upgrade operation.

# Symptoms

Error: [ERROR: (FailedToDeleteClusterExtension) DELETE https://management.azure.com/subscriptions/{subscription}/resourceGroups/{resource-group}/providers/microsoft.resourceconnector/appliances/{appliance}/providers/Microsoft.KubernetesConfiguration/extensions/kva-management-operator/providers/Microsoft.KubernetesConfiguration/extensions/kva-management-operator 
--------------------------------------------------------------------------------
RESPONSE 409: 409 Conflict ERROR CODE: ScopeLocked
--------------------------------------------------------------------------------

```json
{  
 "error":
  {     
   "code": "ScopeLocked",     
   "message": "The scope 
'/subscriptions/********************/resourceGroups/***********/providers/microsoft.resourceconnector/appliances/**********/providers/Microsoft.KubernetesConfiguration/extensions/kva-management-operator' 
   cannot perform delete operation because following scope(s) are locked: '/subscriptions/********************/resourceGroups/***********/providers/microsoft.resourceconnector/appliances 
  /**********'. 
   Please remove the lock and try again."   
 } 
}
```
# Cause
ASZ/Azure Stack HCI documentation recommends the customer to create a DELETE lock on the ARB resource here . This is the reason the customer runs into the error mentioned above during ARB upgrade operation.

**ARB RP by design deploys and deletes the cluster extension during ARB cluster upgrade operation. Delete locks prevents ARB RP from successfully performing upgrade operation. Therefore, delete lock recommendation in the HCI documentation is incorrect and documentation needs to be updated to remove this recommendation.**

The work item to get this recommendation removed from public documentation is tracked by 275231 . However, if a customer faces this issue, the following mitigation is recommended.

# Mitigation

 We recommend executing the following steps for mitigation.

1. Remove Lock from Resource bridge object.
* In the Azure portal, navigate to the resource group into which you deployed your Azure Stack HCI system.
* On the Overview > Resources tab, you should see an Arc Resource Bridge resource.
* Select and go to the resource. In the left pane, select Locks. To remove the lock for the Arc Resource Bridge, you must have the Azure Stack HCI Administrator role for the resource group.
* In the right pane, select Delete.

2. Delete the cluster extension

* Requirements
You need to be aware of the ARB resource's subscriptionID, resourceGroupName, and resourceName

```powershell
# Ensure k8s-extension CLI is installed
az extension add -n k8s-extension
    
# Set subscription id
az account set -s "<subscriptionID>"
    
# Delete cluster extension
```powershell
az k8s-extension delete -g "<resourceGroupName>" -t "appliances" --cluster-name "<resourceName>" --name "kva-management-operator"
```
3. Do a PUT call on the ARB resource

Requirements
* Path to config file

* For ASZ/HCI this will be ``C:\ClusterStorage\Infrastructure_1\Shares\SU1_Infrastructure_1\MocArb\WorkingDirectory\Appliance\hci-appliance.yaml``
* For VMware and SCVMM this will be wherever the customer chose to save the configuration. If lost, please rebuild the config using ``az arcappliance createconfig`` (documentation found [here](https://learn.microsoft.com/en-us/cli/azure/arcappliance/createconfig))
Path to kubeconfig file
* For ASZ/HCI this will be ``C:\ClusterStorage\Infrastructure_1\Shares\SU1_Infrastructure_1\MocArb\WorkingDirectory\Appliance\kubeconfig``
For VMware and SCVMM this will wherever the customer has saved the kubeconfig.
This can be retrieved if lost using az arcappliance get-credentials (documentation here )
Provider type (hci, vmware, scvmm)

```powershell
az arcappliance create "<provider>" --config-file "<path to appliance yaml>" --kubeconfig "<path to kubeconfig>"
```
Verify the resource is in status Running and Provisioning State Succeeded
Requirements
The expected version of ARB post-mitigation should remain the same as it was before the mitigation was applied. If the version doesn't match the expected version, please reach out to the ARB team.

```json
{
        "id": "/subscriptions/**********/resourceGroups/********/providers/Microsoft.ResourceConnector/appliances/****",
        "name": "**********",
        "location": "**********",
        "identity": {
            "type": "SystemAssigned",
            "principalId": "**********",
            "tenantId": "**********"
        },
        "type": "Microsoft.ResourceConnector/appliances",
        "properties": {
            "distro": "AKSEdge",
            "version": "<expected version>", <----- Important
            "status": "Running", <----- Important
            "provisioningState": "Succeeded" <---- Important
        }
    }
```
