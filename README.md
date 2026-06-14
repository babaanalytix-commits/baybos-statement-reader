# Baybos Statement Reader

A private, browser-only tool that categorizes bank statements for **Baybos Home School**.

Drop a bank statement PDF → see every transaction sorted into school categories (Payroll, Books, Sports Day, Maintenance, Bank Charges, School Fees, etc.) → download clean CSVs for the accounts.

**Privacy:** the PDF is read inside the browser. Nothing uploads anywhere.

**Live:** [open the tool](https://babaanalytix-commits.github.io/baybos-statement-reader/)

---

## What it does

1. **Reads bank statements.** Currently GTCO (Guaranty Trust Bank); Zenith, Access, FBN scaffolded.
2. **Strips boilerplate.** Each PDF page has a repeating footer ("This is a computer generated Email…") and table header — the parser removes these before extracting transactions.
3. **Auto-categorizes** using rules tuned for the school's "GAPSLITE" expense narration convention:
   - `GAPSLITE T SHIRTS` → **Uniforms & Sports**
   - `GAPSLITE CEMENT TO MONIEMFB` → **Maintenance & Construction**
   - `GAPSLITE SATURDAY ALLOWANCE` → **Payroll & Allowances**
   - `MOBILE TRF TO GTB … BAYBOS HOME SCHOOL` → **School Fees (Parents)**
   - `Commission on NIP Transfer` → **Bank Charges**
4. **Lets the accountant correct anything.** Two ways:
   - **Quick:** change the dropdown on any row. The tool remembers and applies the same category to that description in future statements.
   - **Bulk:** download the categorized CSV, fill in the **Category** column in Excel for any Uncategorized or wrong rows, then click **Import master CSV** and upload the edited file. Every category you set becomes a rule.
5. **Exports** a row-by-row categorized CSV or a summary CSV (per category).
6. **Shares rules.** Export your custom rules as JSON, send to someone else, they import — instant training transfer.

---

## How the process improves over time

| Month | Auto-categorized | Manual work |
|---|---|---|
| 1 | ~50% | Brother fills in the rest, becomes rules |
| 2 | ~80% | Only new/unique transactions need attention |
| 3+ | ~95% | Almost just edge cases |

Each correction the accountant makes becomes a permanent rule in the browser's localStorage. Export the rules JSON periodically as a backup or to merge with the master tool.

---

## Categories

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

---

## Architecture

### Bank adapters (extensible)

The parser is split into per-bank adapters. Each one knows how to:
- Detect its own bank from PDF text (`detect`)
- Extract statement metadata (`extractMeta`) — opening/closing balance, period, account
- Strip noise specific to its layout (`cleanNoise`) — page footers, table headers
- Parse transactions into a normalized model (`parseTxns`)

The normalized transaction model is bank-agnostic:

```js
{ transDate, valueDate, remarks, debit, credit, balance }
```

Adding a new bank means adding one adapter object — categorization, UI, CSV export, override tracking all work unchanged.

### Master CSV workflow

The "Import master CSV" workflow lets the accountant teach the tool in bulk:

1. Download the categorized CSV.
2. Open in Excel. For any row marked **Uncategorized** (or wrongly categorized), set the **Category** column to the correct value.
3. Save and upload via "Import master CSV". Every (Description, Category) pair becomes a rule.
4. The next statement uses those rules automatically.

Rules match on description text (lowercase, whitespace-normalized, long numeric references replaced by `#`). A shorter rule key is matched as a substring against longer descriptions — so a rule set on a small distinctive phrase applies broadly.

### Future: bank APIs

The same normalized transaction model works for bank API responses (Mono, Okra, Plaid). Adding API ingestion = one more adapter at the input side, no other code touched.

---

## Deploy

Already deployed at https://babaanalytix-commits.github.io/baybos-statement-reader/ via GitHub Pages from the `master` branch.

To update: commit + push to `master`. Pages redeploys in ~30 seconds.

---

## License

MIT. See [LICENSE](LICENSE).
