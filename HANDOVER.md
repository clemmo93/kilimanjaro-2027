# Handover: Kilimanjaro for Dad → GitHub Pages

**Prepared:** 29 September 2026
**Owner:** Matthew Clementson (GitHub: `clemmo93`, taken from the existing `europe-703-2027` repo; confirm before running)
**For:** Claude Code, run on Matthew's Mac from this folder
**Goal:** publish this folder as a new GitHub repository and serve it on GitHub Pages, following the same pattern as `clemmo93/europe-703-2027`.

---

## 0. What you're publishing

Two static pages with no build step, taken from a claude.ai artifact and its companion Claude doc:

| File | Role | Notes |
|---|---|---|
| `index.html` | Interactive comparison (the main page) | ~52 KB. Inline CSS and JS; only external request is Google Fonts. Uses `localStorage` for remembered filters and ticked to-dos (keys `kili-view`, `kili-todo`). |
| `write-up.html` | Full written research with sources | ~48 KB. Static HTML generated from the Claude doc. Contains a static SVG rainfall chart. |
| `README.md` | Repo readme | Summary, file list, privacy note. |
| `HANDOVER.md` | This file | Keep it in the repo so future edits have context. |
| `.nojekyll` | Empty file | Stops GitHub Pages running Jekyll. Must be committed. |

The two pages link to each other with **relative** links (`index.html` ↔ `write-up.html`). There are no absolute paths, so the site works at a project URL like `https://clemmo93.github.io/kilimanjaro-2027/` or from a `/docs` folder without changes.

Already done before handover, so you don't need to redo it:

- Added a full `<!doctype html>`, `<head>`, `lang="en-GB"`, charset, viewport, description, Open Graph tags and an inline SVG favicon. The claude.ai host used to supply these.
- Added the small base reset the claude.ai host used to inject: `body{margin:0}`, `[hidden]{display:none!important}`, safe-area padding. The cost calculator relies on the `[hidden]` rule.
- Replaced the two links to the private claude.ai doc with links to `write-up.html`.
- Set `<meta name="robots" content="noindex, nofollow">` on both pages (see §2).
- Checked both pages in Chromium: no console errors, no horizontal scroll at 390 px width, correct in light and dark mode.

---

## 1. Decisions to confirm with Matthew before running

Ask these in one go, then proceed. The defaults are sensible if he says "just do it".

| Decision | Default | Why it matters |
|---|---|---|
| GitHub account | `clemmo93` | Taken from the remote of `europe-703-2027`. Confirm with `gh api user --jq .login`. |
| Repo name | `kilimanjaro-2027` | Becomes the URL path: `https://clemmo93.github.io/kilimanjaro-2027/`. |
| Repo visibility | **Public** | Pages on a private repo needs GitHub Pro, Team or Enterprise. **The published site is public either way.** |
| Search indexing | **Off** (`noindex, nofollow`) | This is a family planning page that mentions his dad and health preparation. It's reachable by link but not listed by search engines. Change only if Matthew asks. |

---

## 2. Safety rules for this job

1. **Only initialise git inside `kilimanjaro-2027-site/`.** The parent folder (`Personal / Health`) also contains `Medical Records/` and other personal files. Never run `git init`, `git add` or `gh repo create --source` from the parent folder. Before the first commit, run `git status` and confirm that only the five files in §0 are staged.
2. Don't add a build system, framework, package.json or GitHub Actions workflow. The site is intentionally plain files served from `main` / root.
3. Don't remove `.nojekyll`.
4. Don't change prices, dates or recommendations while publishing. Content changes are a separate task (§7).
5. Don't flip `noindex` to `index` unless Matthew explicitly asks.

---

## 3. Prerequisites

```bash
git --version                 # any recent git
gh --version                  # GitHub CLI
gh auth status                # must show logged in to github.com as clemmo93
gh api user --jq .login       # expect: clemmo93
```

If `gh` isn't authenticated, stop and ask Matthew to run `gh auth login`. Don't try to handle tokens yourself.

---

## 4. Publish: step by step

The folder path contains spaces (`Personal / Health`), so always quote it.

### 4.1 Go to the site folder and check its contents

```bash
cd "/Users/mpc93/Documents/Claude/Projects/Personal / Health/kilimanjaro-2027-site"
ls -la
# Expect exactly: .nojekyll  HANDOVER.md  README.md  index.html  write-up.html
```

If the folder is somewhere else (for example Downloads), `cd` there instead. The same rule applies: it must be the folder that contains `index.html` directly.

### 4.2 Initialise and commit

```bash
git init -b main
git add .nojekyll README.md HANDOVER.md index.html write-up.html
git status          # confirm only these five files are staged
git commit -m "Kilimanjaro for Dad: 2027 comparison site"
```

