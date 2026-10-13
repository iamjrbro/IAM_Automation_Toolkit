# IAM Automation Toolkit — Identity Audit

PowerShell toolkit for read-only identity auditing in Microsoft Entra ID using Microsoft Graph. This first module provides inventory and review reports for users, guests, potentially inactive accounts, active directory-role assignments, and disabled accounts with direct group memberships or direct app-role assignments.

> **Safety:** These scripts only read data and export reports. They do not disable accounts, remove memberships, revoke consent, or modify role assignments. Findings are review indicators, not automatic proof of risk.

## Features

- Read-only Microsoft Graph collection.
- Common connection, pagination, retry, logging, and report-export helpers.
- CSV, JSON, and HTML reports.
- Output timestamps and collection timestamps.
- Pester test examples that do not require a live tenant.
- Explicit permission notes and limitations.

## Requirements

- Windows PowerShell 5.1 or PowerShell 7.2+ (PowerShell 7 recommended).
- Microsoft Graph PowerShell Authentication module.
- A Microsoft Entra tenant and an account authorized to grant/use the requested delegated Graph scopes.

Install the dependency:

```powershell
Install-Module Microsoft.Graph.Authentication -Scope CurrentUser
```

Run from the repository root. The first run prompts for sign-in and consent as required.

## Scripts

| Script | Purpose | Requested delegated scopes |
|---|---|---|
| `Get-EntraIdentityInventory.ps1` | Inventory users and basic profile fields | `User.Read.All` |
| `Get-EntraGuestUsers.ps1` | Inventory guest users and external-user state | `User.Read.All` |
| `Get-EntraInactiveUsers.ps1` | Identify users with no sign-in timestamps within a threshold | `User.Read.All`, `AuditLog.Read.All` |
| `Get-EntraPrivilegedUsers.ps1` | Export active Entra directory-role assignments | `RoleManagement.Read.Directory`, `Directory.Read.All` |
| `Get-EntraDisabledUsersWithAccess.ps1` | Find disabled users with direct group membership and direct app-role assignments | `User.Read.All`, `GroupMember.Read.All`, `Application.Read.All` |

The actual minimum permission required can vary with endpoint behavior, tenant configuration, and whether delegated or application authentication is used. Review Microsoft Graph documentation and tenant consent policy before production use. These scripts currently use delegated sign-in.

## Usage

```powershell
# Full user inventory
.\src\IdentityAudit\Get-EntraIdentityInventory.ps1 -OutputDirectory .\reports -Format CSV

# Guest users as an HTML report
.\src\IdentityAudit\Get-EntraGuestUsers.ps1 -OutputDirectory .\reports -Format HTML

# Users with no recent sign-in activity (90-day threshold by default)
.\src\IdentityAudit\Get-EntraInactiveUsers.ps1 -InactiveDays 90 -OutputDirectory .\reports -Format CSV

# Include disabled accounts in the inactivity review
.\src\IdentityAudit\Get-EntraInactiveUsers.ps1 -InactiveDays 120 -IncludeDisabled -Format JSON

# Active Entra directory role assignments
.\src\IdentityAudit\Get-EntraPrivilegedUsers.ps1 -Format CSV

# Disabled accounts with direct group memberships or direct app-role assignments
.\src\IdentityAudit\Get-EntraDisabledUsersWithAccess.ps1 -Format HTML
```

Reports are timestamped and written to the selected output directory. Logs default to `./logs/`. Change `-LogPath` to choose a different log file.

## Important interpretation notes

- **Inactive users:** Sign-in activity can be unavailable because of licensing, permissions, retention, or reporting delays. A missing timestamp is reported as “No sign-in timestamp available”; it is not proof that the account has never been used. The script compares the available interactive and non-interactive timestamps, but it does not inspect every possible workload-specific activity source.
- **Privileged assignments:** This report includes active directory-role assignments returned by `/roleManagement/directory/roleAssignments`. It does not currently include PIM-eligible assignments, Azure resource RBAC assignments, or transitive group-based role membership. Treat it as a starting inventory, not a complete privilege graph.
- **Disabled users with access:** The report checks direct group memberships and direct app-role assignments only. It does not resolve nested/transitive group access, Azure RBAC, SharePoint permissions, or every downstream application authorization model. An assignment is a review item, not a recommendation to remove it automatically.
- **Guest users:** Guest classification is based on `userType = Guest`; it does not determine whether the guest is still sponsored or required by the business.
- **Read-only:** The scripts use GET requests and do not call mutation endpoints. Review code and permissions before running in a production tenant.

## Repository layout

```text
src/
  Common/IAM.Common.psm1
  IdentityAudit/
    Get-EntraIdentityInventory.ps1
    Get-EntraInactiveUsers.ps1
    Get-EntraGuestUsers.ps1
    Get-EntraPrivilegedUsers.ps1
    Get-EntraDisabledUsersWithAccess.ps1
tests/
docs/
```

## Testing

Install Pester:

```powershell
Install-Module Pester -Scope CurrentUser -Force
Invoke-Pester .\tests
```

The test suite is designed to validate the report helper and basic filtering logic without contacting a real tenant. Live Graph integration tests are intentionally not part of the default test run.

## Security and contribution notes

- Never commit client secrets, access tokens, tenant exports, real user data, or production logs.
- Use synthetic data in screenshots and sample reports.
- Request only the scopes needed for the selected script.
- Run PSScriptAnalyzer and Pester before opening a pull request.
- Keep write operations out of the Identity Audit module; any future remediation should be a separate, explicitly confirmed module.

## License

MIT — see `LICENSE`.
