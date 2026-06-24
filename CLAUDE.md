# CLAUDE.md

## Project

Single-file HTML call center dashboard: `Ytel_Daily_Monitor_ADP.html`
No framework, no build step — vanilla JS + SheetJS 0.18.5 + Chart.js 4.4.0 via CDN.

## Rules

- Read existing files before editing.
- Make the minimum necessary change.
- Do not rewrite entire files unless requested.
- Preserve existing styling and layout.
- Do not remove features unless asked.
- Keep explanations under 5 sentences.
- Ask before changing business logic.
- Run JS syntax check after edits: `node -e "const fs=require('fs');const h=fs.readFileSync('Ytel_Daily_Monitor_ADP.html','utf8');const s=h.match(/<script>([\s\S]*?)<\/script>/g);s.forEach((b,i)=>{try{new Function(b.replace(/<\/?script>/g,''));console.log('OK',i);}catch(e){console.log('ERR',i,e.message);}});"`
- Always push to branch `claude/confident-allen-ddaf00` on `pixelme1369/Ytel_Daily_Monitor_ADP`, then merge to `main` when asked.

## Agent Roles

```js
const CLOSERS = new Set([...]);   // closers — no Opener tag, no >2min % display
const RETENTION = new Set([...]);  // retention agents
const OPENERS = new Set([...]);    // openers — show Opener tag, show >2min % in color
```

Agents not in any set show no role tag.
All three sets are defined at lines ~588–590 of `Ytel_Daily_Monitor_ADP.html`.

## Enrollment Logic

- One enrollment per unique phone number per day
- `Cordoba Enrolled Date` must match the analysis date
- Credit goes to the agent named in **`Assigned To`** column (if populated)
- Fallback: agent with the latest call timestamp to that phone on that day
- Debt comes from `Enrolled Debt` column on the enrolled row

## Hourly Breakdown Logic

- **Unique #s per hour** — phone counted in the hour of its **first call of the day**
- **Enrolled per hour** — credited to the hour of the agent's **last call to that enrolled phone**

## Missed Callbacks Logic

- Tracks inbound `TIMEOT` calls and checks for any follow-up call (inbound OR outbound, non-TIMEOT) after the last timeout
- Phone is only flagged if **zero follow-up activity** occurred after the timeout
- Split into: Enrolled Clients (have `Cordoba Enrolled Date`) vs Other Calls
- Filterable by campaign dropdown

## Agent Call Funnel Table

- Columns show % of agent's total calls in each time bracket
- **Sorting sorts by raw count (the number in parentheses), not percentage**
- Filterable by role (All / Closers / Openers) and campaign/direction dropdowns

## Openers — Transfer Breakdown by Campaign

Card is only visible when opener calls exist. Features:
- Time brackets: Dead ≤30s, 30s–2min, 2–5min, 5–10min, 10–15min, 15–20min, 20–30min, 30+min
- Each bracket cell shows: transfer count (clickable → phone number modal), enrolled count, total calls
- **Clicking a transfer count opens the enrolled phones modal with that bracket's transferred phone numbers**
- Filterable by campaign and direction dropdowns
- Click agent row to expand per-campaign sub-rows

### Time Bracket Accumulation (seconds)

| Bracket | Range |
|---------|-------|
| Dead ≤30s | sec ≤ 30 |
| 30s–2min | 31 ≤ sec < 120 |
| 2–5min | 120 ≤ sec < 300 |
| 5–10min | 300 ≤ sec < 600 |
| 10–15min | 600 ≤ sec < 900 |
| 15–20min | 900 ≤ sec < 1200 |
| 20–30min | 1200 ≤ sec < 1800 |
| 30+min | sec ≥ 1800 |

### Per-bracket counters (in `emptyS`)

- `rXtoY` — total calls in bracket
- `cltrns_XtoY` — CLtrns calls in bracket
- `enroll_XtoY` — enrolled AND CLtrns calls in bracket (used in breakdown cell)
- `enroll_all_XtoY` — enrolled calls in bracket regardless of transfer status (used in summary)
- `phones_XtoY` — Set of phone numbers for CLtrns calls (clickable modal)
- `enroll_short` — enrolled calls with sec ≤ 30 (for summary Dead row)

### Openers Summary Table

Appears below the breakdown. Rows = time brackets. Columns: Total Calls | Total Transfers | Total Enrolled.
- "Total Enrolled" uses `enroll_all_XtoY` (all enrolled in bracket, not just transfers).
- Totals must match the breakdown TOTAL row exactly.

### Openers — Lowest Transfer Rate (>2 min)

Ranking table below the summary. Shows every opener agent sorted by transfer rate ascending (worst first).
- `calls` = sum of all brackets over 2 min (r2to5m + r5to10 + r10to15 + r15to20 + r20to30 + gt30m)
- `trns` = sum of CLtrns across same brackets
- Rate = trns / calls, color-coded: red < 15%, neutral 15–30%, green ≥ 30%

## Key Columns Expected in XLSX

| Column | Notes |
|--------|-------|
| `call_date` | datetime, SheetJS reads as Date object with `cellDates:true` |
| `phone_number` / `phone_number_dialed` | phone number |
| `length_in_sec` | duration in seconds |
| `status` | dispo code (SALE, DNC, DROP, TIMEOT, CLtrns, CC, etc.) |
| `direction` | inbound / outbound |
| `full_name` | agent display name |
| `user` | agent ID (exclude VDCL / VDAD rows from agent table) |
| `campaign_id` | campaign identifier |
| `source_id` | VDCL / AGENTDIRECT / etc. |
| `Cordoba Enrolled Date` | date-only cell — SheetJS returns UTC midnight Date object |
| `Enrolled Debt` | dollar amount, may include `$` and commas |
| `Assigned To` | enrollment credit override |
| `CRM Status` | lead status |

## Date Parsing (`getDateStr`)

- Date objects at UTC midnight → use `getUTCFullYear/Month/Date` (date-only cells like `Cordoba Enrolled Date`)
- Date objects with time → use local `getFullYear/Month/Date` (datetime cells like `call_date`)
- Strings: match `YYYY-MM-DD` or `M/D/YYYY`

## Branch & Deploy

- Development branch: `claude/confident-allen-ddaf00`
- Repo: `pixelme1369/Ytel_Daily_Monitor_ADP`
- Merge to `main` when user asks to ship
- No CI, no build — just open the HTML file in Chrome