### 4.3 Create the GitHub repo and push

```bash
gh repo create clemmo93/kilimanjaro-2027 \
  --public \
  --description "Kilimanjaro 2027 options for Dad and me: operators, routes, timing and costs" \
  --source=. \
  --remote=origin \
  --push
```

If the repo name is taken, ask Matthew for another name. Don't overwrite an existing repo.

### 4.4 Turn on GitHub Pages (deploy from `main`, root)

```bash
gh api -X POST repos/clemmo93/kilimanjaro-2027/pages \
  -f build_type=legacy \
  -f "source[branch]=main" \
  -f "source[path]=/"
```

- If this returns **409** (Pages already enabled), update the config instead:

  ```bash
  gh api -X PUT repos/clemmo93/kilimanjaro-2027/pages \
    -f build_type=legacy -f "source[branch]=main" -f "source[path]=/"
  ```

- If it returns **422** about the plan, the repo is private without a paid plan. Make it public (`gh repo edit clemmo93/kilimanjaro-2027 --visibility public --accept-visibility-change-consequences`) only after confirming with Matthew.
- UI equivalent: **Settings → Pages → Build and deployment → Source: Deploy from a branch → `main` / `(root)` → Save.**

### 4.5 Wait for the first build

```bash
for i in $(seq 1 30); do
  s=$(gh api repos/clemmo93/kilimanjaro-2027/pages/builds/latest --jq .status 2>/dev/null)
  echo "build: ${s:-pending}"
  [ "$s" = "built" ] && break
  [ "$s" = "errored" ] && { gh api repos/clemmo93/kilimanjaro-2027/pages/builds/latest; break; }
  sleep 10
done
gh api repos/clemmo93/kilimanjaro-2027/pages --jq '.html_url, .status'
```

A first build usually takes one to two minutes. If `builds/latest` returns 404 at first, keep polling; the first build is queued after the push.

### 4.6 Set the repo's website link

```bash
gh repo edit clemmo93/kilimanjaro-2027 --homepage "https://clemmo93.github.io/kilimanjaro-2027/"
```

---

## 5. Verify the live site

### 5.1 From the command line

```bash
BASE="https://clemmo93.github.io/kilimanjaro-2027"
curl -s -o /dev/null -w "index %{http_code}\n"    "$BASE/"
curl -s -o /dev/null -w "write-up %{http_code}\n" "$BASE/write-up.html"
curl -s "$BASE/" | grep -o "<title>[^<]*</title>"             # <title>Kilimanjaro for Dad</title>
curl -s "$BASE/" | grep -c 'noindex, nofollow'                 # 1
curl -s "$BASE/" | grep -c 'claude.ai'                         # 0
```

A 404 in the first few minutes is normal while the CDN catches up. Retry for up to 10 minutes before treating it as a problem.

### 5.2 In a browser (manual checklist for Matthew)

Open `https://clemmo93.github.io/kilimanjaro-2027/` and check:

- [ ] The recommendation panel shows **Lemosho, 8 days · September 2027 · ≈ 85–90% odds · £7,400–9,400**.
- [ ] Operators: tier chips filter the cards; the "Oxygen and health checks included" tick-box narrows the list; each card shows the price each, the climb for you both and the all-in for you both.
- [ ] Routes: clicking a route chip redraws the altitude profile and the facts panel.
- [ ] When to go: clicking a month updates the detail panel; September is selected by default.
- [ ] Cost: with the defaults (Kandoo, September flights, $375 tips, some kit, the two of you sharing), it shows **≈ £4,150 each and ≈ £8,300 for you both**. Switching to "One person only" hides the "for you both" row.
- [ ] Next steps: ticking a box survives a page reload (this uses `localStorage` in that browser only).
- [ ] "Full write-up with sources →" opens `write-up.html`, and its "Interactive comparison →" link comes back.
- [ ] It looks right on a phone and in dark mode.

---

