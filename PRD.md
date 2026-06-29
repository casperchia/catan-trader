# PRD: Catan Trader

A single-screen, language-free mobile web app for proposing Catan trades visually.

## Problem Statement

I play Catan with friends who don't read English. When I want to propose a trade
("selling 1 sheep for 1 wheat", "buying 1 ore for 2 wood", "buying 2 wheat for 3
bricks"), I have no fast, unambiguous way to communicate the offer across the
language barrier. Speaking and writing don't work, and gesturing with physical
cards is slow and error-prone — especially for compound offers ("1 wood for
1 wheat AND 1 brick") or offers with alternatives ("...for X OR Y").

## Solution

A web app I load on my phone and lay flat on the table between me and the person
I want to trade with. The screen is split into two halves: the bottom half (facing
me) shows what **I give**, and the top half (facing them, rotated 180° so it reads
correctly to them) shows what **they give**. Resources are shown purely as icons —
no words — so anyone can understand the offer at a glance. A double-headed arrow on
the dividing line communicates that this is a swap. I build offers by tapping, and
can express compound offers (multiple resources in one pile) and alternatives
(multiple piles separated by a vertical line, meaning OR).

The app is a single self-contained HTML file using vanilla JavaScript and inline
SVG, works fully offline, and can be added to my phone's home screen.

## User Stories

1. As a player, I want the screen split into a top and bottom half, so that the offer reads correctly to both me and the person across the table.
2. As a player, I want the bottom half to represent what I give, so that my side of the trade is closest to me.
3. As a player, I want the top half to represent what the other person gives, so that their side is closest to them.
4. As a player, I want the top half rotated 180°, so that icons and arrows read right-side-up to the person across the table.
5. As a player, I want a double-headed swap arrow on the dividing line, so that the screen communicates "this is an exchange" without words.
6. As a player, I want to tap a half to add resources, so that I can build an offer quickly with one hand.
7. As a player, I want a centered popup picker showing the 6 resource tiles, so that I can choose what to add.
8. As a player, I want the picker to always face me regardless of which half I'm editing, so that I (the operator) can always read it.
9. As a player, I want six choices — wood, brick, sheep, wheat, ore, and a wildcard "?" — so that I can also offer "any one card".
10. As a player, I want to add a wildcard to a trade, so that I can express offers like "1 sheep for any 1 card".
11. As a player, I want each picker tile to show its current count in the pile being edited, so that I can see what I've added.
12. As a player, I want tapping a tile to add one (+1), so that increasing a count is fast.
13. As a player, I want a small "−" on each tile to remove one (−1), so that I can correct mistakes without long-pressing.
14. As a player, I want to tap outside the picker to close it, so that dismissing is effortless.
15. As a player, I want counts shown as repeated icons (three bricks = three brick icons), so that the quantity is unambiguous to someone who can't read numbers or English.
16. As a player, I want repeated icons to wrap when there are many, so that the layout stays readable.
17. As a player, I want to put multiple different resources in one pile (AND), so that I can offer compound deals like "1 wheat and 1 brick".
18. As a player, I want to create multiple piles on a side, so that I can offer alternatives.
19. As a player, I want piles on a side separated by a vertical line, so that the "OR" relationship between alternatives is visually clear without words.
20. As a player, I want to tap empty space in a half to start a new alternative pile, so that adding an option needs no extra buttons.
21. As a player, I want to tap an existing pile to reopen the picker for that pile, so that I can edit it.
22. As a player, I want emptying a pile (removing its last resource) to delete the pile, so that I don't leave empty boxes behind.
23. As a player, I want both halves to support alternatives, so that either side of the trade can have OR options.
24. As a player, I want a clear/reset button on the center divider, so that I can wipe both halves and start a new trade in one tap.
25. As a player, I want resources drawn as inline SVG icons in Catan-inspired colors, so that they're recognizable and tie color to resource.
26. As a player, I want a warm parchment/wood theme, so that the app feels familiar to Catan players.
27. As a player, I want the app to work fully offline, so that it loads at a game table with poor or no wifi.
28. As a player, I want the app to load instantly with no build step, so that it's reliable and simple.
29. As a player, I want to add the app to my home screen, so that it launches like a native app.
30. As a player, I want the screen to stay awake while the app is open, so that it doesn't sleep mid-game.
31. As a player, I want subtle tap feedback (haptics and press states), so that my taps feel confident on mobile.
32. As a player, I want to leave a side empty if I choose, so that I'm not forced to fill both halves.
33. As a player, I want the app usable in portrait, so that the top/bottom split stays consistent when the phone lies flat.

## Implementation Decisions

**Architecture**
- Single self-contained `.html` file: vanilla JavaScript, inline SVG, inline CSS. No React, no CDN dependencies, no build step. Works fully offline.
- Hosted on GitHub Pages with a `manifest.webmanifest` and `service-worker.js` for offline caching and Add-to-Home-Screen.

**Modules**
1. **Trade model** (deep, pure, no DOM) — single source of truth for trade state.
   - State shape: a trade has two sides, `give` and `want`; each side is an ordered list of piles; each pile is a map of `resourceId → count` (count ≥ 1).
   - Interface: `addResource(side, pileIndex, resourceId)` (+1), `decrementResource(side, pileIndex, resourceId)` (−1, removing the key at 0 and the pile when it becomes empty), `addPile(side)` (returns new pile index), `removePile(side, pileIndex)`, `clearTrade()`, and serialize/deserialize helpers.
   - No validation: empty sides are allowed; no maximum counts.
2. **Resource catalog** (static data) — the 6 resources (wood, brick, sheep, wheat, ore, wildcard) with id, display color, and inline SVG markup. Catan-inspired palette: wood=green, brick=red-orange, sheep=light green, wheat=gold, ore=grey, wildcard=neutral.
3. **Renderer** (DOM) — renders the model: two halves (top rotated 180°), piles separated by vertical dividers, resources as repeated icons (wrapping), center divider with double-headed swap arrows and clear button.
4. **Picker controller** (DOM + model) — centered popup scoped to one pile, always oriented facing the operator; shows 6 tiles with current counts; tap = +1, "−" = −1; tap-outside closes; opening targets either an existing pile (edit) or a newly created pile (when empty space tapped).
5. **App shell** — event wiring between DOM and model, re-render on change, Screen Wake Lock acquisition, haptic/visual tap feedback.
6. **PWA infra** — `manifest.webmanifest`, `service-worker.js` (cache-first), and an app icon (SVG/PNG).

**Key interactions**
- Tap empty area of a half → create a new pile and open picker on it.
- Tap an existing pile → open picker scoped to that pile.
- Inside picker: tile tap (+1), tile "−" (−1), tap outside to close; pile auto-deletes when emptied.
- Clear button on divider → `clearTrade()`.

**Orientation**
- Portrait layout; top half visually rotated 180°. No hard orientation lock (full lock only reliable as installed PWA).

## Testing Decisions

- No automated tests for v1 (developer decision).
- Note for future test work: the **Trade model** is the deep, DOM-free module and the natural target for unit tests — its external behavior (add/decrement, pile creation/deletion, OR alternatives, clear, serialize/deserialize) can be tested without any DOM. Good tests would assert observable state transitions through the public interface only, not internal representation. DOM-bound modules (renderer, picker) are deferred.

## Out of Scope

- Two-person simultaneous editing — the operator builds the whole trade; the partner only reads.
- Trade acceptance/negotiation flow, history, or logging of past trades.
- Persistence across reloads (localStorage) — an accidental reload loses the in-progress offer.
- Portrait orientation hard-lock.
- Maximum count limits or trade validation.
- Real Catan card art / licensed imagery (using original inline SVG instead).
- React or any framework; any build/bundling step.
- Multi-language text UI (the app is intentionally text-free).

## Further Notes

- Files to be produced: `index.html`, `manifest.webmanifest`, `service-worker.js`, and an app icon.
- The wildcard "?" participates like any other resource in the model.
- Example offers this must support: "1 sheep → 1 wheat"; "2 wood → 1 ore"; "3 brick → 2 wheat"; "1 sheep → any 1 card" (wildcard); "1 wood → {1 wheat + 1 brick} OR {1 wheat + 1 sheep}" (compound + alternatives).
- GitHub issue submission intentionally skipped for now.
