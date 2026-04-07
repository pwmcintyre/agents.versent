---
name: workday-timesheet
description: Fill in timesheets, request leave, or manage absences in Workday via the browser using Microsoft SSO. Use when the user asks to enter time, submit timesheets, or request annual/personal leave.
compatibility: opencode
---

## general

Use the `playwright-versent` browser (Microsoft SSO). SSO is automatic — no credentials needed.

**Workday home:** https://wd3.myworkday.com/aett/d/home.htmld

Standard day: **7.6 hours**. Standard week (no holidays): **38 hours**.

Always treat Workday's UI as the source of truth for dates, public holidays, and leave balances — never rely on training data for these.

## sub-tasks

Read the relevant reference doc before starting:

| task | reference |
|---|---|
| Enter, submit, or correct timesheet hours | `reference/timesheets.md` |
| Request annual leave, personal leave, or any absence | `reference/leave.md` |
