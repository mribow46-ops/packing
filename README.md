# Packing List

A personal packing checklist. Replaces a Google Sheet that was painful to use on a phone.

## What it is

One self-contained `index.html` — no build step, no dependencies, no framework, no backend.
Deployed to GitHub Pages and saved to the home screen on iOS as a standalone web app.
State lives in the browser (`localStorage`), so the list is per-device and nothing is sent anywhere.

`index.html` also detects `window.storage` and uses it when present, which is only true when
the file is previewed inside Claude. On a normal web host it falls through to `localStorage`.
Keep that shim if you edit the storage layer.

## The data model

A tree of nodes, nested to any depth:

```js
{ id, text, st, mode, qty, open, kids: [] }
```

- Categories are just nodes that have `kids`. There is no separate category type.
- Loose top-level items (Laptop, Wallet, Keys, Tefillin) are nodes with no kids. That's intentional —
  they get taken nearly every trip and don't belong under a heading.
- `st` — 0 unsorted, 1 packed, 2 not taking this trip
- `mode` — `'check'` or `'count'`. Count items use `qty` instead of `st` to mean packed.
- Only leaf nodes carry real state. A parent's appearance is always **derived** from its
  leaf descendants via `view()`. Never store state on a parent.

## The interaction that matters most

This is the part to preserve if you refactor anything.

Packing is a process of elimination, not just ticking things off. For any given trip most items
in a category are *not* coming. So there are three states per item, and "not taking" has to count
as **dealt with** — otherwise categories never read as finished.

- Tap an item's box: unsorted → packed (check) → not taking (struck, dashed box) → unsorted
- Tap a **category's** box: *settle* it. Keeps whatever is already checked, marks everything
  still unsorted as "not taking". Tap again to reopen the whole category.

The result on screen: one obvious check mark surrounded by struck-through lines. That's how the
owner reads "here is what I'm actually taking from this category" at a glance. Preserve that
visual — a settled category should be visually quiet, with the check marks being the only
thing that draws the eye.

Counts exist for items where a number matters more than a check: socks, boxers, undershirts,
polos, books. Any quantity above zero means packed. Any item can switch between count and
checkbox from its `···` menu, and adding an item with a trailing `#` creates it as a count.

## Everything is editable from the page

The list is data, not code. Add, rename, delete, reorder, re-nest, switch item type — all from
the UI. The seed list in `SEED` only applies on very first load. Do not add features that
require editing the source to change the list contents.

## Design

Deliberately not a generic to-do app. Header is a luggage tag — dark slate with a perforated
bottom edge. Oswald condensed caps for the title and category names, Inter for body, JetBrains
Mono for counts and tallies. Teal for packed, grey dashed for not-taking. Keep it restrained;
the check marks are the only thing that should pop.

Quality floor: works down to phone width, keyboard focus visible, `prefers-reduced-motion`
respected, tap targets at least 38px.

## Possible next steps

Not built yet, roughly in order of usefulness:

- Drag an existing item under a different parent (currently you retype it)
- An "always take" flag so Reset pre-checks the givens
- Counts that hold both a target and a packed number ("3 of 5")
- Multiple saved lists — weekend vs. two-week vs. flying — switched from the header
- Offline support via a service worker

## Deploying

Push to `main`. GitHub Pages serves from the root. Editing the live version is just committing
a change to `index.html`; stored check marks survive because they live in the browser, not the file.
