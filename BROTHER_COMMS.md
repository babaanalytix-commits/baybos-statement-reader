# Draft message to your brother

Below is a ready-to-send draft. Edit names/details as needed.

---

## WhatsApp / email — short version

> Hey [brother's name],
>
> I've built a small tool that reads Baybos's bank statements and categorizes every transaction (Payroll, Books, Sports Day, Maintenance, Bank Charges, parent fees, etc.) so we can see at a glance where money's going each month.
>
> It runs entirely in your browser — the PDF never leaves your computer.
>
> **Link:** https://babaanalytix-commits.github.io/baybos-statement-reader/
>
> **What I need from you:**
>
> 1. Drop the GTCO statement PDF I'll send you next.
> 2. The tool will auto-categorize what it recognizes. Anything it doesn't lands in **Uncategorized**.
> 3. For each Uncategorized row (or any row in the wrong category), pick the right category from the dropdown. The tool remembers and uses that same category next month.
> 4. If there are a lot of corrections, click **Download CSV (categorized)**, fill in the Category column in Excel for any blanks/wrong rows, then click **Import master CSV** to upload your edited file — bulk training.
> 5. When done, click **Export my custom rules** and send me the JSON file. I'll merge it into the master tool so future statements are even better.
>
> Each month the tool gets smarter. Month 1 you'll do a lot of categorization. Month 2 most things are auto-tagged. By month 3 you should only be checking edge cases.
>
> This is the foundation — once it's reliable we're rolling it into the school website (`baybos.school`) so Aunty K can drop in any statement and get an instant breakdown.
>
> Any feedback, categories I'm missing, transactions that look strange — text me. Thanks for the help.
>
> Yomi

---

## Email — longer version

**Subject:** Baybos · web tool for categorizing the bank statements (~20 min the first time)

> Hi [brother's name],
>
> I've put together a small web tool to make Baybos's accounting easier. The idea: drop a GTCO bank statement PDF into the page, and every transaction is sorted into a school category — Payroll & Allowances, Books, Sports Day, Maintenance, Food, Bank Charges, parent fee receipts, etc. At the end you get clean CSVs that you can drop straight into your books.
>
> **Important — privacy:** the PDF is read inside your browser. Nothing uploads anywhere. The statement stays on your computer.
>
> **The link:** https://babaanalytix-commits.github.io/baybos-statement-reader/
>
> ## How to use it
>
> 1. Open the link in Chrome or Safari.
> 2. Drag the statement PDF onto the page (or click and choose it).
> 3. After a few seconds, you'll see a summary at the top: total transactions, total credits, total debits, net movement, opening → closing balance. The Total Debits and Total Credits should match the ones printed on page 1 of the statement (a green ✓ confirms that).
> 4. Below that: two breakdown tables (Expenses by category, Income by source) and a full transaction list.
> 5. **For each row, there's a Category dropdown.** If the tool got it wrong, just change it. Your change is remembered — the next time the same description appears (even in next month's statement), your category is applied automatically.
>
> ## The bulk-edit shortcut
>
> If a lot of rows are wrong or Uncategorized, use the CSV workflow:
>
> 1. Click **Download CSV (categorized)** — you get a spreadsheet with one row per transaction and a Category column.
> 2. Open it in Excel. For every row marked **Uncategorized** (or every row where the category is wrong), set the Category column to the right value.
> 3. Save the file.
> 4. Back on the tool, click **Import master CSV** and upload your edited file. Every Description → Category pairing becomes a permanent rule.
> 5. Re-run the statement and most of those will now auto-categorize.
>
> Repeat the cycle each month and the tool gets dramatically smarter — Month 1 takes the most work, Month 2 you'll mostly review, Month 3 you'll be doing almost nothing.
>
> ## What I'd love from you
>
> - **Categorize at least one month of statements** so we have a real working master rule-set. Pick the most recent month and walk through it.
> - **Tell me what's missing.** If there's a category I haven't included that you'd actually use (e.g. "Insurance", "Bank Loan Repayment", "Auditor's Fee"), let me know and I'll add it.
> - **Tell me what's wrong.** If a parent-fee payment isn't being recognized, or if a vendor is being mis-categorized, the tool can be tuned.
> - **Export your rules JSON when you're done** (button at the top of the transactions table). Send it to me. That way your training becomes part of the master tool so future months start from a richer baseline — and so anyone else who uses the tool benefits from your work.
>
> ## What this leads to
>
> Once we know the categorization is reliable, this becomes part of the school's website (`baybos.school`). Aunty K — or you, or whoever's running accounts — drops a statement into the management page, and a categorized monthly P&L appears. From there we can show fee collection vs target, biggest expense categories, month-on-month trends, all without anyone having to type a single transaction.
>
> Eventually we'll connect it directly to the school's bank account through an API (Mono or Okra in Nigeria) so the statements come in automatically.
>
> ## Categories currently supported
>
> Income: School Fees (Parents), Other Income, Refunds.
>
> Expense: Payroll & Allowances, Books & Educational Materials, Uniforms & Sports, Sports Day & Events, Food & Catering, Maintenance & Construction, Transport & Fuel, IT/Internet/Equipment, Health & First Aid, Legal & Admin, Sponsorship & Donations, Online Shopping, Bank Charges, Vendor/Supplier Payment, POS/Card Purchase, Other Transfer, Other Expense.
>
> Tell me which ones are missing and we'll add them.
>
> Thanks — and don't worry if you hit a snag, this is v3 and we're improving it as we go. Any feedback at all is useful.
>
> Yomi
