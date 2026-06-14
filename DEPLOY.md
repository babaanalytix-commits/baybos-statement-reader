# Publishing this tool to GitHub (so your brother gets a clickable link)

Two routes — pick whichever you prefer.

---

## Route A — terminal (5 commands)

If you have the GitHub CLI (`gh`) installed:

```bash
cd ~/Desktop/CoworkStation/baybos-statement-reader

git init
git add .
git commit -m "Initial: Baybos statement reader"
gh repo create baybos-statement-reader --public --source=. --push
gh repo edit --enable-pages --pages-branch main
```

Then enable Pages from the repo settings if the last command didn't:
- Visit `https://github.com/<your-username>/baybos-statement-reader/settings/pages`
- Source: **Deploy from a branch**, Branch: **main / (root)**, Save.

After ~30 seconds, your link will be:
```
https://<your-username>.github.io/baybos-statement-reader/
```

That's the URL to share with your brother.

---

## Route B — web UI (no CLI needed)

1. Go to https://github.com/new and create a new public repo called `baybos-statement-reader` (don't add a README — we already have one).
2. On the empty repo page, click **uploading an existing file**.
3. Drag in `index.html`, `README.md`, `LICENSE`, and the empty `.nojekyll` file from this folder (`~/Desktop/CoworkStation/baybos-statement-reader/`).
4. Commit straight to `main`.
5. Go to **Settings → Pages**. Source: "Deploy from a branch", Branch: `main` / `/ (root)`. Save.
6. Wait 30 seconds. The link will appear at the top of the same page.

---

## After it's live

- Share the GitHub Pages link with your brother. No login required on his side.
- Updates to the rules: edit `index.html`, commit, GitHub Pages redeploys automatically in ~30 seconds.
- Your brother's custom category overrides are stored in his browser's localStorage — they survive page reloads but live only on his machine. He can back them up via the **Export my custom rules** button.

---

## When Fable5 is back up

The two pieces from `index.html` you'd lift into the Fable5 management tool:

1. `BUILTIN_RULES` array (the category regexes) — line ~150 in `index.html`
2. `parseGtcoText` function (the PDF transaction extractor) — line ~270 in `index.html`

Everything else is presentation. Drop those two functions into your existing Fable5 page's JS, point them at the file the user uploads, and the same categorization runs natively in the site.
