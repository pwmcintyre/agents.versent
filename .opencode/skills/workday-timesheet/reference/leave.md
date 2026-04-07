# Workday — Leave / Absence

## access

From Workday home (https://wd3.myworkday.com/aett/d/home.htmld), click **Request Absence** in the Quick Actions bar. Opens a dialog inline — no separate navigation needed.

## date selection

The dialog has two modes:

- **Calendar** — multi-select individual days (not suitable for contiguous ranges)
- **Date Range** — enter start/end dates; use this for consecutive leave

### Date Range spinbutton input — known behaviour

The date fields are segmented spinbuttons (Day / Month / Year). They **cannot** be filled with `fill()` or JS injection — React state will not update. Use this approach:

1. Click the **Date Range** tab
2. Click the visible date field container (~164×40px styled wrapper) to focus it
3. Type day digits (e.g. `09`) — auto-advances to Month after 2 digits
4. Press `Tab`, type month digits (e.g. `04`)
5. Press `Tab` to reach Year — **do not type the year**; digits are interpreted cumulatively and produce wrong values
6. Use **ArrowUp / ArrowDown** to increment/decrement the year to the target value
7. Repeat steps 2–6 for the End Date field
8. Verify with `page.getByRole('spinbutton').nth(i).inputValue()` before clicking Continue

## absence type selection

After clicking **Continue**, select **Type of Absence** — two-level dropdown:

**Level 1 — category:**
- Paid Leave
- Unpaid Leave
- Parental Leave

**Level 2 — specific type** (under Paid Leave):
- Annual Leave ← standard paid annual leave
- Personal Leave
- Compassionate Leave
- Long Service Leave
- Purchased Leave
- Study Leave
- Time Off in Lieu
- Versent Day
- Jury Duty
- Pandemic Leave
- Family Domestic Violence Leave
- 26 January Substitute

## daily hours and total

- **Hours (Daily)** defaults to 7.6 — leave as-is unless user specifies otherwise
- **Total Request Amount** shown at bottom — verify it equals expected working days × 7.6h
- Weekends and public holidays within the range are automatically excluded
- **Edit Individual Days** button available for per-day overrides

## payroll cutoff

Leave entered after the **5th of the month** for the current or previous month will appear on the **next** month's pay slip.

## submitting

Click **Submit Request** → success dialog: **"Success! Request Submitted"**. Goes to manager for approval.

## what to report back

- Date range submitted
- Leave type selected
- Total hours in the request
- Any payroll cutoff warning (if leave is for current/prior month and today is after the 5th)
