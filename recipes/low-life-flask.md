---
title: Low-life flask
description: Use the life and mana belt flasks automatically when the player's vitals drop below a threshold
published: true
date: 2026-07-07T00:00:00.000Z
tags: PoEBane, Path of Exile 2, scripting, flasks, recipe
editor: markdown
dateCreated: 2026-07-07T00:00:00.000Z
order: 31
---

# Recipe — low-life flask

Press the life flask when HP drops, and the mana flask when mana drops. Belt slots are
`0` = life, `1` = mana, `2..4` = charms. `World.Flasks.HasItemAt(i)` checks a slot has a usable
flask; `World.Flasks.Use(i)` presses it.

```ts
// Life flask below 50% HP, mana flask below 25% mana.
if (PlayerValid()) {
  const p = World.Player;                 // Entity | null
  if (p) {
    if (p.HP.Percent < 50 && World.Flasks.HasItemAt(0)) {
      World.Flasks.Use(0);
      Info(`life flask @ ${p.HP.Percent.toFixed(0)}% HP`);
    }
    if (p.Mana.Percent < 25 && World.Flasks.HasItemAt(1)) {
      World.Flasks.Use(1);
    }
  }
}
```

Notes:

- `HP` / `Mana` are `Vital` objects — read `.Percent` (0–100), not the raw `.Current`.
- `HasItemAt(i)` guards against pressing an empty or uncharged slot.
- Keep this in the tick body: a couple of cheap reads plus a conditional press.
- Charms sit in slots `2..4` — the same `HasItemAt` / `Use` pattern applies.
