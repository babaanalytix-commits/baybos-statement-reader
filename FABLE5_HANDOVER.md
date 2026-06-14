# Fable5 Handover Brief — Statement Reader Integration into baybos.school

**Read this file carefully and follow it meticulously when integrating the statement reader into the Baybos Home School website.**

This document is the canonical specification for moving the standalone GitHub Pages tool into the school's Management portal on baybos.school. Every section matters. If something here conflicts with what's currently on the site, this document wins.

---

## 0. Quick context

- **What it is:** a browser-only categorizer for Nigerian bank statements (currently GTCO/Guaranty Trust Bank), tuned for Baybos Home School's expense narration conventions.
- **What it produces:** auto-categorized transactions, summary tables (Income by source, Expenses by category), and clean CSV exports for the school's accounting.
- **Privacy guarantee — non-negotiable:** the PDF and all data stay in the user's browser. Nothing uploads anywhere. The existing management portal explicitly says "Everything stays in your browser; nothing is uploaded." That promise must remain true after integration.
- **Reference implementation:** https://babaanalytix-commits.github.io/baybos-statement-reader/
- **Reference source:** https://github.com/babaanalytix-commits/baybos-statement-reader
- **Where this needs to live on the site:** the Management portal (currently linked from the homepage as "Management → Open demo →"). The portal already has a "💷 Finance — read your bank statement" panel with a placeholder drag-drop. Replace that placeholder's logic with this tool's engine.

---

## 1. The pieces to lift, in order

Source file: `index.html` in the repo above. Copy from it in this order:

### 1.1 `CATEGORIES` array (lines ~166-176)

The list of available categories. Used in dropdowns, the categorization output, and CSV exports. Do not rename them; existing rules reference these strings exactly. The order of items in this array is the dropdown order — keep it.

```javascript
const CATEGORIES = [
  'School Fees (Parents)', 'Other Income', 'Refunds',
  'Payroll & Allowances', 'Books & Educational Materials',
  'Uniforms & Sports', 'Sports Day & Events', 'Food & Catering',
  'Maintenance & Construction', 'Transport & Fuel',
  'IT, Internet & Equipment', 'Health & First Aid',
  'Legal & Admin', 'Sponsorship & Donations', 'Online Shopping',
  'Bank Charges', 'Vendor / Supplier Payment', 'POS / Card Purchase',
  'Other Transfer', 'Other Expense', 'Uncategorized',
];
```

### 1.2 `BUILTIN_RULES` array (lines ~180-217)

Regex baseline categorization. Order matters — the first matching pattern wins, so the more specific rules go first. Bank charges are detected first so they don't get mis-categorized as something else. GAPSLITE patterns (the school's expense-prefix convention) come next.

### 1.3 `descKey()` function (lines ~265-273)

Normalises a transaction's remarks into a stable string used as a key for override lookup. Long numeric references (8+ digits) are replaced with `#` so the same transaction across different statements (e.g., a parent paying again with a different transaction reference) generates the same key.

### 1.4 `categorize()` function (lines ~275-301)

Three-layer priority resolver:

