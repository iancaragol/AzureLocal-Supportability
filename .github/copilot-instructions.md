# Project Overview

This is a public repository for all of Azure Local Troubleshooting guides (TSGs), known issues and reporting feedback - this repo is intended to provide a central location for community driven supportability content. This is the material that is referenced by Customer Support Services when a ticket is created, by Azure Local engineering responding to an incident, and by users when self discovering resolutions to active system issues.

## Documentation Standards
- Follow one of the Templates defined in the `/TSG/Templates` folder, where applicable
- Be consistent with existing TSGs and guidance (refer to `CONTRIBUTING.md`)
- Write in clear, concise technical English with proper headings and organization
- Use Markdown formatting consistently including tables, callouts, and code blocks
- README.md files should provide an overview of the folder contents and usage instructions, when adding a new document, the README.md should be updated accordingly.

## Contribution Workflow
- New files should follow naming convention: <Type>-<Topic>-<Specifics>.md
- Update component README.md files when adding new content
- Place images in an images/ subfolder within the relevant component

## PowerShell Code Guidelines
When reviewing or suggesting PowerShell code in documentation:
- Verify code is safe for production environments
- Implement defensive coding techniques (check conditions before taking action)
- Include verification steps before and after changes
- Ensure commands don't disrupt workloads
- Check for proper error handling
- Use placeholders like <hostname> instead of hardcoded values

```powershell
Example:

# DANGEROUS EXAMPLE - Could cause outage without warning
Restart-Service -Name "CriticalService" -Force

# SAFER ALTERNATIVE - Checks status and confirms before action
$serviceName = "CriticalService"
$service = Get-Service -Name $serviceName
Write-Host "Current status of $serviceName is: $($service.Status)"

$confirmation = Read-Host "Are you sure you want to restart $serviceName? (y/n)"
if ($confirmation -eq 'y') {
    Write-Host "Restarting $serviceName..."
    Restart-Service -Name $serviceName
    
    # Verify restart was successful
    $updatedStatus = (Get-Service -Name $serviceName).Status
    Write-Host "$serviceName status is now: $updatedStatus"
} else {
    Write-Host "Restart canceled"
}
```

## Document Types
- Troubleshoot: Helps users fix specific errors (symptoms → root cause → resolution)
- Reference: Provides configuration examples and settings
- How-To: Step-by-step instructions for specific tasks
- Deep Dive: Technical explanations and architecture details
- Overview: High-level introductions and summaries
