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

The first type of traffic represented by the <span style="color:blue;"><strong>the blue box, OS HTTP and HTTPS traffic that must bypass your proxy,</strong></span> refers to specific HTTP and HTTPS connections that should not pass through your organization's standard proxy infrastructure. Instead, these connections must directly reach their intended destinations, typically due to technical requirements or performance considerations.

The second type of traffic represented by the yellow box, **"OS HTTP traffic that cannot use Arc proxy and must be sent to your enterprise proxy or/and firewall,"** describes HTTP traffic that is incompatible with the Arc proxy. This traffic must instead be routed through your organization's existing enterprise proxy or firewall infrastructure. This ensures compliance with internal security policies and maintains proper network management.

The third type of traffic represented by the green box, **"OS HTTPS traffic that always uses Arc proxy,"** identifies HTTPS traffic that must always be routed through the Arc proxy. This ensures secure, controlled, and consistent connectivity to Azure endpoints, leveraging the Arc gateway's built-in security and management capabilities.

Clearly distinguishing these traffic categories helps administrators correctly configure network routing rules, ensuring secure, efficient, and compliant connectivity between on-premises infrastructure and Azure services.

![Azure Local with Arc gateway outbound connectivity](./images/AzureLocalPublicPathFlowsFinal-1Node-ComponentsOnly.drawio.svg)

---

## Traffic Flow Scenarios

### 1. Azure Local Node OS Traffic Bypassing the Proxy

This diagram illustrates traffic from Azure Local nodes that bypasses the Arc proxy entirely. Typical scenarios include:

- Internal communications within your local intranet.
- Node-to-node communications within the Azure Local cluster.
- Traffic destined for internal management or monitoring systems.

This traffic is sent directly to internal endpoints without passing through the Arc gateway or external proxies, ensuring low latency and efficient internal communication.

![Azure Local Node OS Traffic Bypassing Proxy](./images/AzureLocalPublicPathFlowsFinal-1Node-Step1-BypassFlows.drawio.svg)

---

### 2. Azure Local Node OS HTTP Traffic via Enterprise Proxy or Firewall

This diagram shows how standard HTTP (non-HTTPS) traffic from Azure Local nodes is managed:

- If an enterprise proxy is configured, HTTP traffic routes through this proxy.
- If no enterprise proxy is configured, HTTP traffic is sent directly to your firewall, where your organization's security policies determine whether the traffic is allowed or blocked.

This ensures standard HTTP traffic aligns with your existing security infrastructure.

![Azure Local Node OS HTTP Traffic](./images/AzureLocalPublicPathFlowsFinal-1Node-Step2-HTTPFlows.drawio.svg)

---

### 3. Azure Local Node OS HTTPS Traffic via Arc Proxy

This diagram explains how HTTPS traffic from Azure Local nodes is securely routed:

- HTTPS traffic destined for allowed Azure endpoints routes through the Arc proxy running on each node.
- The Arc proxy establishes a secure HTTPS tunnel to the Arc gateway public endpoint hosted in Azure.
- Traffic not allowed by the Arc proxy (non-approved endpoints) is redirected to your firewall/proxy for further inspection or blocking.

This ensures secure, controlled, and compliant outbound HTTPS connectivity.

![Azure Local Node OS HTTPS Traffic](./images/AzureLocalPublicPathFlowsFinal-1Node-Step3-HTTPSFlows.drawio.svg)

---

### 4. Azure Resource Bridge Appliance VM HTTPS Traffic via Cluster IP Proxy

This diagram illustrates HTTPS traffic handling for the Azure Resource Bridge (ARB) appliance VM:

- ARB appliance VM sends HTTPS traffic through a Cluster IP proxy.
- The Cluster IP proxy securely routes allowed traffic through the Arc gateway's HTTPS tunnel to Azure.
- Non-allowed traffic is redirected to your firewall/proxy for security enforcement.

This ensures ARB appliance VM traffic is securely managed and compliant with your organization's policies.

![ARB Appliance VM HTTPS Traffic](./images/AzureLocalPublicPathFlowsFinal-1Node-Step4-ARBFlows.drawio.svg)

---

### 5. AKS Clusters HTTPS Traffic via Cluster IP Proxy

This diagram shows HTTPS traffic handling for Azure Kubernetes Service (AKS) clusters within Azure Local:

- AKS clusters route HTTPS traffic through the Cluster IP proxy.
- The Cluster IP proxy securely forwards allowed traffic through the Arc gateway's HTTPS tunnel to Azure endpoints.
- Traffic not permitted by the Arc gateway is sent to your firewall/proxy for further security checks.

This ensures AKS clusters maintain secure and compliant outbound connectivity.

![AKS Clusters HTTPS Traffic](./images/AzureLocalPublicPathFlowsFinal-1Node-Step5-AKSFlows.drawio.svg)

---

### 6. Azure Local VMs HTTPS Traffic via Dedicated Arc Proxy

This diagram explains HTTPS traffic handling for Azure Local virtual machines (VMs):

- Each Azure Local VM uses its own dedicated Arc proxy to route HTTPS traffic.
- Allowed HTTPS traffic is securely tunneled through the Arc gateway to Azure public endpoints.
- Non-allowed traffic is redirected to your firewall/proxy for security enforcement.

This ensures Azure Local VMs have secure, controlled, and compliant outbound connectivity.

![Azure Local VMs HTTPS Traffic](./images/AzureLocalPublicPathFlowsFinal-1Node-Step6-VMFlows.drawio.svg)

---

## Summary of the Overall Connectivity Model

- **Allowed HTTPS traffic** is securely tunneled through the Arc gateway, significantly reducing firewall rules required (fewer than 30 endpoints).
- **Non-allowed traffic** (highlighted in pink in diagrams) is redirected to your organization's firewall/proxy for inspection and enforcement.
- **Internal traffic** bypasses proxies entirely, ensuring efficient local communication.

This structured approach simplifies network management, enhances security, and ensures compliance with organizational policies.