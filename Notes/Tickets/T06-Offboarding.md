# T06: Departing Employee Offboarding (Blordone Mardo)

**Date:** 2026-10-29
**Environment:** Windows Server 2022 Standard Evaluation (DC01, `lab.local`), simulated in VMware Workstation
**Category:** Account lifecycle / access removal

## Reported issue

Blordone Mardo (`bmardo`, IT) has left the company. His access must be removed.

## Initial state

| Property | Value |
|---|---|
| Enabled | True |
| Location | `OU=IT,OU=Lab Company,DC=lab,DC=local` |
| Group membership | Domain Users, IT-Staff |

Evidence: `Screenshots/T06-1-baseline.png`

## Approach

The account was **disabled and archived, not deleted**. Disabling blocks sign-in immediately while keeping the account available for audit, file-ownership lookups, or reinstatement. Access was removed in order of urgency: block sign-in first, then strip the permissions, then file and document the account.

## Steps taken

1. Reviewed group membership before changing anything:
   ```powershell
   Get-ADPrincipalGroupMembership bmardo | Select Name
   ```
2. Disabled the account to block sign-in right away:
   ```powershell
   Disable-ADAccount -Identity bmardo
   ```
3. Removed the group membership that granted permissions:
   ```powershell
   Remove-ADGroupMember -Identity IT-Staff -Members bmardo -Confirm:$false
   ```
4. Moved the account to the Disabled OU:
   ```powershell
   Move-ADObject -Identity (Get-ADUser bmardo).DistinguishedName -TargetPath "OU=Disabled,OU=Lab Company,DC=lab,DC=local"
   ```
5. Recorded the date and reason on the account:
   ```powershell
   Set-ADUser -Identity bmardo -Description "Disabled 2026-10-29 - left company (T06)"
   ```

All commands completed without errors.

Evidence: `Screenshots/T06-2-fix.png`

## Verification

| Property | Before | After |
|---|---|---|
| Enabled | True | False |
| Location | `OU=IT,...` | `OU=Disabled,OU=Lab Company,DC=lab,DC=local` |
| Description | (none) | Disabled 2026-10-29 - left company (T06) |
| MemberOf | IT-Staff | (empty) |

A second check from the group side, `Get-ADGroupMember IT-Staff`, returned no members. `Domain Users` is the account's primary group and does not appear in `MemberOf`, so it was left in place.

Evidence: `Screenshots/T06-3-verified.png`

## Scope and limits

- This verified the **account state in Active Directory**. It did not include an attempted sign-in from a client computer, because no client had joined the domain at the time of this ticket.
- The account is a simulated user in a personal lab.
- Real offboarding would also cover mailbox handling, file and data transfer, device return, and removal from other systems, none of which exist in this lab.

## Skills practiced

Active Directory account lifecycle management, group membership review, ordered access removal, OU organization, documenting the reason for a change, verification from two angles, support ticket documentation.
