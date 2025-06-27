# Symptoms    
When updating your Azure Local instance to 2506 release, you encounter one of the following errors:

```
Could not update IgvmAgent on physical node Exception: Unable to create a new version folder with in 10 tries.
```
```
Could not update IgvmAgent on physical node Exception: Cannot remove item C:\CloudContent\IgvmAgent\_v2\AszIgvmAgent.exe: Access to the path ‘C:\CloudContent\IgvmAgent_v2\AszIgvmAgent.exe’ is denied.
```
Note, in the sample error message, _v2 refers to a version 2, which in practice can be v*, * range between 1 and 9.

# Issue Validation
      
Ensure the error message you notice matches one of the above error messages.

# Cause
This occurs because there is an underlying issue with removing certain folders. We have fixed this in 2506 release.

# Mitigation Details

For customers updating their Azure Local instance to 2505 release, if you encounter any of the above errors, perform the following steps to resolve the issue:

1. [Required] Restart Azure Local instance (all the machines in your Azure Local instance).
2. [Required] Resume Azure Local update (repeat steps to update your Azure Local instance).

Alternately, you can skip 2505 and directly update your Azure Local instance to 2506 release or above to resolve the issue.