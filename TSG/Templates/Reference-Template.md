<!-- 
Reference Template
- Focus on providing configuration guidance, technical specifications, and examples
- Reference documents help readers understand what needs to be configured and what's possible
- Replace all {placeholders} with relevant content
- This template provides a suggested structure - adapt it to make sense for your specific content
    - The goal is clarity and usability for the reader

Styling
- Images should be placed in the `./images` folder and referenced
- Any code block or JSON should be wrapped in triple backticks (```) with language identifier
- References to Azure Local public documentation should always direct to the latest version
- Use tables for specifications, requirements, and comparisons
- Use code blocks for configuration examples

You can use this regex to find placeholders that need to be replaced (search by Regex in your editor): \{([^}]+)\}
-->
# {Title}

<table border="1" cellpadding="6" cellspacing="0" style="border-collapse:collapse; margin-bottom:1em;">
  <tr>
    <th style="text-align:left; width: 180px;">Component</th>
    <td><strong>{Component Name}</strong></td>
  </tr>
  <tr>
    <th style="text-align:left; width: 180px;">Topic</th>
    <td><strong>{Topic Name}</strong>: {Brief description of what this reference covers}</td>
  </tr>
  <tr>
    <th style="text-align:left; width: 180px;">Scope</th>
    <td>{Brief description of what configurations, scenarios, or deployment patterns this covers}</td>
  </tr>
</table>

## Overview

{Brief description of what this reference document provides - configurations, specifications, examples, etc.}

## Scope

### In Scope
{What configurations, scenarios, or deployment patterns this reference covers}
- {Configuration type 1}
- {Scenario 1}
- {Deployment pattern 1}

### Out of Scope
{What is not covered by this reference - helps set expectations}
- {Configuration type not covered}
- {Scenario not applicable}

## Requirements

{Technical requirements, prerequisites, or constraints that apply to the configurations in this reference}

| Requirement | Specification | Notes |
|-------------|---------------|--------|
| {Requirement 1} | {Specification} | {Additional context} |
| {Requirement 2} | {Specification} | {Additional context} |

## Table of Contents

{Update Table of Contents as needed - organize sections to best serve your readers}
- [Overview](#overview)
- [Scope](#scope)
- [Requirements](#requirements)
- [Configuration Reference](#configuration-reference)
- [Examples](#examples)
- [Validation](#validation)

## Configuration Reference

{Main configuration content - adapt structure based on your topic}

### {Configuration Area 1}

{Description of this configuration area and when it's used}

#### {Sub-configuration or Option 1}

{Explanation of this specific configuration}

```console
# Configuration example with comments
{configuration commands}
```

**Key Parameters:**
- `{parameter1}`: {Description and purpose}
- `{parameter2}`: {Description and purpose}

#### {Sub-configuration or Option 2}

{Explanation of alternative or additional configuration}

### {Configuration Area 2}

{Description of another configuration area}

## Examples

{Real-world configuration examples that demonstrate the concepts}

### {Example Scenario 1}

{Description of the scenario this example addresses}

**Environment:**
- {Environment detail 1}
- {Environment detail 2}

**Configuration:**
```console
{Complete configuration example}
```

### {Example Scenario 2}

{Description of another scenario}

## Validation

{Methods to verify that configurations are working correctly}

### {Validation Method 1}

{Description of what this validates}

```console
# Validation command
{command to run}
```

**Expected Output:**
```console
{sample of expected output}
```

### {Validation Method 2}

{Another validation approach}

## Configuration Comparison

{Optional section - use when comparing different configuration options}

| Feature | {Option 1} | {Option 2} | {Option 3} |
|---------|------------|------------|------------|
| {Feature 1} | {Value} | {Value} | {Value} |
| {Feature 2} | {Value} | {Value} | {Value} |
| {Use Case} | {When to use} | {When to use} | {When to use} |

## Related Documentation

{Links to related documents, official documentation, and additional resources}
- {Related document 1}(link) - Brief description
- {Related document 2}(link) - Brief description
- {Official documentation}(link) - Brief description

---