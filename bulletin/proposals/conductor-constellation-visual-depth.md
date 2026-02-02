# CONDUCTOR DIRECTIVE — Constellation Visual Depth

**Date**: 2026-02-02
**Status**: PROPOSAL — awaiting human approval
**Problem**: With 6 entries all dated within 2 days, the constellation has no visual separation. All nodes render at age 0–1 (same size, color, glow, tight center cluster). The prototype looked rich because it had 16 entries across ages 0–5.

---

## Root Causes

1. **Age algorithm is date-only** — entries from the same week all collapse into the same tier
2. **Positioning is age-driven** — `baseDistance = 80 + (age * 60)` means same-age nodes start at the same radius before jitter
3. **Only 6 entries** — not enough nodes for the constellation to feel full
4. **Particles are too subtle** — 2px at 0.3 opacity on a near-black background

---

## Proposed Changes

### 1. Rank-based visual tiers (instead of pure date-based age)

Currently `calculateAge()` maps date → age tier. When all dates are recent, everything is age 0.

**New approach**: Use the entry's **position in the sorted list** to assign a visual tier, ensuring spread even when dates are concentrated.

```
Tier 0: entries 1–2 (newest)
Tier 1: entries 3–4
Tier 2: entries 5–6
Tier 3: entries 7–10
Tier 4: entries 11–15
Tier 5: entries 16+
```

This guarantees the full visual gradient is always active regardless of date clustering. The tooltip still shows the real date — only the visual treatment changes.

### 2. Wider positioning spread

Change the scatter algorithm so nodes use more of the viewport even at similar ages:

- Increase base jitter from ±80/±60 to ±150/±120
- Use the golden angle (137.5°) for angular distribution instead of linear — this creates a natural spiral that avoids clumping
- Increase the minimum `baseDistance` so nodes aren't all near center

### 3. Bolder particles

- Increase count from 15 → 25
- Add size variation: mix of 2px and 3px particles
- Increase opacity range from 0.3 to 0.15–0.5 (varied per particle)
- Add a second slower drift speed tier (30s) for depth parallax effect

### 4. Connection lines respond to density

When nodes are clustered, the connection algorithm creates a tight web. Add a max-connections-per-node limit (e.g., 3) so the SVG layer doesn't become a dense blob in the center.

### 5. Richer JSON entries

Break the existing 6 entries into ~12–15 by decomposing the larger commits into individual features. Example: the rebrand commit (536a0b4) delivered the cabinet layout, amber palette, scanlines, typing effect, ghost cabinets, CRT glow, and responsive layout — each could be its own node.

---

## Implementation Assignment

**Design agent**: Items 3 (particles) and 4 (connection cap) — CSS and JS in `changelog/index.html`
**UX agent**: Items 1 (rank tiers) and 2 (positioning) — JS in `changelog/index.html`
**Conductor**: Item 5 (richer JSON) — `changelog.json`

---

## Not Changing

- The 6 age-tier CSS styles (sizes, colors, glows) — these are fine, they just need to be activated
- The grain overlay — working as intended
- The tooltip/badge system — working as intended
- The back link and page title — working as intended
