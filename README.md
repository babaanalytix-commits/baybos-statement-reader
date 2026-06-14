# Baybos Statement Reader

A simple, private bank-statement categorizer for **Baybos Home School**.

Drop a GTCO Bank PDF statement → see every transaction sorted into school categories (Payroll, Books, Sports Day, Maintenance, Bank Charges, School Fees, etc.) → download a clean CSV for the accounts.

**Privacy:** the PDF is read inside your browser. Nothing is uploaded anywhere.

**Live tool:** [open here](https://oguntona.github.io/baybos-statement-reader/) *(after you've published to GitHub Pages — see "Deploy" below).*

---

## What it does

1. Reads a GTCO Bank statement (PDF). All pages, multi-line transactions handled correctly.
2. Categorizes every transaction using rules tuned for Baybos's "GAPSLITE" expense narration convention. Examples:
    - `GAPSLITE T SHIRTS` → **Uniforms & Sports**
    - `GAPSLITE CEMENT TO MONIEMFB` → **Maintenance & Construction**
    - `GAPSLITE SATURDAY ALLOWANCE` → **Payroll & Allowances**
    - `MOBILE TRF TO GTB FATIMA BAYBOS HOME SCHOOL` → **School Fees (Parents)**
    - `Commission on NIP Transfer` → **Bank Charges**
3. Shows totals, per-category breakdowns, and a filterable transaction table.
4. Lets you **override any category** via dropdown. Your changes are remembered — the next time the same description appears (even in next month's statement), the tool uses your category automatically.
5. Exports a categorized CSV (line by line) or a summary CSV (per category).
6. Lets you export your custom rules as JSON (so you can share them, back them up, or move them to another browser).

---

## How the accountant uses it

1. Open the live tool URL on a laptop or desktop.
2. Drag the latest GTCO statement PDF into the drop zone.
3. Scan the **Expenses by category** and **Income by source** tables at the top — the categorization happens automatically.
4. Scroll to the **Transactions** table. For any row that's been put in the wrong category, change the dropdown. The tool learns from each correction.
5. Click **Download CSV (categorized)** for a row-by-row file, or **Download summary CSV** for a per-category roll-up.

The tool runs entirely in the browser. Closing the page does not lose your custom rules — they're stored locally and applied to every future statement you open.

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

## Deploy to GitHub Pages (one-time setup)

Once this is in a GitHub repo:

1. Push the `index.html` and `README.md` to a public GitHub repository (any name; `baybos-statement-reader` recommended).
2. In the GitHub repo → **Settings** → **Pages**.
3. Set **Source** to "Deploy from a branch", **Branch** to `main` (folder `/ (root)`), then click **Save**.
4. After ~30 seconds, the tool will be live at `https://<your-github-username>.github.io/<repo-name>/`.

Share that URL with anyone who needs to use the tool. No login, no account, no install.

---

## Running locally

Just open `index.html` in any modern browser (Chrome, Safari, Edge, Firefox). No build step, no dependencies — pdf.js is loaded from a CDN.

---

## Adapting the rules

The built-in rules live in the `BUILTIN_RULES` array near the top of the `<script>` block in `index.html`. Each rule is `[/regex/i, 'Category Name']`. The first regex that matches wins.

To add a new rule, paste it into the array in the right place (more specific rules go first). Example:

```js
[/gapslite party supplies/i, 'Sports Day & Events'],
```

---

## Privacy & data handling

- The PDF is read by the browser. No file or text leaves your device.
- Custom rules (your category overrides) are saved in **localStorage**, scoped to the browser on the machine you used. Clearing browser data clears the rules. Use **Export my custom rules** to back them up.
- pdf.js (the PDF reading library) is loaded from a public CDN at page load. After that, the page works fully offline.

---

## License

MIT. See [LICENSE](LICENSE).
