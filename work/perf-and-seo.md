# Plan · performance + share metadata pre-ship

**Date:** 2026-04-20
**Status:** executed (this commit), minus the OG image (deferred — no visual yet).
**Scope:** `index.html`, `style.css` (deleted), new `fonts/` directory.
**Goal:** fastest possible first-paint for first-time visitors; clean previews when shared on X / LinkedIn / Slack; clean signals to search crawlers.

---

## Problems being addressed

### 1 — Visible FOUT on load (font swap)

Symptom: text renders briefly in the fallback serif (Georgia / Iowan Old Style), then jumps when Alegreya finishes downloading from `fonts.gstatic.com`. The jump is a layout reflow because Alegreya's metrics (x-height, glyph widths) differ from the fallback.

Cause: Google Fonts CSS uses `font-display: swap`, which intentionally trades a flash for "no invisible text." With matched fallback metrics this would be invisible; without them, it's a visible jolt.

Cost: ugly, hurts Cumulative Layout Shift (a Core Web Vitals signal Google ranks on), and adds two third-party DNS + TLS handshakes (`fonts.googleapis.com` for the CSS, `fonts.gstatic.com` for the binaries) before the first font byte arrives.

### 2 — External `style.css` adds a round trip

Symptom: HTML downloads, browser parses `<head>`, sees `<link rel="stylesheet">`, blocks render until that CSS arrives.

Why this matters here specifically: a one-page essay site has no repeat-visit cache benefit to amortise the round trip against. Almost every visitor is a first-time visitor reading once.

### 3 — No canonical URL, no structured data, no share image

Symptom: when shared on X / LinkedIn / Slack, the preview card has no image. Search crawlers have to infer authorship and date from the HTML rather than reading explicit signals. There's no canonical URL set, so any future redirect or alternate URL would dilute search authority.

---

## Decisions (with reasoning)

### Decision 1 — Self-host Alegreya, don't proxy Google Fonts

Tier 3 from the conversation. Tradeoffs considered:

- **Tier 1 — `display=swap` → `display=optional`** (1-line change). Kills the jump for fast connections, but first-time visitors on slow connections see the fallback serif for the entire pageview. For an essay where the typography *is* part of the message, that's the wrong default.
- **Tier 2 — self-host without metric-matched fallback.** Removes the DNS + TLS hops and the third-party dependency, but keeps a small residual jump.
- **Tier 3 — self-host + preload + metric-matched fallback `@font-face`.** Eliminates the jump (fallback occupies the same space as Alegreya), removes third-party calls, lets browsers fetch fonts in parallel with HTML parsing.

Tier 3 chosen. The marginal extra effort is small (a one-time `@font-face` block with Capsize-derived metrics) and the result is genuinely "no jump, ever."

Side benefits:
- **Privacy / GDPR.** Munich district court 2022 ruled embedding Google Fonts illegal under GDPR because it leaks visitor IPs to Google. Self-hosting moots this.
- **Reliability.** Page works when Google Fonts is slow / blocked (corporate firewalls, parts of the EU, China, Russia).
- **Predictability.** No risk of Google changing what `Alegreya:wght@400` resolves to in a future font update.

Cost: ~4 woff2 files × ~50 KB = ~200 KB total, served from GitHub Pages, cached on first visit.

### Decision 2 — Inline `style.css` into `<head>`

For a single-page site with ~5 KB of CSS:

- Cache benefit of an external file requires repeat visits or multi-page reuse — neither applies.
- Inline CSS arrives in the first packet alongside the HTML; no extra round trip blocks first paint.
- 5 KB extra in the HTML is negligible (HTML compresses well).

If a second page is ever added, extraction back to an external file is trivial. Until then, inlining is strictly faster.

### Decision 3 — Add canonical URL, JSON-LD `Article` schema, and OG image

Three independent metadata additions:

- **Canonical URL** (`<link rel="canonical">`). Cheap insurance — tells crawlers and aggregators which URL is authoritative if the page is ever embedded, archived, or proxied. Just a meta tag pointing at `https://zij.github.io/scarcityofnovelty/`.
- **JSON-LD `Article` schema.** Lets Google show this as a proper article in search results, with author and date pulled from explicit fields rather than inferred. ~12 lines of JSON in a `<script type="application/ld+json">` block.
- **OG image.** Currently no image appears when the URL is shared on X / LinkedIn / Slack. Plan: a 1200×630 PNG, generated as a clean title card (title set in Alegreya italic on the cream paper background, byline below). Stored at `/og.png`, referenced from `<meta property="og:image">` and the JSON-LD `image` field.

---

## Order of operations

1. ✓ Downloaded four latin-subset woff2 files from the Google Fonts CDN (fetched with a Chrome User-Agent to force woff2 — the default Safari UA returns woff for Alegreya). Placed in `/fonts/`. Total ~92 KB, under half the 200 KB estimate.
2. ✓ Computed metric overrides with fontkit reading the actual woff2 files. Results:
   - `Alegreya` vs `Georgia`: size-adjust 92.81%, ascent-override 109.48%, descent-override 37.17%.
   - `Alegreya Sans` vs `Helvetica Neue`: size-adjust 93.17%, ascent-override 96.60%, descent-override 32.20%.
3. ✓ Added `@font-face` declarations for the four web fonts plus four fallback declarations (one per style, for both families).
4. ✓ Added `<link rel="preload" as="font" type="font/woff2" crossorigin>` for the two roman 400 weights.
5. ✓ Inlined `style.css` into a `<style>` block in `<head>`. Deleted `style.css`.
6. ✓ Removed the Google Fonts `<link>` and both `preconnect` hints from `<head>`.
7. ✗ Deferred — OG image skipped per user direction ("no visual for now"). Twitter card downgraded from `summary_large_image` to `summary` since there's no image to show.
8. ✓ Added `<link rel="canonical">`, `<meta property="og:url">`, `<meta property="article:published_time">`, `<meta name="theme-color">` (light + dark variants), and the JSON-LD `Article` block. `og:image` intentionally omitted.

---

## Verification

After implementation, confirm:

- **No FOUT.** Open `index.html` with the Network throttle set to "Slow 3G" in DevTools. The fallback should occupy the same space as Alegreya — no visible reflow when the font swaps in.
- **No third-party requests.** DevTools → Network → no requests to `fonts.googleapis.com` or `fonts.gstatic.com`.
- **Lighthouse Performance ≥ 95**, CLS = 0, LCP < 1.5 s on a fast connection. Run via Chrome DevTools.
- **Share previews.** Paste the deployed URL into X's card validator, LinkedIn's post composer, and Slack — image should appear.
- **JSON-LD validates.** Run `og.png` page through Google's Rich Results Test — `Article` schema should be detected with no errors.
