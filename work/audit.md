# Audit · The Scarcity of Novelty

**Date:** 2026-04-20
**Tool:** `/audit` (from impeccable v2.1.1)
**Scope:** `index.html`, `style.css`, `favicon.svg` at commit `ad28792`.
**Verdict:** 17 / 20 — Good. Address weak dimensions before shipping.

**Status (2026-04-20):** All P1 and required-P2 items addressed across `/typeset` (commit `1f36384`) and `/polish` (this commit). Remaining open items are P3 / optional.

---

## Score

| # | Dimension          | Score | Key finding                                                                     |
|---|--------------------|:-----:|---------------------------------------------------------------------------------|
| 1 | Accessibility      |  3/4  | `--ink-faint` fails WCAG AA contrast (~2.45:1) on meaningful section numerals. |
| 2 | Performance        |  3/4  | Five font weights loaded; only two are used after the lightening pass.         |
| 3 | Responsive design  |  4/4  | Fluid type and padding, mobile breakpoint, no overflow risks.                  |
| 4 | Theming            |  4/4  | Full CSS-variable token system, well-tuned dark mode, no hard-coded colours.   |
| 5 | Anti-patterns      |  3/4  | Clean overall, but EB Garamond is the "free classical serif" reflex pick.      |

Issue counts: **1 P1 · 4 P2 · 3 P3 · 0 P0**.

## Anti-pattern verdict

Would a reader assume this was AI-generated? **Probably not.** The page commits to a cohesive literary aesthetic with real typographic conventions: drop cap, asterism, small caps, old-style numerals, hanging punctuation. No gradient text, no glassmorphism, no card grids, no hero metrics, no side-stripe borders, no centred-everything slop.

The one soft tell is the typeface. EB Garamond is not on impeccable's reflex-reject list, but it is the obvious "free classical serif" default on Google Fonts. A more distinctive choice would move the page from "a nice-looking essay" to "this person clearly chose a specific typeface."

## Fix order

Execute in this order. Re-run `/audit` after each major pass.

### 1 — Typography pass (`/typeset`) — ✓ done (`1f36384`)

Addresses P1 numeral contrast, P2 single-family typography, P2 reflex font, P2 unused weights, P3 `::first-line` unpredictability. Roughly one round of work, because every item sits in the typography stack.

- **✓ [P2] Swap EB Garamond for a more distinctive body face.**
  Picked **Alegreya** instead of the suggested Vollkorn — designed explicitly for long-form literature (Tipos Latinos 2012 Best of Shows), more character in italic, full small-caps + old-style numerals.
- **✓ [P2] Introduce a second typeface for display / accents.**
  **Alegreya Sans** (matched-by-design pairing) used on dateline, byline name, section numerals.
- **✓ [P2] Prune loaded font weights.**
  Google Fonts URL trimmed from 5 files to 4 (only roman 400 and italic 400 of each family).
- **✓ [P3] Fix `.after-dropcap::first-line` small-caps.**
  Replaced with explicit `<span class="opening">It's only natural</span>` — deterministic across viewports.

### 2 — Contrast + focus polish (`/polish`) — ✓ done (this commit)

- **✓ [P1] Raise section-numeral contrast to WCAG AA.**
  Folded into the `/typeset` pass — `.numeral` promoted to `--ink-soft` (~5.5 : 1, well above the 4.5 : 1 minimum) when its font-family was reassigned.
- **✓ [P2] Strengthen link underline and add `:focus-visible`.**
  Underline modernised from `background-image` gradient hack to native `text-decoration` properties; colour bumped from `--rule` (~1.64 : 1) to `--ink-soft` (~5.5 : 1 light, ~6.7 : 1 dark) — comfortably above the 3 : 1 WCAG minimum for non-text UI. New `a:focus-visible` rule applies a 2px accent-coloured outline with 4px offset, replacing the browser-default blue ring that clashed with the warm palette. `prefers-reduced-motion` rule strengthened from links-only to global.

### 3 — Optional P3s — open

- **Gradient paper-texture on `body::before`.** Intentional, restrained, defensible as texture rather than effect. Some reviewers would flag it. Keep or swap to a flat `--paper-2` overlay.
  Location: `style.css:62-72`.
- **Port palette to OKLCH.** Purely a future-proofing change; zero visual impact if done carefully.
  Location: `style.css:6-30`.
- **Centred masthead vs left-aligned body.** Defensible as literary convention (chapter openers are centred in books). Run `/critique` for a UX-perspective read if unsure.

## Positive findings

Keep these patterns for future work on this repo:

- Real typographic craft — drop cap, asterism, old-style numerals, small-caps byline, hanging punctuation, language-tagged root, hyphenation with sensible limits, `text-wrap: balance` on display text, print stylesheet, reduced-motion stylesheet.
- Neutrals tinted toward the brand hue (warm brown-cream-brick), not pure grey. Single muted accent used rarely.
- Clean semantic HTML. `<main>`, `<article>`, `<header>`, `<section>`, `<footer>` all used correctly.
- Genuinely responsive; not shrunk.
- Dark mode is real — accent shifts from brick-red to dusty rose, not just inverted.
- No JavaScript. One external dependency (Google Fonts).

## Systemic patterns

- Contrast discipline is inconsistent at the lighter end of the palette. Rule for future edits: any colour touching *meaningful* text must clear 4.5 : 1 against its background. `--ink-faint` should be reserved for pure decoration.
- Single-family typography is a structural gap, not a nit. All hierarchy currently rides on italic vs roman and size.