1. **Per-browser localStorage overrides** (the user's manual corrections). Highest priority. A description-key match wins, and a shorter override key matching as a substring of a longer description also wins (for broader rule application).
2. **`master-rules.json`** (shared team rules, fetched at startup). Middle priority. Same matching strategy.
3. **`BUILTIN_RULES`** (regex baseline). Lowest priority.

This priority order is the foundation of the improvement loop documented in `MASTER_RULES_WORKFLOW.md`. Preserve it exactly.

### 1.5 `BANK_ADAPTERS` array (lines ~310-465)

Pluggable per-bank parsers. Each adapter exposes:

- `name` — display name shown in the bank badge.
- `code` — short identifier (e.g., `'gtco'`).
- `detect(text)` — returns `true` if the extracted text appears to come from this bank.
- `extractMeta(text)` — returns `{ period, accountNo, accountType, currency, openingBalance, closingBalance, totalDebit, totalCredit }`.
- `cleanNoise(text)` — returns the text with bank-specific page headers, footers, and other repeating noise stripped.
- `parseTxns(cleanedText, meta)` — returns an array of normalized transaction objects `{ transDate, valueDate, remarks, debit, credit, balance }`.

`detectAdapter(rawText)` iterates the array in order and returns the first match. **Keep GTCO as `BANK_ADAPTERS[0]`** — it's the only fully-implemented one. The others (Zenith, Access, FBN) intentionally throw an explanatory error from `parseTxns()` so the user sees "not yet implemented" rather than silently failing.

### 1.6 PDF text extraction (lines ~470-510)

Uses `pdf.js` (the Mozilla library, loaded from cdnjs). Per-page text reconstruction is column-aware:

- Group items by Y coordinate (rounded to nearest 2 units) to identify lines.
- Sort lines top-to-bottom (PDF y-axis is bottom-up, so descending Y is top first).
- Within each line, sort items left-to-right by X coordinate.
- Insert larger whitespace between items separated by big X gaps (column boundaries).

Do not modify this without testing against the reference statement (see "Testing" section). Subtle changes break the GTCO parser.

### 1.7 GTCO `cleanNoise()` (lines ~330-370)

**This is the part that gives the parser the most trouble.** GTCO statements have:

- A repeating page footer: `This is a computer generated Email. Please address all enquiries to Guaranty Trust Bank Plc Systems and Control Group 178, Awolowo Road, Ikoyi. P.O.Box 75455, Victoria Island, Lagos, Nigeria. Phone 01-2694276 | Fax 01-2694276 Or the Customer Information Unit of your local branch.1.`
- A repeating table header on every page: `Trans. Date | Value. Date | Reference | Debits | Credits | Balance | Originating Branch | Remarks`.
- Statement-level metadata on page 1: account no, address, opening/closing balance, total debit/credit.
- The branch identifier "635 AKIN ADESOLA" on every transaction row.

If these aren't stripped, they contaminate every transaction's description and break categorization. The current implementation uses a hybrid approach: line-based filtering (drop entire lines matching known noise patterns) + inline regex stripping (for cases where pdf.js merged multiple logical lines into one). Keep both — neither alone is sufficient.

### 1.8 GTCO `parseTxns()` (lines ~375-420)

Uses regex `(date)\s+(date)` to find transaction starts (each GTCO row begins with `Trans Date` `Value Date`). Between two date-pair anchors, the chunk contains: amount, balance, branch identifier, remarks. The chunk is parsed by:

- Extracting numeric values (`#,##0.##` or `0.##` formats). Each transaction has exactly 2 numbers — the amount (debit or credit) and the balance. The LAST number is always the balance.
- Stripping dates, numbers, and leading apostrophes/pipes from the chunk to get clean remarks.

### 1.9 `_resolveDebitCredit()` (lines ~425-475)

**The hardest part.** Determining whether each transaction is a debit or a credit uses three signals in order:

1. **Keyword markers in the remarks** — `TRF FRM` (transfer from), `MOBILE TRF TO GTB … BAYBOS HOME SCHOOL`, `SCHOOL FEES`, `Tuition` → credit. `TRF TO`, `GAPSLITE`, `Stamp Dut`, `VAT charges`, `Commission`, `NIBSS Outward`, `POS`, `Web transfer` → debit.
2. **Balance arithmetic** — given prior balance and current balance, the amount must equal one of `prior - amount` or `prior + amount`. Whichever is closer to the actual `current` wins. Used only if keyword markers are absent or ambiguous.
3. **Default to debit** — school is mostly an outflow; if all else fails, debit is the safer bet.

A second pass logs (but does not auto-correct) total-vs-header discrepancies for the operator to see.

---

## 2. UI integration on baybos.school

The current Management portal at `https://baybos.school/#mgmt` already has a "💷 Finance — read your bank statement" panel. Replace its placeholder with the full tool. Specifically:

### 2.1 Layout adaptation

- Drop zone — match the existing portal's drop-zone styling. Don't reuse the standalone tool's colors verbatim if they clash with the portal's design.
- Bank detection badge — show at the top of results: `Bank: Guaranty Trust Bank (GTCO)`, plus statement period and account number badges (small pills under the drop zone).
- Summary grid — five stats (Transactions, Total credits, Total debits, Net movement, Opening → Closing). The Total credits stat should also show "✓ matches header" or "⚠ off by …" comparing parsed totals against the bank's printed totals (sanity check the parser).
- Expenses-by-category panel — collapsible table with category, amount, count, percentage bar.
- Income-by-source panel — same format.
- Transactions table — every row has a Category dropdown for inline override, a Description column (truncate to ~220 chars), Debit / Credit / Balance columns.

### 2.2 Controls panel above transactions

Eight buttons + a search box, in this order: Search description… (input) · Download CSV (categorized) · Download summary CSV · Import master CSV · Export rules JSON · Import rules JSON · Clear my rules.

The hidden file inputs sit alongside (no visible icon) and are triggered by the buttons.

### 2.3 Empty state

Before any file is loaded, show a yellow help card with the 5-step "How this works" instructions. Hide it after the first parse.

### 2.4 Debug panel (collapsible)

At the bottom of the results, a `<details>` section labelled "🔧 Debug · view extracted text + parser diagnostics". Inside: parse statistics (bank, pages, items, raw lines, cleaned lines, transactions, opening/closing balance), buttons to download raw + cleaned text, and a `<pre>` preview of the first 60KB of cleaned text. This panel is only useful when something goes wrong — most users will never open it. Don't remove it. Keep it discoverable but collapsed.

### 2.5 Toast notifications

After every 10 manual category overrides, show a non-blocking toast (bottom-right) suggesting the user export their rules JSON as a backup. Also show toasts for CSV-import success/failure and rule-import success/failure.

---

## 3. The improvement loop

This is the single most important design decision. **Read `MASTER_RULES_WORKFLOW.md` (in the same folder as this brief).** Do not skip it.

Summary: rules live in three places — `BUILTIN_RULES` (in code), `master-rules.json` (shared file in repo), per-browser `localStorage`. The shared layer is the team's accumulated knowledge. The improvement loop is:

1. Accountant uses tool → makes corrections → corrections live in localStorage.
2. Accountant clicks "Export rules JSON" → sends file to Yomi.
3. Yomi reviews and merges good rules into `master-rules.json` → commits → deploys.
4. Next time anyone opens the tool, they get those rules automatically.

For the baybos.school integration: **`master-rules.json` should be fetched from the same repo on GitHub** (`https://raw.githubusercontent.com/babaanalytix-commits/baybos-statement-reader/main/master-rules.json`) OR copied into the site's static assets and kept in sync. The first approach is preferable — single source of truth, no drift between the GitHub Pages tool and the integrated site version.

Fetch with cache-busting (`?t=` + timestamp) so updates appear within a single deploy cycle.

---

## 4. Categories (full list, do not rename)

| Income | Expense |
|---|---|
| School Fees (Parents) | Payroll & Allowances |
| Other Income | Books & Educational Materials |
| Refunds | Uniforms & Sports |
|  | Sports Day & Events |
|  | Food & Catering |
|  | Maintenance & Construction |
|  | Transport & Fuel |
|  | IT, Internet & Equipment |
|  | Health & First Aid |
|  | Legal & Admin |
|  | Sponsorship & Donations |
|  | Online Shopping |
|  | Bank Charges |
|  | Vendor / Supplier Payment |
|  | POS / Card Purchase |
|  | Other Transfer |
|  | Other Expense |

Plus `Uncategorized` (the default when no rule matches and no override exists). Categories the school may want to add over time: Insurance, Loan Repayment, Auditor's Fee, Taxes (PAYE), Pension. These are not in the current list — wait for the school to ask before adding them.

---

## 5. Files to copy / link

From the GitHub repo into the baybos.school project:

| File | Where it goes | Notes |
|---|---|---|
| `index.html` (lines ~165-820) | Embed JS into the Management portal page's existing script tag, or load as a module | Strip the standalone tool's `<header>` and `<body>` chrome — keep only the engine |
| `master-rules.json` | Either fetch live from the GitHub raw URL, or `cp` into the site's static assets and refresh on each deploy | Live fetch preferred |
| `MASTER_RULES_WORKFLOW.md` | Reference doc for Yomi — does not need to ship to the site | Keep accessible for ongoing maintenance |

Do not copy: `LICENSE`, `DEPLOY.md`, `BROTHER_COMMS.md`, `README.md`. These are repo housekeeping, not site assets.

---

## 6. Dependencies

The tool uses one external library, loaded from a CDN:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
```

With the worker:

```javascript
pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.worker.min.js';
```

Use exactly this version. Mozilla pdf.js 4.x changed several APIs the parser depends on. If a self-hosted copy is preferred (for offline-first), mirror these two files into the site assets and update the paths — the rest of the code is unchanged.

---

## 7. Roadmap (do not implement now, but design for)

These are the four directions this tool grows in. Reflect them in any architectural decisions today so we don't have to refactor later.

### 7.1 Multi-bank support

`BANK_ADAPTERS` is already pluggable. Adding Zenith, Access, FBN, UBA, Stanbic, etc. is one new adapter object per bank. Do not collapse this into a single parser. When a new bank's first statement comes in, implement that bank's adapter and add it to the array.

### 7.2 Bank API integration (Mono, Okra, Plaid)

The current management portal already mentions "🔗 Connect your bank (coming soon) — Nigerian open-banking (e.g. Mono or Okra)". When implemented, this becomes one more input source feeding the same normalized transaction model `{ transDate, valueDate, remarks, debit, credit, balance }`. The categorization, override storage, and UI work unchanged.

When connecting, store API tokens in the user's session only (not localStorage — those leak across browser refreshes). Categorize each transaction on receipt. Send periodic webhooks from the API into a small endpoint that triggers re-categorization and refreshes the Management dashboard.

### 7.3 Backend rules store (replaces master-rules.json)

When manual JSON-merging by Yomi feels like friction (likely after ~3 months of use), migrate the shared rules layer to a Cloudflare Worker + KV namespace. Same shape. Reads are public. Writes are auth-gated (only Yomi and Aunty K). Free tier covers the school for years. Estimated effort: 1 hour.

### 7.4 Dashboard rollup

Once a year of statements have been categorized, surface trends on the Management dashboard: monthly fee collection vs target, top 5 expense categories, year-over-year comparison, GAPSLITE category trends. Don't build a new charting library — Chart.js is already loaded on the site. The data is already in localStorage from past statement parses (or could be cached there explicitly after each parse).

---

## 8. Testing checklist

Before declaring the integration done:

1. Drop the February 2026 GTCO statement (sample available from Yomi). Confirm:
   - **Bank badge** shows "Guaranty Trust Bank (GTCO)"
   - **Transactions count**: between 240 and 260 (the actual number for that statement is ~248)
   - **Total credits**: matches the "Total Credit" printed on page 1 (`9,070,880.63`)
   - **Total debits**: matches the "Total Debit" printed on page 1 (`13,593,065.88`)
   - **Opening balance**: `8,024,261.58`
   - **Closing balance**: `3,502,076.33`
   - Summary stat shows "✓ matches header" in green
2. Spot-check 10 random transactions:
   - Description is NOT contaminated with "This is a computer generated Email…" footer text
   - Debit OR credit (not both, not zero) is populated correctly
   - Balance is non-zero
   - Category is plausible
3. Categorization quality (rough targets):
   - Bank Charges: ~50-70 transactions (Stamp Duty + Commission + VAT)
   - GAPSLITE expense categories: ~100-150 transactions across various categories
   - School Fees (Parents): ~30-50 transactions
   - Uncategorized: should be the minority — under 30% of total
4. Click any Category dropdown, change it, reload the page — the change persists.
5. Click "Download CSV (categorized)" — file downloads, opens cleanly in Excel.
6. Click "Export rules JSON" — file downloads with the user's overrides.
7. Empty the localStorage (Settings → Site Settings → Storage → clear for the page). Reload. Drop the statement again. Open `master-rules.json` from the repo, paste a few `rules` entries, redeploy. Reload the page. The new categories from `master-rules.json` apply automatically.

If all 7 pass, the integration is correct.

---

## 9. Things that have already been tried — do not repeat the mistakes

These approaches were attempted and abandoned. Don't re-try them without reading the history.

- **Greedy "footer regex" that matches the entire footer block in one shot.** pdf.js splits the footer across multiple text items in unpredictable ways. A single greedy regex fails on roughly 1 in 4 statements. The line-based filter + bounded inline strip is the correct approach.
- **Pure balance-arithmetic for debit/credit determination.** Fails when even one balance is misparsed (which happens when the description chunk is contaminated). The keyword-first hybrid is more robust.
- **Trusting cadence_log-style summary counts vs. ID-level matching.** This is a different domain (the engine's MD/ORACLE pipelines) but the same lesson applies: when summaries and details disagree, the details win and you should trust ID-level matching.
- **Storing rules only in localStorage, no shared layer.** Each user redoes the same work. The shared `master-rules.json` layer is the answer; preserve it.

---

## 10. When in doubt

Ask Yomi. If Yomi is unreachable, ask the team's accountant (Yomi's brother, who is the primary user). If both are unreachable, default to the safer choice: never auto-submit anything, never silently change a user's rules, always default to debit (school is mostly outflow), always categorize as Uncategorized rather than guess.

This document is the canonical spec. Treat any deviation from it as a defect unless explicitly approved.

---

## Appendix A — Repository layout (current state)

```
baybos-statement-reader/
├── index.html                  # The tool itself (the source of truth for code to lift)
├── master-rules.json           # Shared rules layer (start empty, grows over time)
├── README.md                   # Public-facing description (for GitHub visitors)
├── LICENSE                     # MIT
├── DEPLOY.md                   # How to publish to GitHub Pages (one-time)
├── BROTHER_COMMS.md            # Draft email/WhatsApp to the accountant
├── MASTER_RULES_WORKFLOW.md    # How the rule-improvement loop works (for Yomi)
├── FABLE5_HANDOVER.md          # This document (for you, Fable5)
└── .nojekyll                   # Tells GitHub Pages to serve files as-is
```

## Appendix B — Brand and tone

Match the rest of baybos.school: Christian school, 20 years of operation, "Studying to be approved" (2 Timothy 2:15), Aunty K's voice — warm, practical, no jargon. The Management portal is the most operational/technical part of the site, so dense tables are fine, but keep button labels in plain English ("Download CSV (categorized)" not "Export Data v1"), avoid emoji noise (a few are fine — 📊, 💸, 💰, 📋 — but not in every header).
