# Update Network ATC Intent Overrides

<table border="1" cellpadding="6" cellspacing="0" style="border-collapse:collapse; margin-bottom:1em;">
  <tr>
    <th style="text-align:left; width: 180px;">Component</th>
    <td><strong>Network ATC</strong></td>
  </tr>
  <tr>
    <th style="text-align:left; width: 180px;">Topic</th>
    <td><strong>Update Intent Overrides</strong>: Instructions for updating intent overrides in Network ATC</td>
  </tr>
</table>

## Overview

In some cases you may need to update the Network ATC Intent overrieades.

## What and Why

### What This Guide Covers

This guide provides step-by-step instructions for updating the intent overrides in the Network ATC configuration.

### When to Use This Guide

Use this guide when you need to modify the intent overrides for a specific network configuration in Network ATC.

## Prerequisites

You should have a clear understanding of why the Network ATC Intents must be overridden.

## Table of Contents

  - [Overview](#overview)
  - [What and Why](#what-and-why)
  - [Prerequisites](#prerequisites)
  - [Updating Intent Overrides](#updating-intent-overrides)
  - [Verification](#verification)
  - [Troubleshooting](#troubleshooting)

## Updating QOS Intent Overrides

This section, we will cover the steps to update the intent overrides in the Network ATC configuration.

### Step 1: Review Current QOS Settings

Before making any changes, review the current QOS settings in your Network ATC configuration.

1. Open PowerShell as Administrator

2. Run the following command to view current Network ATC settings:
   ```powershell
   Get-NetIntentStatus -ClusterName <YourClusterName>
   ```

### Step 2: Create QOS Override Configuration

Prepare the QOS override configuration based on your requirements. Azure Local default QOS settings are:

- Storage (RDMA): CoS 3, 50% bandwidth
- Cluster Heartbeat: CoS 7, 1-2% bandwidth (2% for ≤10Gbps, 1% for >10Gbps)
- Default Traffic: CoS 0, remaining bandwidth

1. Create a JSON configuration file (e.g., `qos-override.json`) with the desired overrides:

```json
{
  "StorageIntents": [
    {
      "Name": "Storage",
      "AdapterPropertyOverrides": {
        "QoS": {
          "PriorityFlowControl": "Enabled",
          "Enhanced Transmission Selection": {
            "StorageBandwidthPercent": 50,
            "ClusterBandwidthPercent": 1
          },
          "ExplicitCongestionNotification": "Enabled",
          "ClassOfService": {
            "Storage": 3,
            "Cluster": 7,
            "Default": 0
          }
        }
      }
    }
  ]
}
```

2. Validate the JSON configuration:
   - Ensure all required fields are present
   - Verify bandwidth percentages total to 100% or less
   - Confirm CoS values are between 0-7

### Step 3: Apply QOS Overrides

Apply the configuration using Network ATC PowerShell cmdlets.

1. Import the override configuration:
   ```powershell
   $overrideConfig = Get-Content -Path "qos-override.json" | ConvertFrom-Json
   ```

2. Apply the overrides:
   ```powershell
   Set-NetIntentOverride -ClusterName <YourClusterName> -Override $overrideConfig
   ```

### Verification

1. Verify the override was applied successfully:
   ```powershell
   Get-NetIntentStatus -ClusterName <YourClusterName>
   ```
   - Check that QOS settings match your configuration
   - Confirm no errors in the status output

2. Verify network functionality:
   - Test storage performance
   - Monitor cluster heartbeat stability
   - Check for any packet loss or congestion issues

---
