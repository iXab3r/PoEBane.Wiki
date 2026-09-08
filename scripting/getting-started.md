---
title: Getting started
description: The PoEBane scripting model — reading World, projection, driving input, the SetAuto toggles, overlay and logging
published: true
date: 2026-07-07T00:00:00.000Z
tags: PoEBane, Path of Exile 2, scripting, getting started, ai-translated
editor: markdown
dateCreated: 2026-07-07T00:00:00.000Z
order: 10
---

# Getting started with PoEBane scripts

A PoEBane script is **TypeScript/JavaScript** run against a **live** *Path of Exile 2* process.
You edit it in the Studio's Monaco editor (typed against `poebane.d.ts`) and it is evaluated by
the embedded runtime. Everything below is grounded in the generated `poebane.d.ts` — the
authoritative API surface emitted by the running build. This wiki carries the prose; the `.d.ts`
carries the exact signatures.

## 1. Reading game state

`World` (a `Poe2WorldState`) is the entry point for live state:

| Accessor | What it exposes |
| --- | --- |
| `World.Player` | the player `Entity` (or `null` when not in-game) |
| `World.Entities` | nearby entities (monsters, NPCs, objects) |
| `World.Flasks` | the 5-slot belt (life/mana flasks + charms) |
| `World.Inventory` | inventory items |
| `World.ItemsOnGround` | ground item labels |
| `World.Area` | current area / zone |
| `World.Camera` | camera + projection |
| `World.Party` | party members and invites |
| `World.Ui` | UI panels (stash, inventory, trade, ground labels) |
| `World.Stash` | stash panel state |
| `World.Osd` | on-screen-display drawing |

An `Entity` exposes `IsValid`, `Id`, `Type`, `Path`, `Metadata`, `PlayerName`, `IsAlive`,
`IsTargetable`, `WorldPosition` (`Vector3`), `GridPosition` (`Vector2`), and vitals
(`HP` / `ES` / `Mana` / `Ward`, each a `Vital` with `Current` / `Max` / `Percent`).

**Always guard reads:** call `PlayerValid()` and null-check `World.Player` first.

## 2. Projection — world to screen

- `World.W2S(worldPos)` → `Vector2` screen point; `World.TryW2S(worldPos)` → `Vector2 | null`.
- `World.W2C(worldPos)` / `World.TryW2C(worldPos)` → client-space point.

Use these to place overlay drawing at a world position.

## 3. Driving input

- `PressKey(key)`, `HoldKey(key)`, `ReleaseKey(key)`, `IsKeyPressed(key)` — `key` is a member of
  the `Key` enum (see `poebane.d.ts`).
- `Input` (a `Poe2InputController`) for lower-level control.
- `World.Flasks.Use(index)` uses a belt slot directly (`0`/`1` = life/mana, `2..4` = charms).

## 4. High-level automation toggles

Prefer these over re-implementing behavior from primitives:

```ts
SetAutoStash(true);
SetAutoFarm(true);
SetAutoFollow(true);
SetAutoPickup(true);
```

## 5. Overlay and logging

- `DisplayText(text, worldPosition, color?)` draws text at a world position.
- `Log` / `Trace` / `Debug` / `Info` / `Warn` / `Error` and `console.log(...)` for logging.

## 6. Reactive rules (work in progress)

`Group(name)`, `OnTimer(interval_ms)`, and `OnHotkey(gesture, scope?)` return builder contexts,
and `EnableGroup` / `DisableGroup` toggle named groups. This reactive layer is still being
wired — **check `poebane.d.ts` for what the returned context actually supports** before relying
on it. The imperative surface above is stable today.

## Example — low-life life flask

Accurate against the current API (`PlayerValid`, `World.Player`, `Vital.Percent`,
`Poe2FlasksAccessor.HasItemAt` / `Use`, `Info`):

```ts
// Evaluated against the live game. Use belt slot 0 (life flask) when HP drops below 50%.
if (PlayerValid()) {
  const player = World.Player;            // Entity | null
  if (player) {
    const hp = player.HP;                 // Vital { Current, Max, Percent }
    if (hp.Percent < 50 && World.Flasks.HasItemAt(0)) {
      World.Flasks.Use(0);
      Info(`used life flask at ${hp.Percent.toFixed(0)}% HP`);
    }
  }
}
```

See the [recipes](../recipes/low-life-flask.md) for more copy-ready scripts, and
[best practices](../best-practices.md) before you ship one.


## Waits and timers

[Execution and waiting primitives](execution-and-waits.md) explains Sleep without await, independent rules, WaitUntil and Stop. [Waiting recipes](../recipes/waiting.md) covers sequential processing and following a leader. These POE-81 features are unreleased; check your installed declarations.
