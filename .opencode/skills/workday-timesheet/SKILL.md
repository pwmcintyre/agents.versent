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

## delegation

**Always delegate Workday browser tasks to a sub-agent via the Task tool.**

- Use `subagent_type: general` — specify model `claude-haiku-4-5` or similar small/fast model if the option exists; the reference docs are thorough enough that a smaller model handles this well
- This keeps Playwright snapshots and multi-step browser interactions out of the main context window
- The sub-agent should return a concise summary (not raw snapshots)

## sub-tasks

Read the relevant reference doc before starting:

| task | reference |
|---|---|
| Enter, submit, or correct timesheet hours | `reference/timesheets.md` |
| Request annual leave, personal leave, or any absence | `reference/leave.md` |
