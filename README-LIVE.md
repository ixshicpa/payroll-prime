# LIVE UPDATE — checklist (Prime Laundry / TSL / Prime CBL DTR & Payroll)

This folder has two parts. Nothing is deployed automatically; you upload it yourself.

## Part 1 — The app (GitHub)
1. In your GitHub repo (payroll-prime), keep a note of the current commit (that is your rollback point — you can always "Revert" or re-upload the previous index.html).
2. In the live app, take a backup first (Backup/Export in the app) just in case.
3. Upload ONLY `index.html` from `live-app-upload/` (replace the old index.html).
   - Do NOT upload any config.js — your existing live config.js (database-55b53) stays as it is.
4. Wait ~1 minute, open https://ixshicpa.github.io/payroll-prime/ and press Ctrl+F5.
5. If the old data does not show / app looks wrong: re-upload the previous index.html (rollback).

The update does not change or delete any existing data. It only READS `companies/tsl/portal_sync/*` for the portal import.

## Part 2 — The portal sync on your PC (portal-sync-live/)
This is a separate copy that writes ONLY to the LIVE database (it refuses to run with any other project's key) and only into `companies/tsl/portal_sync/*`. It reads (never changes) `companies/tsl/employees`.

1. Unzip `portal-sync-live` to a new folder, e.g. `C:\Users\MJ SABERDO\Downloads\portal-sync-live` (extract fully).
2. Firebase console → project **database-55b53** → Project settings → Service accounts → Generate new private key. Save as `service-account.json` inside `portal-sync-live\standalone`. (Keep it private; never upload it to GitHub.)
3. Open PowerShell in `portal-sync-live\standalone`:
   ```
   npm install
   powershell -ExecutionPolicy Bypass -File .\setup-credential.ps1     (enter the portal username/password)
   powershell -ExecutionPolicy Bypass -File .\run-sync.ps1
   type sync-log.txt
   ```
   The log should say `sync ok`. If it says `wrong_project`, the key is not from database-55b53.
4. In the live app → Import DTR → Portal sync: you should see the synced attendance. If it says "Missing or insufficient permissions", your live Firestore rules do not allow reading `portal_sync` — send me your Firestore rules and I will add the line.
5. When all is good, register the automatic tasks:
   ```
   powershell -ExecutionPolicy Bypass -File .\register-task.ps1
   ```
   (tasks: LivePortalAttendanceSync daily 6:00 AM, LivePortalNewStaffSync every 30 min).
6. Remove the old TEST tasks so they stop running:
   ```
   Unregister-ScheduledTask -TaskName PortalAttendanceSync -Confirm:$false
   Unregister-ScheduledTask -TaskName PortalNewStaffSync -Confirm:$false
   ```

Notes
- The PC must be on (and signed in) for the sync; missed runs catch up automatically.
- The live sync uses employees in the live database whose department contains "Laundry". Use the "Name in the portal" field if the portal spells a name differently.
- The test employee seeding script is NOT included here.
- Hik-Connect CSV upload for Prime Laundry/Prime CBL works as before in the app (no PC sync for it).


## UPDATE 2 — Branch names + Manage branches (new)
- Branch names now follow the portal: Igay 1 → **Igay RD**, Igay 2 → **Igay Skyline**, Taytay 1 → **Taytay Rizal**, Taytay 2 → **San Isidro Taytay**, Bagong Pag-asa → **Pag-asa Sablayan**.
- Import DTR → From portal → **Branches to sync → Manage branches**: add, rename (Edit), Pause/Resume or Remove a branch. The app shows, per branch, whether it was found in the portal (and the portal's closest names if not).
- The sync job follows this list. A branch you add or rename is looked up automatically by the 30-minute task (LivePortalNewStaffSync), same as a new Laundry employee. If you have not registered the tasks yet, run `run-sync.ps1 -NewStaff` by hand.
- The list is saved at `companies/tsl/portal_sync/branches_config` (same place as the other portal_sync data).

### How to apply UPDATE 2
1. GitHub: replace `index.html` with the new one from `live-app-upload` (no config.js).
2. PC: unzip `portal-sync-live-update.zip` OVER your existing portal-sync-live folder (3 files: functions\multi.js, standalone\sync-run.js, standalone\branches.json). Your service-account.json, portal-cred.xml and node_modules stay as they are.
3. In `standalone` run: `powershell -ExecutionPolicy Bypass -File .\run-sync.ps1` (full sync, picks up the renamed branches), then check `type sync-log.txt`.

## UPDATE 3 — Payroll frequency (new)
- Employees → Edit → **Payroll frequency**: *Every cut-off (1st and 2nd) — default*, *Once a month — 1st cut-off only*, or *Once a month — 2nd cut-off only*. Everyone already in the system stays on the default (nothing changes for them).
- For "once a month" staff the whole month is paid in one payout on the chosen cut-off: full salary + allowance, late/absences/OT/holiday pay of BOTH cut-offs, any company advances entered on either cut-off, and the SSS/PhilHealth/Pag-IBIG/withholding tax. The other cut-off tab does not list them (not in its totals, approval count, export or payslip list). The Monthly tab and the BIR figures are the same as before.
- Only index.html changed. Nothing to do on the PC.

## UPDATE 4 — Print all payslips, Payroll records, Company advances (new)
- Payroll → **Print all payslips**: every payslip in the open tab (1st, 2nd or Monthly), one per page.
- New tab **Payroll records**: saved automatically when a cut-off is approved (snapshot, does not change later).
  1st cut-off record: DTR summary, 1st cut-off payroll, payslips. 2nd cut-off record: DTR summary, 2nd cut-off payroll, payslips, Monthly, BIR Purposes. Print any section or export it to Excel.
  Cut-offs approved BEFORE this update: open them in Payroll and press **Save to Payroll records**. Reopening a cut-off marks its record "Reopened"; approving again replaces it.
- New tab **Company advances**: record an advance once (employee, amount, deduction per payday, first cut-off). The installment is added automatically to the employee's "Company advances" deduction in Payroll every payday until paid. Approving a cut-off records what was deducted; reopening removes it again. Summary per employee: total advanced, deducted so far, balance, next deduction. Edit / Pause / Close / Remove per advance.
  The old per-cut-off box is now **One-time advances** (still works for one-off amounts).
- New data (same company area as the rest): `advance_ledger`, `payroll_records`, `payroll_record_index`. If saving shows "permission" errors, the live Firestore rules need these collections allowed — send the rules.
- Only index.html changed. Nothing to do on the PC.
