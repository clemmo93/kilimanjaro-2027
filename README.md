# Kilimanjaro for Dad

A two-page static site comparing ways for Matthew and his dad to climb Kilimanjaro together in 2027: 15 operators, 7 routes, month-by-month conditions, and a cost calculator for two people sharing.

**Researched 28 September 2026.** Prices, park fees and flight costs are a point-in-time snapshot, not a live feed.

## What's in here

| File | What it is |
|---|---|
| `index.html` | The interactive comparison. Self-contained: inline CSS and JS, no build step, no local assets. Only Google Fonts is loaded from outside. |
| `write-up.html` | The full written research with sources, generated from the original Claude doc. Static HTML, no JavaScript. |
| `HANDOVER.md` | How to publish this to GitHub Pages with Claude Code, how to verify it, and how to update the data. Read it before changing anything. |
| `.nojekyll` | Tells GitHub Pages to serve the files as they are, without running Jekyll. |

## Publishing

Follow `HANDOVER.md`. In short: push this folder to a new repo and set **Settings → Pages → Deploy from a branch → `main` / `(root)`**. The site then appears at `https://<you>.github.io/<repo>/`.

## Privacy

GitHub Pages sites are public on the internet even when the repository is private. Both pages carry `<meta name="robots" content="noindex, nofollow">`, so search engines are asked not to list them, but anyone with the link can open them. The content mentions a family member and general health preparation. It contains no medical records.

## Disclaimer

This is an unofficial personal planning page with no affiliation to any operator named. Success rates are mostly operator claims; the write-up marks which figures come from independent studies. Confirm every price with the operator before booking.
