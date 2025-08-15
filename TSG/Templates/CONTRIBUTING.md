# Contribution Guidelines for Azure Local Supportability

## Overview

This repository provides troubleshooting guides (TSGs) and reference documentation for Azure Local. All contributions should try to follow these guidelines.

## Getting Started

### Quick Start for New Contributors

**Step 1: Identify Document Type**
What is the purpose of the document?
- **Troubleshoot a specific issue?** → Use `Troubleshoot-` prefix and [`TSG-Template.md`](TSG-Template.md)
- **Providing configuration examples?** → Use `Reference-` prefix and [`Reference-Template.md`](Reference-Template.md)
- **How to do x?** → Use `HowTo-` prefix and [`HowTo-Template.md`](HowTo-Template.md)
- **Explaining technical concepts in detail?** → Use `DeepDive-` prefix and [`DeepDive-Template.md`](DeepDive-Template.md)

**Step 2: Name Your File**
Use this pattern: `<type>-<topic>-<specifics>.md`

Examples:
- `Troubleshoot-StorageSpaces-DiskFailure.md`
- `Reference-TOR-BGPConfiguration.md`
- `HowTo-Deployment-PrepareActiveDirectory.md`
- `DeepDive-ArcGateway-OutboundTraffic.md`

**Step 3: Copy the Right Template**
1. Go to the [`/TSG/Templates/`](../Templates/) folder
2. Copy the template that matches your document type
3. Replace all the placeholder text (marked with `!!` or `[brackets]`)
4. Fill in the metadata table at the top

**Step 4: Write Safe Code**
- Always test your PowerShell/code examples first
- Use placeholders like `<hostname>` instead of real values
- Include comments explaining what each command does
- Add verification steps to confirm changes worked

**Step 5: Update your component's README and CONTRIBUTING**
Add your new file to your component's README.md table of contents. If you added a new `<topic>`, update the table of contents in the `CONTRIBUTING.md` file as well.

### Need Help?
- Check component-specific `CONTRIBUTING.md` files for additional guidance
- Review the [Guidelines](#guidelines) section below for detailed requirements

## Guidelines

### 1. Safety First
- All code examples MUST be safe to execute in production environments
- Use defensive coding techniques
- Check environment state before performing changes
- Provide rollback steps when applicable

### 2. File Naming Convention

Use the following naming schema for new files. Use CamelCase and hyphens to separate words.

```
<type>-<topic>-<specifics>.md
```

### `<type>`:
| Type           | Use for...                                    |
|----------------|-----------------------------------------------|
| `Overview`     | High-level summaries and architecture diagrams|
| `DeepDive`     | Detailed technical explanations               |
| `HowTo`        | Step-by-step deployment or config guides      |
| `Troubleshoot` | Troubleshooting known issues                  |
| `Reference`    | Reference configurations and resources        |

### `<topics>`:
- `ArcGateway` - Arc Gateway and related services
- `OutboundConnectivity` - Network connectivity and routing
- `SDNExpress` - Software-defined networking components
- `TOR` - Top of Rack switch management

These are examples from `Networking`, topics are defined based on the component area.

### `<specifics>`:
- Use descriptive keywords that summarize the content, e.g., `2Node-Switchless-Storage`,  `Recreate-Intent-No-SRIOV`, etc.
- Try to keep it concise but informative.

### 3. Required Template Usage
All TSGs must use the appropriate template from `/TSG/Templates/`

### 4. Code Examples
All code MUST:
- Be wrapped in appropriate code blocks with language specification
- Include comments explaining what the code does
- Be tested and verified safe
- Use placeholder values (e.g., `<hostname>`, `<ip-address>`)

### 5. Consistency Requirements
- Use the metadata table for all TSGs
- Follow the universal document structure
- Update the component README.md when adding new files
- Use proper Markdown formatting

## Document Types

| Type | Use Case | Template |
|------|----------|----------|
| `Overview` | High-level summaries and architecture diagrams | `Overview-Template.md` |
| `TSG` | Troubleshooting known issues | `TSG-Template.md` |
| `EnvironmentValidator` | Environment validator failures | `EnvironmentValidator-Template.md` |
| `Reference` | Configuration examples | `Reference-Template.md` |
| `HowTo` | Step-by-step guides | `HowTo-Template.md` |
| `DeepDive` | Technical explanations | `DeepDive-Template.md` |

## File Structure

Each Component can define their own file structure within the TSG folder. However, the following structure is recommended:

```
TSG/
└── [Component]/
    ├── README.md                    # Component overview and table of contents
    ├── CONTRIBUTING.md              # Component-specific contribution guidelines
    ├── [Topic-Area-1]/
    │   ├── images/                  # Screenshots, diagrams, etc.
    │   ├── examples/                # Configuration files, scripts, etc.
    │   ├── Reference-[Topic]-[Specific].md
    │   ├── Troubleshoot-[Topic]-[Specific].md
    │   └── HowTo-[Topic]-[Specific].md
    └── [Topic-Area-2]/
        ├── images/
        ├── DeepDive-[Topic]-[Specific].md
        └── Overview-[Topic]-[Specific].md
```

Any images or diagrams referenced in the documentation should be placed in the appropriate `images/` directory within the relevant topic area.

## Additional Considerations

### Renaming
Renaming files should be done with care to ensure that all references are updated accordingly. When renaming:
- Update any links or references in other documents
- Ensure the new name follows the established naming conventions
- Consider the impact on users who may have bookmarked or linked to the old file