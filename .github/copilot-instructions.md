# Project Overview

This is a public repository for all of Azure Local Troubleshooting guides (TSGs), known issues and reporting feedback - this repo is intended to provide a central location for community driven supportability content. This is the material that is referenced by Customer Support Services when a ticket is created, by Azure Local engineering responding to an incident, and by users when self discovering resolutions to active system issues.

## Folder Structure

- `/TSG/<component>`: Contains folders for each major component area

### Components

- AKS: Azure Kubernetes Service on Azure Local
- ArcRegistration
- ArcVMs
- AVD: Azure Virtual Desktop
- Cluster Registration
- Deployment: Azure Local Deployment
- Environment Validator: Azure Local Environment Validator
- LCM: Lifecycle Manager
- Lifecycle: Add Node, Repair Node
- Networking: Host Networking, TOR Configuration, Software Defined Networking (SDN)
- Observability: Azure Local Telemetry and Diagnostics Agent
- Security: Azure Local Security
- Solution Extension
- Storage
- Update: Update Azure Local Solution
- Upgrade: Upgrade OS Versions

## Coding Standards

This repository should not contain any code files. No .ps1, .psm1, etc. All code should be embedded in the MD file with the correct type. For example:

```powershell
Get-Process
```

Any code snippets MUST:
- Be safe to execute in a production environment (consider any impact to workloads)
- Should use defensive coding tecniques
- Should check that the environment is in the expected state before performing the mitigation. For example, check that a Network Adapter is already disabled, before enabling it.

## Documentation Guidelines

All documentation should be clear and concise. It should follow one of the templates defined in `/TSG/Templates`


