# How the rule-improvement loop works

This is the documentation for **you** (Yomi / repo maintainer). The accountant and parents don't need to read this.

## The three layers of rules

When the tool decides what category a transaction belongs to, it consults three sources in order of priority:

1. **Per-browser overrides** (`localStorage`) — highest priority.
   Lives in the user's browser. Their personal corrections.
2. **`master-rules.json`** — shared, in the repo.
   Cumulative team knowledge. Pulled by the tool at startup. Everyone sees these.
3. **`BUILTIN_RULES`** in `index.html` — regex baseline.
   Tuned to known GTCO/Baybos patterns. Ships in the code.

## The improvement loop

1. **Your brother (or any accountant) uses the tool.**
   They categorize Uncategorized rows, fix wrong ones. Their corrections write to **localStorage**.

2. **They click "Export rules JSON".**
   Downloads a `baybos-custom-rules.json` file with all their overrides.

3. **They send the file to you.**
   WhatsApp/email/Drive — whatever's convenient.

4. **You review the rules.**
   Open the JSON. Each entry is a `description-key → category` mapping. Spot-check that the categorizations look right.

5. **You merge the good ones into `master-rules.json`.**

   ```bash
   # Pull the latest
   cd ~/Desktop/CoworkStation/baybos-statement-reader
   git pull
   ```

   Open `master-rules.json`. Copy entries from your brother's exported JSON into the `rules` object. Example:

   ```json
   {
     "version": 1,
     "updated_at": "2026-06-15",
     "rules": {
       "transfer between customers via gapslite t shirts from baybos home school to opadoyin waheed": "Uniforms & Sports",
       "transfer between customers # -trf frm josephine odafen ojika to baybos home school": "School Fees (Parents)",
       "...": "..."
     },
     "regex_rules": []
   }
   ```

   You can also add **regex rules** for broader patterns:

   ```json
   "regex_rules": [
     ["trf frm [^\\n]+to baybos home school", "School Fees (Parents)"],
     ["mobile trf to gtb [^\\n]+baybos", "School Fees (Parents)"]
   ]
   ```

6. **Commit and push.**

   ```bash
   git add master-rules.json
   git commit -m "Add brother's June rules: 47 parent fees, 12 vendor payments"
   git push
   ```

7. **GitHub Pages redeploys in ~30 seconds.**
   Anyone opening the tool from then on gets the new rules automatically.

## How rule priority works in practice

If your brother has a localStorage override for "GAPSLITE T SHIRTS … OPADOYIN" set to **Uniforms & Sports**, and `master-rules.json` says the same thing → no conflict, his override applies.

If your brother has localStorage saying "Uniforms" but you changed `master-rules.json` to say "Sports Equipment" → his localStorage still wins for him. Other users see "Sports Equipment". This is intentional — local users can disagree with the team baseline.

## When `master-rules.json` gets large

Over time, this file will grow. A few hundred rules is fine — the tool parses it in a single pass at startup. If it grows past a few thousand entries, we'll split it into bank-specific files (`master-rules-gtco.json`, etc.) and the tool will load only the relevant one based on the detected bank.

## Future: backend rules store (optional)

Once the manual JSON-merge feels like friction, swap `master-rules.json` for a Cloudflare Worker + KV namespace. Same shape, fetched live, anyone with edit-permission can update from the tool itself. Free tier covers the whole school for years. Maybe 1 hour of work to migrate when we get there.

## For the Fable5 / baybos.school version

When this lands inside the management page on baybos.school:

- Same `BUILTIN_RULES` and `master-rules.json` files (or fetched from the same GitHub raw URLs).
- Same per-browser overrides for whoever is logged in to the management portal.
- Same Export/Import buttons.

The tool's architecture is bank-agnostic — adding new banks (Zenith, Access, FBN) is one adapter object in `BANK_ADAPTERS` array. Adding bank-API ingestion (Mono/Okra/Plaid) is one more adapter at the input side. None of the categorization, override, or master-rules logic changes.
