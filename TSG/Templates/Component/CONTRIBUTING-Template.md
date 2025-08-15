# Contributing to Azure Local {COMPONENT_NAME} Documentation

<!-- Instructions: Replace {COMPONENT_NAME} with the actual component name -->

This document extends the [universal contribution guidelines](../Templates/CONTRIBUTING.md) with component-specific requirements.

## Component-Specific Guidelines

<!-- Instructions: Replace this section with component-specific guidelines that are unique to this component area. Examples:
     - Networking: Different topologies (Storage Switched vs Switchless)
     - Storage: Different storage types (S2D, external storage)
     - Security: Different security contexts (on-premises vs cloud-connected)
     - Update: Different update channels and timing considerations
-->
{COMPONENT_SPECIFIC_GUIDELINES}

## File Naming

Follow the universal naming convention: `<Type>-<Topic>-<Specifics>.md`

For this component (`{COMPONENT_NAME}`), use these specific topic areas, or create new ones as needed:

### `<topics>`:
<!-- Instructions: List 4-6 main topic areas for this component. Use PascalCase without spaces. Examples:
     - Networking: ArcGateway, OutboundConnectivity, SDNExpress, TOR
     - Storage: S2D, ExternalStorage, Performance, Replication
     - Security: Authentication, Authorization, Certificates, Compliance
-->
- `{TOPIC_1}` - {TOPIC_1_DESCRIPTION}
- `{TOPIC_2}` - {TOPIC_2_DESCRIPTION}
- `{TOPIC_3}` - {TOPIC_3_DESCRIPTION}
- `{TOPIC_4}` - {TOPIC_4_DESCRIPTION}

## Structure

The repo is organized by major topic areas, you can add new files to existing folders or create new ones as needed. Here's a quick overview of the main folders:

<!-- Instructions: Create a table describing the main folders/topic areas for this component -->
| Folder                    | Description                                                          |
|---------------------------|----------------------------------------------------------------------|
| `{FOLDER_1}/`            | {FOLDER_1_DESCRIPTION}                                               |
| `{FOLDER_2}/`            | {FOLDER_2_DESCRIPTION}                                               |
| `{FOLDER_3}/`            | {FOLDER_3_DESCRIPTION}                                               |