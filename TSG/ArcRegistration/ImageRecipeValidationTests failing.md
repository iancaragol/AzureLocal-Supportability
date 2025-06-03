# Troubleshooting Guide: Arc Registration Failing with ImageRecipeValidationTests

# Error
Arc Registration fails in pre-check with `ImageRecipeValidationTests`.
  "Responses": [
    {
      "Name": "ImageRecipeValidation",
      "Status": "Failed",
      "Errors": [
        {
          "ErrorMessage": "Diagnostics failed for the test category: ImageRecipeValidation.",
          "StackTrace": null,
          "ExceptionType": "DiagnosticsTestFailedException",
          "RecommendedActions": [
            "Please contact Microsoft support."
          ]
        }
      ]
    }


# Issue Validation

1. Open PowerShell as Administrator.

2. Run the following command:

   ```powershell
   Get-ScheduledTask -TaskName ImageCustomizationScheduledTask
   ```
If the state is **"Ready"**, then is the expected issue and please follow the Resolution steps.

If the scheduled task `ImageCustomizationScheduledTask` was **already Disabled**, this is **not a known issue**. Further debugging will be required.

 ## Resolution
 
1. If the state is **"Ready"**, start the task:

   ```powershell
   Get-ScheduledTask -TaskName ImageCustomizationScheduledTask | Start-ScheduledTask
   ```

2. Wait for the task to reach the **"Disabled"** state:

   ```powershell
   # Repeat this until the task state shows 'Disabled'
   Get-ScheduledTask -TaskName ImageCustomizationScheduledTask
   ```

---

Retry the Arc Registration Operation.


