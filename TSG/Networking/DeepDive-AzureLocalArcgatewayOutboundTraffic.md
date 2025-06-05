# Azure Local Arc Gateway Outbound Connectivity Deep Dive

## Overview

Azure Local with the Arc gateway makes it much easier and safer for your on-premises servers to connect to Azure. Instead of opening hundreds of firewall rules, you only need to allow fewer than 30 outbound connections. This means less work for IT admins, fewer security risks, and easier compliance with your organization’s policies.

**Key benefits:**

- **Better Security:** Fewer open connections mean less chance for attackers to get in.
- **Easier Setup:** You use your existing network and security tools, while the Arc gateway takes care of connecting to Azure.
- **Simpler Management:** With fewer endpoints to manage, it’s easier to keep track of what’s allowed and troubleshoot issues.

This guide explains how outbound connections work with the Arc gateway and Azure Local, including a diagram and what you need to configure.

## How Azure Local with Arc Gateway Connects to Azure

The diagram below shows how your on-premises servers (Azure Local instances) connect to Azure public services using the Arc gateway. Here’s what each part means:

![Azure Local with Arc gateway outbound connectivity](./images/AzureLocalPublicPathAllFlows.svg)

### Components Explained

- **Azure Local Instance(s):** These are your on-premises servers or clusters running Azure services locally.
- **Management Network:** The part of your network where the Azure Local instances live. Outbound connections to Azure start from here.
- **Arc Gateway:** This acts like a secure messenger. It collects all the traffic from your local servers and sends it safely to Azure. You only need to open outbound connections from the Arc gateway, not from every server.
- **Firewall/Proxy:** Your existing security layer. All outbound traffic from the Arc gateway passes through here. You control which connections are allowed.
- **Azure Public Endpoints:** The Azure services your local environment needs to reach (like Azure Resource Manager, Key Vault, Microsoft Graph, etc.). These are the only destinations you need to allow through your firewall or proxy.
- **Traffic Flows:** The arrows in the diagram show how data moves: from your local servers, through the Arc gateway, out your firewall/proxy, and up to Azure.
- **Labels and Annotations:** The diagram shows which protocols are used (usually HTTPS on port 443) and may highlight the specific URLs or IP ranges you need to allow.

### Why This Diagram Matters

- **Clear Network Requirements:** IT admins can quickly see which connections need to be open and how traffic should flow.
- **Easier Troubleshooting:** If something isn’t working, use the diagram to check if all the right paths are open.
- **Security Reviews:** Security teams can confirm that only the necessary connections are allowed, and that the Arc gateway is set up correctly.
- **Visual Reference:** The diagram makes the written instructions easier to understand and follow.

By following this guide and using the diagram, IT admins can confidently set up and manage outbound connectivity for Azure Local with the Arc gateway, keeping things secure and simple.
