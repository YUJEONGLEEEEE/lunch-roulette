# Lunch Roulette · 오늘 뭐 먹지?

A spinning wheel that decides where to eat lunch. One HTML file — no build step, no dependencies, no backend. Open it by double-clicking, or drop it on any static host.

**Live demo:** https://yujeongleeeee.github.io/lunch-roulette/

## Features

- **Unlimited entries** — add restaurants one by one, or paste a comma-separated list to add several at once. Duplicates are rejected with a visual nudge.
- **Honest draw** — the winner is picked first with `crypto.getRandomValues` (uniform, rejection-sampled), then the target rotation is computed so that wedge stops under the pointer. The animation can never disagree with the outcome.
- **Spin again without this one** — drops the winner from the list and immediately re-spins, for when the group vetoes a result.
- **Persistence** — the list and the last five results are kept in `localStorage`, so the next day starts where you left off.
- **Shareable list** — the list is encoded into the URL hash (`#r=name|name|name`). Send the link and the wheel opens pre-filled.
- **Adaptive rendering** — canvas wedges scale to any entry count; label size and truncation are derived from wedge width, and labels flip to stay upright on the left half.
- **Weighting toggle** — with it on, the three most recent winners are damped to ×0.25 / ×0.5 / ×0.75, so yesterday's restaurant is less likely to come up again. Weights drive the wedge widths as well as the draw, so a slice's size always equals its real probability, and each entry shows that percentage in the list. One button turns the whole thing off for an even draw.
- **Six languages** — Korean, English, Japanese, Chinese, Spanish and German. The language is detected from the browser on first visit, switchable at any time, and remembered afterwards.
- **Installable** — a web app manifest and Apple touch icons let it be added to a phone's home screen and open full-screen, without a native shell.
- **Light and dark themes**, responsive down to phone width, keyboard support (Space to spin, Esc to close), and `prefers-reduced-motion` handling.

## How it works

The wheel is a single `<canvas>` redrawn each frame at device pixel ratio. A spin picks the winner index `w`, then solves for the final rotation:

```
rot = -(w + 0.5) * segment - jitter
```

where `segment = 2π / n` and `jitter` keeps the pointer from always landing dead center. The wheel eases to `rot + turns * 2π` on a quartic ease-out, and a short Web Audio click fires each time a wedge boundary crosses the pointer.

With weighting on, `segment` is no longer uniform: each wedge spans `2π · wᵢ / Σw`, and the draw samples from the same weight table, so the arc you see is the probability you get. The landing math was verified over 82,600 simulated spins across 2–60 entries and every weight arrangement — the wedge under the pointer matched the drawn winner every time — and a 400,000-spin run confirmed the observed win rates match the wedge widths.

Wedge colors come from a fixed eight-color palette rather than generated HSL steps, with per-wedge label color chosen by relative luminance so text stays legible on every fill.

## Localization

Every string lives in one `STRINGS` table keyed by language code, including the pluralized entry counter and the screen-reader labels, so adding a language means adding one object. The type stack is a CSS variable swapped by `:root[data-lang]`, which lets Japanese and Chinese render in Hiragino / PingFang instead of falling back through a Korean face. Append `#lang=de` (or `ja`, `zh`, `es`, `en`, `ko`) to the URL to force a language — useful for linking someone straight to their own.

## Running locally

```bash
open index.html
```

No server required — the page uses no network requests, external fonts, or third-party scripts.

## Deploying

Any static host works. For GitHub Pages: **Settings → Pages → Source: Deploy from a branch → `main` / `(root)`**.

## Data

There is no server and no analytics. Restaurant lists live only in the visitor's own browser; nothing is transmitted anywhere.

## Browser support

Any current browser. Uses Canvas 2D, Web Audio, `localStorage`, and `crypto.getRandomValues`; the page still works if audio or storage is unavailable.
