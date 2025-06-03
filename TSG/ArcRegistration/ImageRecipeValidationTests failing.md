# Troubleshooting Guide: Arc Registration Failing with ImageRecipeValidationTests

## Description
Arc Registration fails in pre-check with `ImageRecipeValidationTests`.

1. Open PowerShell as Administrator.

2. Run the following command:

   ```powershell
   Get-ScheduledTask -TaskName ImageCustomizationScheduledTask
   ```

3. If the state is **"Ready"**, start the task:

   ```powershell
   Get-ScheduledTask -TaskName ImageCustomizationScheduledTask | Start-ScheduledTask
   ```

4. Wait for the task to reach the **"Disabled"** state:

   ```powershell
   # Repeat this until the task state shows 'Disabled'
   Get-ScheduledTask -TaskName ImageCustomizationScheduledTask
   ```

---

Retry the Arc Registration Operation.

### 3. If Task Was Already Disabled

- If the scheduled task `ImageCustomizationScheduledTask` was **already Disabled**, this is **not a known issue**. Further debugging will be required.

---

## Summary Table

| Step | Check | Action |
|------|-------|--------|
| 3 | Task state is "Ready" | Start the task and wait for "Disabled" |
| 4 | Task already "Disabled" | Not a known issue, debug further |

---

## References
- [Bug 32995193 (Fixed in 2506)](https://dev.azure.com/msazure/One/_workitems/edit/32995193/)

---
**End of TSG**
