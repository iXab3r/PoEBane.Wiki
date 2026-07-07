---
title: Auto farm and pickup
description: Turn on the built-in farming and loot automations instead of re-implementing them from input primitives
published: true
date: 2026-07-07T00:00:00.000Z
tags: PoEBane, Path of Exile 2, scripting, automation, recipe
editor: markdown
dateCreated: 2026-07-07T00:00:00.000Z
order: 32
---

# Recipe — auto farm and pickup

For clearing and looting, prefer the high-level toggles over hand-rolled movement. Each is a
simple on/off switch handled inside PoEBane, so your script stays tiny.

```ts
SetAutoFarm(true);      // clear the current area
SetAutoPickup(true);    // grab drops
SetAutoStash(true);     // stash when full / at a stash
// SetAutoFollow(true); // follow the party leader instead of farming

Info("auto farm + pickup enabled");
```

When to use which:

- `SetAutoFarm` — kill and clear the current area.
- `SetAutoPickup` — pick up ground items (pair it with farming).
- `SetAutoStash` — offload inventory into the stash.
- `SetAutoFollow` — follow instead of farm (party play). Don't combine it with `SetAutoFarm`.

You flip the switches and let the built-in automation drive. Add your own per-tick logic (flasks,
alerts) alongside — just keep it cheap, since it runs on the game loop.
