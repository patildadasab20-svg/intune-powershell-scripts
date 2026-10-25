# Intune PowerShell Scripts

PowerShell scripts for Microsoft Intune and Entra ID administration using the Microsoft Graph PowerShell SDK. Written from scratch with no organisation-specific data.

## Scripts

| Script | Purpose |
|---|---|
| `scripts/Get-IntuneDeviceReport.ps1` | Exports Intune-managed Windows devices with compliance state and last sync time to CSV, and flags stale devices |

## Requirements
- PowerShell 5.1 or 7+
- Microsoft Graph PowerShell SDK: `Install-Module Microsoft.Graph -Scope CurrentUser`
- An account with permission to consent to `DeviceManagementManagedDevices.Read.All`

## Usage
```powershell
# All Windows devices
.\scripts\Get-IntuneDeviceReport.ps1

# Only non-compliant devices
.\scripts\Get-IntuneDeviceReport.ps1 -ComplianceState noncompliant -OutputPath .\noncompliant.csv

# Mark devices with no sync in 14 days as stale
.\scripts\Get-IntuneDeviceReport.ps1 -StaleDays 14
```

## Notes
- The script is read-only and makes no changes to devices.
- Test in a lab or test tenant before using in production.
- CSV output is excluded from the repository by `.gitignore`.

## Author
Dadasab Patil - [LinkedIn](https://linkedin.com/in/dadasab-patil-56190930a)
