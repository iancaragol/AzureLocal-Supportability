# Azure Local Arc Gateway Outbound Connectivity Deep Dive

## Overview

Azure Local with the Arc gateway simplifies and secures the connectivity of your on-premises servers to Azure. Instead of managing hundreds of firewall rules, you only need to allow fewer than 30 outbound connections. This significantly reduces administrative overhead, enhances security, and simplifies compliance with your organization's policies.

### Key Benefits:

- **Better Security:** Fewer open connections reduce the potential attack surface.
- **Easier Setup:** Leverage your existing network and security infrastructure while the Arc gateway manages Azure connectivity.
- **Simpler Management:** Fewer endpoints to manage means easier tracking and troubleshooting.

This guide explains how outbound connections work with the Arc gateway and Azure Local, including detailed diagrams and configuration requirements.

---

## Azure Local Components Required for Arc Gateway Connectivity

The following diagram introduces the core components involved in Azure Local connectivity using the Arc gateway:

- **Azure Local Instance:** Your on-premises Azure Local cluster.
- **Nodes:** Individual servers within your Azure Local instance.
- **Arc Proxy (on Arc connected machine agent):** A local proxy service running on each node, responsible for securely routing HTTPS traffic through the Arc gateway.
- **Arc Gateway Public Endpoint:** The Azure-hosted endpoint establishing a secure HTTPS tunnel between your local Arc proxy and Azure.
- **Firewall/Proxy:** Your organization's existing security infrastructure controlling outbound traffic.
- **Azure Public Endpoints:** Azure services (e.g., Azure Resource Manager, Key Vault, Microsoft Graph) required by your local environment.

Types of operating system (OS) network traffic based on how they should be routed when using Azure Local with the Arc gateway.

## Types of OS Network Traffic and Routing with Azure Local and Arc Gateway

When using Azure Local with the Arc gateway, operating system (OS) network traffic is categorized based on how it should be routed. Clearly distinguishing these categories helps administrators correctly configure network routing rules, ensuring secure, efficient, and compliant connectivity between on-premises infrastructure and Azure services.

### Traffic Categories:

1. **🟦 🔵  OS HTTP and HTTPS traffic that must bypass your proxy** 
   Specific HTTP and HTTPS connections that should not pass through your organization's standard proxy infrastructure or through Arc proxy. Instead, these connections directly reach their intended internal destinations, typically due to technical requirements or performance considerations.

2. **🟨 🟡 OS HTTP traffic that cannot use Arc proxy and must be sent to your enterprise proxy or firewall**  
   HTTP traffic incompatible with the Arc proxy. This traffic must instead be routed through your organization's existing enterprise proxy or firewall infrastructure, ensuring compliance with internal security policies.

3. **🟩 🟢 OS HTTPS traffic that always uses Arc proxy**  
   HTTPS traffic that must always be routed through the Arc proxy. This ensures secure, controlled, and consistent connectivity to Azure endpoints, leveraging the Arc gateway's built-in security and management capabilities.

4. **🟥 🔴 Third-party OS HTTPS traffic not permitted through Arc gateway**

All HTTPS traffic from the operating system initially goes to the Arc proxy. However, the Arc gateway only permits connections to Microsoft-managed endpoints. This means that HTTPS traffic destined for third-party services—such as OEM endpoints, hardware vendor update services, or other third-party agents installed on your servers—cannot pass through the Arc gateway. Instead, this traffic is redirected to your organization's enterprise proxy or firewall. To ensure these third-party services function correctly, you must explicitly configure your firewall or proxy to allow access to these external endpoints based on your organization's requirements.

5. **🔵 Arc Resource Bridge VM and AKS Clusters using Azure Local Instance cluster IP as proxy**

This structured approach simplifies network management, enhances security, and ensures compliance with organizational policies.

![Azure Local with Arc gateway outbound connectivity](./images/AzureLocalPublicPathFlowsFinal-1Node-ComponentsOnly.dark.svg)

## Outbound traffic types Flows

### 1. Azure Local Node OS Traffic Bypassing the Proxy

This diagram with the animated blue arrow illustrates traffic from Azure Local nodes that bypasses the customer proxy and the Arc proxy entirely. Typical scenarios include:

- Internal communications within your local intranet.
- Node-to-node communications within the Azure Local cluster.
- Traffic destined for internal management or monitoring systems.

This traffic is sent directly to internal endpoints without passing through your enterprise proxy or the Arc proxy, ensuring low latency and efficient internal communication. Typical examples of endpoints that should bypass proxies from your Azure Local nodes include internal Active Directory domains, internal subnets, and traffic between Azure Local nodes.

When defining your proxy bypass list during Azure Local deployment, ensure that all required subnets, domains, and individual nodes are explicitly added. For detailed guidance on building the proxy bypass list during Arc registration, refer to the following articles:

![Azure Local Node OS Traffic Bypassing Proxy](./images/AzureLocalPublicPathFlowsFinal-1Node-Step1-BypassFlows.dark.svg)

---

### 2. Azure Local Node OS HTTP Traffic via Enterprise Proxy or Firewall

