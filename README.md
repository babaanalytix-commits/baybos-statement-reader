# Baybos Statement Reader

A private, browser-only tool that categorizes Nigerian bank statements for **Baybos Home School**.

Drop a bank statement PDF → see every transaction sorted into school categories (Payroll, Books, Sports Day, Maintenance, Bank Charges, parent fees, etc.) → download clean CSVs for the accounts.

**Privacy:** the PDF is read inside the browser. Nothing uploads anywhere.

**Live:** [open the tool](https://babaanalytix-commits.github.io/baybos-statement-reader/)

---

## Documents in this repo

| File | Purpose | Audience |
|---|---|---|
| `index.html` | The tool itself — single-file HTML/CSS/JS | Anyone (it's the live source) |
| `master-rules.json` | Shared categorization rules curated from accountant feedback | The tool fetches it at startup |
| `README.md` | Public-facing description | GitHub visitors |
| `MASTER_RULES_WORKFLOW.md` | How the rule-improvement loop works | **Yomi (repo maintainer)** — read carefully |
| `FABLE5_HANDOVER.md` | Integration spec for migrating the tool into baybos.school | **Fable5 (or whoever rebuilds the site)** — canonical spec |
| `BROTHER_COMMS.md` | Draft email/WhatsApp template for inviting the accountant | Yomi |
| `DEPLOY.md` | One-time GitHub Pages setup steps | Yomi |
| `LICENSE` | MIT | Anyone |

If you're going to lift this into another site (Fable5 / baybos.school), **read `FABLE5_HANDOVER.md` first and follow it meticulously.**

---

## What the tool does

1. **Reads bank statements.** Currently GTCO (Guaranty Trust Bank). Zenith, Access, FBN scaffolded — adding them is one new adapter object.
2. **Strips boilerplate.** Repeating page footers ("This is a computer generated Email…"), table headers, statement-level metadata — all removed before parsing.
3. **Determines debit vs credit** using a hybrid of keyword markers, balance arithmetic, and a safe default.
4. **Categorizes** every transaction using a 3-layer rule system: per-browser overrides (top priority) → shared `master-rules.json` (team rules) → built-in regex baseline.
5. **Lets the accountant correct anything.**
   - **Quick:** change the dropdown on any row. The tool remembers and applies that category to the same description in future statements.
   - **Bulk:** download the categorized CSV, fill in the Category column in Excel for any Uncategorized or wrong rows, then click **Import master CSV**. Every category you set becomes a rule.
6. **Exports** a row-by-row categorized CSV or a summary CSV (per category).
7. **Shares rules.** Export custom rules as JSON. Send to anyone, they import — instant training transfer. Merged into `master-rules.json` by the maintainer, then everyone benefits.

---

## How the process improves over time

| Month | Auto-categorized | Manual work |
|---|---|---|
| 1 | ~50% | Accountant fills in the rest, becomes rules |
| 2 | ~80% | Only new/unique transactions need attention |
| 3+ | ~95% | Almost just edge cases |

Each correction the accountant makes becomes a permanent rule in the browser's localStorage. Export the rules JSON periodically as a backup or to merge with the master tool. See `MASTER_RULES_WORKFLOW.md` for the full loop.

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

## Architecture (one-paragraph version)

The parser is split into per-bank adapters (`BANK_ADAPTERS` array) with a uniform interface — `detect`, `extractMeta`, `cleanNoise`, `parseTxns`. The categorizer is bank-agnostic, consulting localStorage → `master-rules.json` → `BUILTIN_RULES` in priority order. PDF text extraction uses pdf.js with column-aware line reconstruction. The normalized transaction model `{transDate, valueDate, remarks, debit, credit, balance}` is what bank API integrations (Mono, Okra, Plaid) will eventually produce too — one more adapter at the input side, no other code changes.

For the deeper integration spec, see `FABLE5_HANDOVER.md`.

---

## License

MIT. See [LICENSE](LICENSE).
