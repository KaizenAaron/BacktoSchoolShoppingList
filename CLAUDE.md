# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page, static back-to-school supply shopping list for "Sunshine State Academy," priced at a specific Target store. It is a self-contained interactive checklist a parent uses while shopping: check items off, watch collected/remaining totals update, and print or reset.

## Architecture

The entire app is **one file, `index.html`** — inline `<style>` and inline `<script>`, no build step, no dependencies, no framework, no package manager. Deployed as-is via GitHub Pages (`.nojekyll` disables Jekyll processing so the raw file is served).

To preview: open `index.html` directly in a browser, or `python3 -m http.server` and visit it. There is nothing to build, lint, or test.

### Data model lives in the DOM

There is no separate data source — each supply item **is** a `<tr>` in one of two `<tbody>`s (one `.listcard` section per list: "Elementary & Middle School" and "Preschool & VPK"). The item's line-item cost is stored on the row as `data-line="5.98"`, and each row has a checkbox with a unique id (`i0`, `i1`, …). To add, remove, or reprice an item, edit its `<tr>` markup directly.

The JS (`recompute`, `onCheck`, `resetAll`) reads the DOM as its source of truth: it sums `data-line` over checked rows, counts items, and updates the sticky summary bar and each section's "N / M collected" counter.

Checked state is persisted to `localStorage` under the key `sssa-supply-checklist-v1` as a JSON array of checked checkbox ids. `saveState()` runs on every check and on reset; `loadState()` restores checks on page load (before the initial `recompute()`), guarded by try/catch so private-mode / disabled storage degrades to in-memory only. This is per-browser/per-device — it does not sync across browsers or devices (that would require a backend the static site doesn't have). Bumping the row ids or the storage key version invalidates saved state.

### Values that must be kept consistent by hand

Several numbers are hardcoded and are **not** derived from the row data, so they can silently drift when items change:

- `var GRAND = 330.11` in the script — the grand total used for the "Remaining" figure and progress bar. Must equal the sum of every row's `data-line`.
- The two `Subtotal $…` labels in each `.lhead`, the `data-total` attributes, and the `.grand` block's total and per-list split (`$330.11`, `$134.31`, `$195.80`).
- The `sRemaining` / `sItems` initial text and the item count (`0 / 63`) shown before any JS runs.

When you add, remove, or reprice a row, recompute and update all of the above to match.

## Conventions

- Prices reflect a point-in-time capture at one Target store (Aventura, FL) including sale prices; substitutions where an exact size/count isn't sold are flagged with an italic `<span class="note">` inside the product cell.
- CSS is driven by custom properties in `:root` (navy/teal theme). A `@media print` block hides the summary bar and buttons for printing.
