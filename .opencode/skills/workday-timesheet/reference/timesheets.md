# Workday — Timesheets

## access

**Direct link (preferred):**
```
https://wd3.myworkday.com/aett/d/task/2998$10895.htmld
```

**Backup:** Workday home → search "Enter My Time"

## orientation

- Weekly calendar view (Mon–Sun)
- Navigate with **Previous Week** / **Next Week**
- Summary panel (bottom) shows total hours for the week
- Entry statuses: `Not Submitted` → submitted → `Approved`
- **Let Workday's calendar be the source of truth** for week boundaries and public holidays — do not pre-calculate from memory

## before entering time

| condition | action |
|---|---|
| Hours already entered | skip — do not duplicate |
| Public holiday (shown in calendar) | skip — read from the page, never rely on training data |
| Absence / leave | skip — auto-populated from absence portal |
| Future day | skip unless user explicitly asks to pre-fill |
| Submitted or Approved | skip |

## entering time — Quick Add workflow

Use **Quick Add** (not "Enter Time for Worker" — that is a manager function).

1. Click **Actions** → **Quick Add**
2. In the **Time Type** field: click **Most Recently Used** directly — do not type first; the search box has a delay and the category menu stays visible while loading
3. Select the project from MRU list; only fall back to search if not found there
4. Click **Next**
5. Click a cell in the hours grid to activate edit mode (all editable cells become textboxes)
6. Enter hours for each day that needs time
7. Click **OK** → confirm "Quick Add Complete"
8. Click **Review** → verify date range and total hours
9. Click **Submit**

## adding a comment to an entry

The **Comment** field is on the individual entry, not the weekly submission. It lives in the Details section of the "Enter Time" dialog (below Practice, above Attachments).

Quick Add does not expose the Comment field — you must edit each entry afterwards:

1. Click the entry link on the calendar (e.g. "VER-101 7.6 Hours...")
2. The **Enter Time** dialog opens
3. In the **Details** section, find the **Comment** textbox
4. Type the comment
5. Click **OK**

**VER project codes require a comment on every entry.** If entering a VER-prefixed project (e.g. VER-101-PS-STH Back to Base), always add a comment explaining the activity (e.g. "Internal meetings", "Travel", "Admin") before submitting. Ask the user what comment to use if not specified.

## deleting or correcting an entry

1. Click the entry link on the calendar (e.g. "CBA | Digital Team Augmentation 7.6 Hours...")
2. An **Enter Time** dialog opens — click **Delete**
3. Confirmation: "If you delete this time block, you may need to resubmit your time for approval." — click **OK**
4. "Delete was completed successfully" — hours drop to 0
5. Click **Review** → verify corrected total → click **Submit** to resubmit

**Note:** Deleting an approved entry un-submits the week. Always resubmit after deleting.

## submitting

- One Submit action covers the whole week
- Confirmation dialog shows "You have submitted"
- Entries move to `Approved` once the manager approves

## time period locking

Workday locks periods at month end (e.g. March locks 31 March). Locked entries cannot be modified. Look for **Time Period End** / **Time Period Lockout** labels on entries.

## example project

| field | value |
|---|---|
| Project | PROJ-004249 CBA \| Digital Team Augmentation > Delivery > Build |
| Hours | 7.6 per day |
| Billing | Billable |
| Sub-type | Delivery > Build |
| Auto-filled | Project Role, Cost Center, Region, Practice |

Standard week (no holidays): 7.6h × 5 days = 38h

## what to report back

- Which days were filled and submitted
- Which days were skipped and why
- Total hours submitted
- Any warnings (period locking, unapproved entries)
