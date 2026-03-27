# Skill: workday-timesheet

Complete or review a Workday timesheet on behalf of the user via the browser.

## when to use

Invoke this skill when the user asks to:
- fill in, submit, or review their Workday timesheet
- check what hours are missing or unsubmitted
- enter time for a specific day or week

## access

Use the `playwright-versent` browser (Microsoft SSO profile). SSO is automatic — no credentials are needed.

**Direct link (preferred):**
```
https://wd3.myworkday.com/aett/d/task/2998$10895.htmld
```

**Backup — navigate from Workday home:**
1. Go to https://myapps.microsoft.com/
2. Click the **Workday** tile — SSO signs you in automatically
3. In the Workday search box, type **"Enter My Time"** and select it from results (it appears in recent searches after first use)

## orientation

- The timesheet uses a **weekly calendar view** (Mon–Sun)
- Navigate weeks with **Previous Week** / **Next Week** buttons
- The **Summary** panel (bottom of page) shows total hours for the current week
- Time entries show a status: `Not Submitted` → submitted → `Approved`

## before entering time

Check each day carefully:

| condition | action |
|---|---|
| Day already has hours entered | skip — do not duplicate |
| Public holiday (e.g. Good Friday) | auto-populated — skip |
| Absence / leave already entered | auto-populated from absence portal — skip |
| Future day (not yet reached) | skip unless user explicitly asks to pre-fill |
| Submitted or approved | skip |

## entering time — Quick Add workflow

Use **Quick Add** (not "Enter Time for Worker" — that is a manager function).

1. Click **Actions** → **Quick Add**
2. Select **Time Type** → choose from Most Recently Used or search for the project
3. Click **Next**
4. In the hours grid, click the cell for each day and type the number of hours
5. Click **OK** → confirm "Quick Add Complete"
6. Click **Review** → the Submit Time dialog appears
7. Verify the date range and total hours shown
8. Click **Submit**

## submitting

- You can submit one or multiple days in a single Submit action — Workday batches the current week
- After clicking Submit, a confirmation dialog shows "You have submitted"
- Entries move to `Approved` status once the manager approves (often within minutes)

## time period locking

Workday locks time periods at month end (e.g. the March period locks 31 March). Entries for a locked period cannot be modified. Check the **Time Period End** / **Time Period Lockout** labels on entries.

## example project entry

> This is an example — confirm the correct project, billing type, and hours with the user before entering.

| field | example value |
|---|---|
| Project | PROJ-004249 CBA \| Digital Team Augmentation > Delivery > Build |
| Hours | 7.6 per day |
| Billing | Billable |
| Sub-type | Delivery > Build |
| Auto-filled | Project Role, Cost Center, Region, Practice |

Standard week: 7.6h × 5 days = 38h

## what to report back

After completing the timesheet, tell the user:
- Which days were filled and submitted
- Which days were skipped and why (holiday, leave, already entered, future)
- The total hours submitted
- Any warnings (e.g. period locking soon, entries not yet approved)