This diagram with the animated yellow arrow shows how standard HTTP (non-HTTPS) traffic from Azure Local nodes is managed:

- If an enterprise proxy is configured, HTTP traffic routes through this proxy.
- If no enterprise proxy is configured, HTTP traffic is sent directly to your firewall, where your organization's security policies determine whether the traffic is allowed or blocked.

The primary reason for routing HTTP traffic through your enterprise proxy or firewall is that the Arc proxy and Arc gateway only support HTTPS traffic. Since HTTP traffic isn't supported by these components, you must explicitly define an alternative outbound path for HTTP connections. This ensures that HTTP traffic is properly managed according to your organization's existing security policies and infrastructure.

![Azure Local Node OS HTTP Traffic](./images/AzureLocalPublicPathFlowsFinal-1Node-Step2-HTTPFlows.dark.svg)

---

### 3. Azure Local Node OS HTTPS Traffic via Arc Proxy

This diagram explains how HTTPS traffic from Azure Local nodes is securely routed:

- Green arrow represent HTTPS traffic destined for allowed Azure endpoints routes through the Arc proxy via the HTTPS tunnel between Arc proxy and Arc gateway
- Pink Arrow represents Traffic not allowed by the Arc gateway (non-approved endpoints) is redirected from Arc proxy to your firewall/proxy for further inspection or blocking. For example,Azure Local OEM endpoints for SBE uses third party endpoints not allowed by Arc gateway. This means you will need to explicity allow these third party HTTPS endpoints in your proxy and or firewall. Other example of traffic 

This ensures secure, controlled, and compliant outbound HTTPS connectivity.

![Azure Local Node OS HTTPS Traffic](./images/AzureLocalPublicPathFlowsFinal-1Node-Step3-HTTPSFlows.dark.svg)

---

### 4. Azure Resource Bridge Appliance VM HTTPS Traffic via Cluster IP Proxy

This diagram with the animated light blue arrow illustrates HTTPS traffic handling for the Azure Resource Bridge (ARB) appliance VM:

- ARB appliance VM uses the Azure Local Cluster IP as proxy and sends HTTPS traffic to the Arc proxy running on the node.
- The Arc proxy securely routes allowed traffic through the Arc gateway's HTTPS tunnel from the node to Azure.
- Non-allowed traffic is redirected to your firewall/proxy for security enforcement.

This ensures ARB appliance VM traffic is securely managed and compliant with your organization's policies.

![ARB Appliance VM HTTPS Traffic](./images/AzureLocalPublicPathFlowsFinal-1Node-Step4-ARBFlows.dark.svg)

---

### 5. AKS Clusters HTTPS Traffic via Cluster IP Proxy

This diagram shows HTTPS traffic handling for Azure Kubernetes Service (AKS) clusters within Azure Local:

- AKS clusters route HTTPS traffic through the Cluster IP proxy.
- The Cluster IP proxy securely forwards allowed traffic through the Arc gateway's HTTPS tunnel to Azure endpoints.
- Traffic not permitted by the Arc gateway is sent to your firewall/proxy for further security checks.

This ensures AKS clusters maintain secure and compliant outbound connectivity.

![AKS Clusters HTTPS Traffic](./images/AzureLocalPublicPathFlowsFinal-1Node-Step5-AKSFlows.dark.svg)

---

### 6. Azure Local VMs HTTPS Traffic via Dedicated Arc Proxy

This diagram explains HTTPS traffic handling for Azure Local virtual machines (VMs):

- Each Azure Local VM uses its own dedicated Arc proxy to route HTTPS traffic.
- Allowed HTTPS traffic is securely tunneled through the Arc gateway to Azure public endpoints.
- Non-allowed traffic is redirected to your firewall/proxy for security enforcement.
- Azure Local VM traffic that needs to bypass Arc proxy and or customer proxy will be send directly to the internal endpoint. 

This ensures Azure Local VMs have secure, controlled, and compliant outbound connectivity.

![Azure Local VMs HTTPS Traffic](./images/AzureLocalPublicPathFlowsFinal-1Node-Step6-VMFlows.dark.svg)

---

## Summary of the Overall Connectivity Model

- **Azure Local Nodes HTTP traffic** 🟨 (highlighted in yellow arrows in diagrams) is redirected to your organization's firewall/proxy for inspection and enforcement.
- **Azure Local Nodes Allowed HTTPS traffic** 🟩 (highlighted in green arrows in diagrams) is securely tunneled through the Arc gateway, significantly reducing firewall rules required (fewer than 30 endpoints).
- **Azure Local Nodes Non-allowed HTTPS traffic** (highlighted in pink in diagrams) is redirected to your organization's firewall/proxy for inspection and enforcement.
- **Azure Local Nodes Internal traffic** 🟦 (highlighted in dark blue arrows in diagrams) bypasses proxies entirely, ensuring efficient local communication.
- **Azure Resource Bridge VM traffic** (highlighted in light blue arrows in diagrams) is securely tunneled through the Arc gateway, significantly reducing firewall rules required (fewer than 30 endpoints).
This structured approach simplifies network management, enhances security, and ensures compliance with organizational policies.