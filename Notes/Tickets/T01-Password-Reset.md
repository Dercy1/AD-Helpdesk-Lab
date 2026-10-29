# T01: Forgotten Password (Jiva May)

**Date:** 2026-10-29
**Environment:** Windows Server 2022 Standard Evaluation (DC01, `lab.local`), simulated in VMware Workstation
**Category:** Account access

## Reported issue

Jiva May (`jmay`, Finance) forgot her password and cannot sign in.

## Initial checks

Before resetting anything, I checked the account state in Active Directory to rule out a disabled or locked account.

| Property | Value |
|---|---|
| Enabled | True |
| LockedOut | False |
| PasswordLastSet | 10/28/2026 8:12:03 PM |
| PasswordExpired | False |

**Finding:** The account was enabled, not locked out, and its password had not expired. A forgotten password was the remaining likely cause, so a password reset was the correct fix.

Evidence: `Screenshots/T01-1-baseline.png`

## Fix

1. Stored a temporary password as a secure string so it was never displayed on screen or typed into the command line:
   ```powershell
   $new = Read-Host "Temporary password" -AsSecureString
   ```
2. Reset the password as an administrator (no old password needed):
   ```powershell
   Set-ADAccountPassword -Identity jmay -Reset -NewPassword $new
   ```
3. Required the user to choose her own password at next sign-in, so the technician does not know her permanent password:
   ```powershell
   Set-ADUser -Identity jmay -ChangePasswordAtLogon $true
   ```

All three commands completed without errors.

Evidence: `Screenshots/T01-2-fix.png` (the password is masked)

## Verification

Re-ran the same account query after the fix.

| Property | Before | After |
|---|---|---|
| Enabled | True | True |
| LockedOut | False | False |
| PasswordLastSet | 10/28/2026 8:12:03 PM | (blank) |
| PasswordExpired | False | True |

A blank `PasswordLastSet` with `PasswordExpired: True` is how Active Directory records that a password change is required at next sign-in.

Evidence: `Screenshots/T01-3-verified.png`

## Scope and limits

- This verified the **account state in Active Directory**. It did not include an actual user sign-in, because no client computer had joined the domain at the time of this ticket.
- The account is a simulated user in a personal lab, with a lab-only password.
- The temporary password is not recorded in this repository.

## Skills practiced

Active Directory user management, account triage (disabled vs. locked vs. expired), secure password handling in PowerShell, verification against a baseline, support ticket documentation.
