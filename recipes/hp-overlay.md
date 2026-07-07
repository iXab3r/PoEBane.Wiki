---
title: HP overlay
description: Draw the player's HP and ES percentages as on-screen text anchored to the character
published: true
date: 2026-07-07T00:00:00.000Z
tags: PoEBane, Path of Exile 2, scripting, overlay, recipe
editor: markdown
dateCreated: 2026-07-07T00:00:00.000Z
order: 33
---

# Recipe — HP overlay

`DisplayText(text, worldPosition, color?)` draws overlay text at a world position — no manual
projection needed. Anchor it to the player for a floating vitals readout.

```ts
if (PlayerValid()) {
  const p = World.Player;
  if (p) {
    const hp = p.HP.Percent.toFixed(0);
    const es = p.ES.Percent.toFixed(0);
    DisplayText(`HP ${hp}%  ES ${es}%`, p.WorldPosition);
  }
}
```

Notes:

- `WorldPosition` is a `Vector3`; `DisplayText` takes the world point directly and places the
  label for you.
- Call it from the tick body so the label tracks the character each frame.
- Pass an optional `color` as the third argument to tint the text — check `poebane.d.ts` for the
  accepted color type.
