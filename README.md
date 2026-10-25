<#
.SYNOPSIS
    Exports a report of Intune-managed Windows devices to CSV.

.DESCRIPTION
    Connects to Microsoft Graph, retrieves Windows devices managed by Intune,
    and exports name, user, OS version, compliance state and last sync time.
    Optionally filters by compliance state and flags devices that have not
    synced recently.

.PARAMETER OutputPath
    Path of the CSV file to create.

.PARAMETER ComplianceState
    Filter by compliance state: All, compliant, noncompliant or unknown.

.PARAMETER StaleDays
    Devices with no sync in this many days are marked as stale.

.EXAMPLE
    .\Get-IntuneDeviceReport.ps1 -ComplianceState noncompliant -OutputPath .\noncompliant.csv

.NOTES
    Requires: Microsoft.Graph PowerShell SDK
    Permission: DeviceManagementManagedDevices.Read.All (read-only)
#>

[CmdletBinding()]
param(
    [string]$OutputPath = ".\IntuneDeviceReport.csv",

    [ValidateSet("All", "compliant", "noncompliant", "unknown")]
    [string]$ComplianceState = "All",

    [int]$StaleDays = 30
)

if (-not (Get-Module -ListAvailable -Name Microsoft.Graph.DeviceManagement)) {
    Write-Error "Microsoft Graph module not found. Run: Install-Module Microsoft.Graph -Scope CurrentUser"
    return
}

Connect-MgGraph -Scopes "DeviceManagementManagedDevices.Read.All" -NoWelcome

Write-Host "Retrieving Windows managed devices..." -ForegroundColor Cyan
$devices = Get-MgDeviceManagementManagedDevice -Filter "operatingSystem eq 'Windows'" -All

if ($ComplianceState -ne "All") {
    $devices = $devices | Where-Object { $_.ComplianceState -eq $ComplianceState }
}

$now = Get-Date
$report = foreach ($d in $devices) {
    $daysSinceSync = if ($d.LastSyncDateTime) { [int]($now - $d.LastSyncDateTime).TotalDays } else { $null }

    [pscustomobject]@{
        DeviceName        = $d.DeviceName
        UserPrincipalName = $d.UserPrincipalName
        OSVersion         = $d.OSVersion
        Manufacturer      = $d.Manufacturer
        Model             = $d.Model
        ComplianceState   = $d.ComplianceState
        LastSync          = $d.LastSyncDateTime
        DaysSinceSync     = $daysSinceSync
        Stale             = ($null -eq $daysSinceSync) -or ($daysSinceSync -ge $StaleDays)
    }
}

$report | Sort-Object ComplianceState, DeviceName | Export-Csv -Path $OutputPath -NoTypeInformation
Write-Host "Exported $($report.Count) devices to $OutputPath" -ForegroundColor Green

Disconnect-MgGraph | Out-Null
