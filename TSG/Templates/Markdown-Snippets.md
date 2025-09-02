# Markdown Formatting Reference

This document provides copy-paste snippets for common formatting elements used in Azure Local Supportability documentation.

---

## Alert & Emphasis Boxes

### Important Information

```markdown
> [!IMPORTANT]
> This is critical information that readers must understand before proceeding.
```

### Warning Messages

```markdown
> [!WARNING]
> This action could cause system downtime or data loss. Proceed with caution.
```

### Helpful Notes

```markdown
> [!NOTE]
> This provides additional context or clarification for the reader.
```

### Helpful Tips

```markdown
> [!TIP]
> This offers a useful suggestion or best practice.
```

### Caution Alerts

```markdown
> [!CAUTION]
> This warns about potential issues or risks to consider.
```

---

## Mermaid Markdown Diagrams and Flow Charts

For consistency and readability in Light and Dark mode, use the following flowchart template.

```mermaid
flowchart TD
    %% Basic nodes with different shapes/text formatting
    Start["Starting Node"]
    Decision{"Decision Node"}
    Action["Action Node"]
    Warning["Warning Node"]

    %% Different types of arrows and labels
    Start --> Decision                    %% Simple arrow
    Decision -- "Yes" --> Action         %% Arrow with text
    Decision --"No"--> Warning           %% Arrow with text (no spaces)
    Action ==> Warning                    %% Thick arrow
    Warning -.-> Start                    %% Dotted arrow
    
    %% Subgraph example
    subgraph ProcessGroup
        direction TB  %% Top to Bottom direction
        Step1["First Step"]
        Step2["Second Step"]
        Step3["Third Step"]

        Step1 --> Step2
        Step2 --> Step3
    end

    %% Connection to subgraph
    Decision --> ProcessGroup

    %% Styling definitions
    classDef action stroke:#dc2626,stroke-width:3px,fill:none;
    classDef warning stroke:#ea580c,stroke-width:3px,fill:none;
    classDef decision stroke:#2563eb,stroke-width:3px,fill:none;
    classDef success stroke:#059669,stroke-width:3px,fill:none;

    %% Apply styles to nodes
    class Action action;
    class Warning warning;
    class Decision decision;
    class Start success;
```

---

## Advanced HTML Callout Boxes

### Basic Note Box

<div style="border-left: 4px solid #0366d6; padding: 15px; margin: 20px 0; background: rgba(3, 102, 214, 0.1); border-radius: 6px;">
  <strong>📘 Note:</strong> This is a general note for additional information or clarification on a topic.
</div>

```html
<div
  style="border-left: 4px solid #0366d6; padding: 15px; margin: 20px 0; background: rgba(3, 102, 214, 0.1); border-radius: 6px;"
>
  <strong>📘 Note:</strong> This is a general note for additional information or
  clarification on a topic.
</div>
```

### Warning/Info Callout Box

<div style="border-left: 4px solid #f9c74f; padding: 15px; margin: 20px 0; background: rgba(249, 199, 79, 0.1); border-radius: 6px;">
  <strong>⏳ Waiting Time:</strong> Allow 10–15 minutes for logs to accumulate before proceeding with the next steps.
</div>

```html
<div
  style="border-left: 4px solid #f9c74f; padding: 15px; margin: 20px 0; background: rgba(249, 199, 79, 0.1); border-radius: 6px;"
>
  <strong>⏳ Waiting Time:</strong> Allow 10–15 minutes for logs to accumulate
  before proceeding with the next steps.
</div>
```

### Important Info Box

<div style="border-left: 4px solid #28a745; padding: 15px; margin: 20px 0; background: rgba(40, 167, 69, 0.1); border-radius: 6px;">
  <strong>⚠️ Important:</strong> Please ensure you follow the recommended steps carefully to avoid unintended issues.
</div>

```html
<div
  style="border-left: 4px solid #28a745; padding: 15px; margin: 20px 0; background: rgba(40, 167, 69, 0.1); border-radius: 6px;"
>
  <strong>⚠️ Important:</strong> Please ensure you follow the recommended steps
  carefully to avoid unintended issues.
</div>
```