## 6. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| 404 at the URL right after enabling | First build or CDN not finished | Wait up to 10 minutes; check `pages/builds/latest`. |
| Build `errored` | Jekyll tried to process the files | Confirm `.nojekyll` is committed at the repo root. |
| 422 when enabling Pages | Private repo on a free plan | Make the repo public (with Matthew's OK) or use a paid plan. |
| Page loads but fonts look plain | Google Fonts blocked (network, privacy extension) | Nothing to fix. The CSS has fallback font stacks. |
| Old version still showing after a push | Pages CDN caching (around 10 minutes) | Hard refresh, or wait. |
| Filters or to-dos not remembered | Private window or blocked site data | Expected. The page works without storage. |
| Wrong base URL in links | Someone added absolute `/…` links | Keep all internal links relative. |

---

## 7. Updating the content later

All data lives inline in `index.html`, in one `<script>` block near the end. Line numbers are as delivered and will drift.

| What | Where in `index.html` | Notes |
|---|---|---|
| Recommendation panel | `<div class="pick-grid">` (~line 227) | Hard-coded text. Keep it in step with the write-up summary. |
| Dollar-to-pound rate | `var FX = 1.34;` (~line 382) | Used for every $ → £ conversion on the page. |
| All-in extras (per person) | `var EXTRA_LO = 1312, EXTRA_HI = 2317;` (~line 383) | Flights £600–1,000, tips £225–330, visa £37, insurance £150–350, jabs £100–200, kit £200–400. |
| Operators | `var ops = [ … ]` (~line 387) | One object per operator. See the field list below. |
| Routes and altitude profiles | `var routes = { … }` (~line 499) | `stops: [day, metres, name, flag]`; flag `'h'` = acclimatisation hike, `'t'` = summit. `odds: [low, high]` percent. |
| Months | `var months = [ … ]` (~line 589) | `rate` is out of 5 (1.5, 2.5, 4.5 display as ranges); `rain` is mm; `moon` is the 2027 full-moon summit night. |
| Calculator constants | `['Insurance to 6,000 m', 250]`, `['Malaria tablets and jabs', 150]` (~line 642); flight options (~line 316); tips slider (~line 323) | |
| To-do list and operator questions | `var todos=[…]`, `var qs=[…]` (~lines 660, 677) | |
| "Researched" date | `<footer>` (~line 374) and `write-up.html` byline | Update both whenever figures change. |

**Operator object fields**

- `id`: unique slug.
- `rank`: order under "Shortlist first". 1–5 are the shortlist.
- `pick`: shortlist badge text. Omit it for non-shortlisted operators.
- `tier`: `'local'` | `'mid'` | `'premium'`.
- `base`, `name`, `trip`, `url`.
- `days`: days on the mountain, or `null` if unknown.
- `lo` / `hi`: price **each, sharing**, in the currency given by `cur` (`'gbp'` or `'usd'`).
- `share`: how it's priced (for example `'Group, twin share'`).
- `supp`: single supplement in £, or `null`. `suppTxt` is the label shown on the card.
- `f`: safety features, each `1` (stated), `'extra'` (costs extra) or `null` (not stated, shown with "?"). Keys: `oxygen`, `checks`, `chamber`, `toilet`, `kpap`.
- `note`, and `warn` (optional amber warning).

**`write-up.html`** is generated from a Claude doc. For small fixes, edit the HTML directly. For a large rewrite, ask Claude (Cowork) to update the doc and regenerate this file. Figures that appear on **both** pages must match:

- the headline totals (£7,400–9,400 for two with Kandoo; £3,700–7,200 each overall);
- the shortlist prices;
- the 2027 full-moon dates;
- the yellow fever and insurance notes.

**Figures most likely to go stale**

1. Kilimanjaro park fees. No tariff after 30 June 2026 had been published at research time, so a 2027 rise is possible.
2. Operator prices, especially sale prices (Jagged Globe shown at £2,895, reduced from £3,895; G Adventures $3,899, usually $5,199).
3. Flight prices.

**To publish an update**

```bash
cd "/Users/mpc93/Documents/Claude/Projects/Personal / Health/kilimanjaro-2027-site"
git add -A && git status        # check nothing unexpected is staged
git commit -m "Update prices (<month year>)"
git push
```

Pages rebuilds automatically in about a minute.

---

## 8. Optional extras (only if Matthew asks)

- **Custom domain:** add a `CNAME` file containing the domain, then `gh api -X PUT repos/clemmo93/kilimanjaro-2027/pages -f cname=<domain> -F https_enforced=true -f "source[branch]=main" -f "source[path]=/"`. Also add a DNS record at the registrar.
- **Allow search indexing:** change `noindex, nofollow` to `index, follow` in both files.
- **Social preview image:** add a 1200×630 `og-image.png` and a `<meta property="og:image" content="https://clemmo93.github.io/kilimanjaro-2027/og-image.png">` tag. Open Graph images need an absolute URL.
- **Share with Dad:** send him `https://clemmo93.github.io/kilimanjaro-2027/`. No GitHub account is needed to view it.

---

## 9. Paste-in prompt for Claude Code

> Read `HANDOVER.md` in this folder and publish the site to GitHub Pages exactly as it describes. Before running anything, confirm the four decisions in §1 with me (GitHub account, repo name, public repo, keep noindex). Follow the safety rules in §2, especially: only run git inside this folder, and show me `git status` before the first commit. After publishing, run the checks in §5.1 and give me the live URL.
