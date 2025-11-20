# STIG ID: WN10-SO-000030 - Audit policy using subcategories must be enabled.

## Synopsis
This PowerShell script ensures that the Audit policy using subcategories is enabled.

## Notes
- **Author**: Dion Alexander
- **LinkedIn**: 
- **GitHub**: 
- **Date Created**: 2025-11-19
- **Last Modified**: 2025-11-19
- **Version**: 1.0
- **CVEs**: N/A
- **Plugin IDs**: N/A
- **STIG-ID**: WN10-SO-000030
-   
## Tested On
- **Date(s) Tested**: 
- **Tested By**: 
- **Systems Tested**: 
- **PowerShell Ver.**: 

## Usage
Put any usage instructions here.

Example syntax:
```powershell
# Enable audit policy subcategory settings (SCENoApplyLegacyAuditPolicy = 1)
# Alternate remediation script for the same STIG

function Test-IsAdmin {
    $currentIdentity = [Security.Principal.WindowsIdentity]::GetCurrent()
    $principal       = New-Object Security.Principal.WindowsPrincipal($currentIdentity)
    return $principal.IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
}

if (-not (Test-IsAdmin)) {
    Write-Host "This script must be run from an elevated PowerShell session (Run as Administrator)." -ForegroundColor Red
    exit 1
}

$RegistryPath = 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa'
$ValueName    = 'SCENoApplyLegacyAuditPolicy'
$DesiredValue = 1

Write-Host "Enforcing audit policy subcategory setting via $RegistryPath\$ValueName ..." -ForegroundColor Cyan

# Ensure the registry key exists
if (-not (Test-Path -Path $RegistryPath)) {
    Write-Host "Registry path does not exist. Creating $RegistryPath ..." -ForegroundColor Yellow
    New-Item -Path $RegistryPath -Force | Out-Null
}

# Get current value (if any)
$currentValue = (Get-ItemProperty -Path $RegistryPath -Name $ValueName -ErrorAction SilentlyContinue).$ValueName

if ($null -eq $currentValue) {
    Write-Host "Registry value $ValueName does not exist. Creating it now..." -ForegroundColor Yellow
    New-ItemProperty -Path $RegistryPath -Name $ValueName -Value $DesiredValue -PropertyType DWord -Force | Out-Null
}
else {
    Write-Host "Current $ValueName value: $currentValue"
    if ($currentValue -ne $DesiredValue) {
        Write-Host "Updating $ValueName from $currentValue to $DesiredValue ..." -ForegroundColor Cyan
        Set-ItemProperty -Path $RegistryPath -Name $ValueName -Value $DesiredValue -Force
    }
    else {
        Write-Host "$ValueName is already set to the required value ($DesiredValue). No change needed." -ForegroundColor Green
    }
}

# Verify
$verified = (Get-ItemProperty -Path $RegistryPath -Name $ValueName -ErrorAction Stop).$ValueName
if ($verified -eq $DesiredValue) {
    Write-Host "Verification successful: $ValueName is set to $verified." -ForegroundColor Green
    Write-Host "A reboot may be required for the audit policy change to fully take effect."
}
else {
    Write-Host "Verification FAILED: $ValueName is $verified, expected $DesiredValue." -ForegroundColor Red
    exit 1
}
```