### Common Causes Box

<div style="border-left: 4px solid #dc3545; padding: 15px; margin: 20px 0; background: rgba(220, 53, 69, 0.1); border-radius: 6px;">
  <h4 style="margin-top: 0; color: #dc3545;">Common Causes</h4>
  <ul>
    <li>Potential cause one</li>
    <li>Potential cause two</li>
    <li>Potential cause three</li>
  </ul>
</div>

```html
<div
  style="border-left: 4px solid #dc3545; padding: 15px; margin: 20px 0; background: rgba(220, 53, 69, 0.1); border-radius: 6px;"
>
  <h4 style="margin-top: 0; color: #dc3545;">Common Causes</h4>
  <ul>
    <li>Potential cause one</li>
    <li>Potential cause two</li>
    <li>Potential cause three</li>
  </ul>
</div>
```

### Advanced Tips Box

<div style="border-left: 4px solid #6f42c1; padding: 12px; margin: 20px 0; background: rgba(111, 66, 193, 0.1); border-radius: 6px; font-size: 0.9em;">
  <strong>💡 Tip:</strong> This box can provide advanced tips or optional steps for users seeking deeper insights.
</div>

```html
<div
  style="border-left: 4px solid #6f42c1; padding: 12px; margin: 20px 0; background: rgba(111, 66, 193, 0.1); border-radius: 6px; font-size: 0.9em;"
>
  <strong>💡 Tip:</strong> This box can provide advanced tips or optional steps
  for users seeking deeper insights.
</div>
```

---

## Code Blocks

### PowerShell Commands

````markdown
```powershell
# Description of what this command does
Get-Process | Where-Object {$_.ProcessName -eq "example"}
```
````

### Console/Terminal Output

````markdown
```console
# Network configuration example
interface Ethernet1/1
  description Azure Local Node Connection
  switchport mode trunk
  switchport trunk allowed vlan 100,200,711,712
```
````

### JSON Configuration

````markdown
```json
{
  "property": "value",
  "setting": {
    "enabled": true,
    "timeout": 30
  }
}
```
````

### Generic Code Block

````markdown
```
Generic text or configuration content
that doesn't fit a specific language
```
````

---

## Tables

### Basic Specifications Table

```markdown
| Requirement   | Specification | Notes                    |
| ------------- | ------------- | ------------------------ |
| **Component** | Details here  | Additional context       |
| **Setting**   | Value here    | Important considerations |
```

### Comparison Table

```markdown
| Feature         | Option 1          | Option 2   | Option 3    |
| --------------- | ----------------- | ---------- | ----------- |
| **Performance** | High              | Medium     | Low         |
| **Complexity**  | Low               | Medium     | High        |
| **Use Case**    | Small deployments | Enterprise | Specialized |
```

### Metadata Table (for TSG documents)

<table border="1" cellpadding="6" cellspacing="0" style="border-collapse:collapse; margin-bottom:1em;">
  <tr>
    <th style="text-align:left; width: 180px;">Component</th>
    <td><strong>Component Name</strong></td>
  </tr>
  <tr>
    <th style="text-align:left; width: 180px;">Topic</th>
    <td><strong>Topic Name</strong>: Brief description</td>
  </tr>
</table>

```markdown
<table border="1" cellpadding="6" cellspacing="0" style="border-collapse:collapse; margin-bottom:1em;">
  <tr>
    <th style="text-align:left; width: 180px;">Component</th>
    <td><strong>Component Name</strong></td>
  </tr>
  <tr>
    <th style="text-align:left; width: 180px;">Topic</th>
    <td><strong>Topic Name</strong>: Brief description</td>
  </tr>
</table>
```

---

## Emojis

```markdown
- ✅ Recommended approach
- ⚠️ Proceed with caution
- 🚫 Not recommended
- 🔧 Requires configuration
- 📋 Note
```

---
