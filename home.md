---
title: PoEBane scripting
description: Write TypeScript automation for a live Path of Exile 2 process — read World, drive input, toggle the built-in automations
published: true
date: 2026-07-07T00:00:00.000Z
tags: PoEBane, Path of Exile 2, scripting, TypeScript, automation, ai-translated
editor: markdown
dateCreated: 2026-07-07T00:00:00.000Z
order: 1
---

# PoEBane scripting

PoEBane scripts are **TypeScript / JavaScript** evaluated against a **live** *Path of Exile 2*
process. A script reads game state from the `World` global, drives input, and can flip on built-in
automations — all from a few lines you edit in the Studio editor.

## The model in one paragraph

You **read** live state from `World` (`Player`, `Entities`, `Flasks`, `Inventory`, `Area`,
`Camera`, `Party`, `Ui`, `Stash`, `ItemsOnGround`, `Osd`) — always after `PlayerValid()`. You
**act** through `PressKey` / `HoldKey` / `ReleaseKey`, use belt slots via `World.Flasks.Use(i)`,
and project world→screen with `World.W2S`. High-level automation is toggled with `SetAutoStash` /
`SetAutoFarm` / `SetAutoFollow` / `SetAutoPickup`. You draw overlay text with `DisplayText` and log
with `Log` / `Info` / `Warn` / `Error`.

## Where to start

| You want to… | Read |
| --- | --- |
| write your first script | [Getting started](scripting/getting-started.md) |
| avoid the common traps | [Best practices](best-practices.md) |
| copy a working example | [Recipes](recipes/low-life-flask.md) |

## Authoritative API

The exact signatures live in the generated `poebane.d.ts`, **not** in this wiki — the running build
emits its own copy so it can never drift. If a member is not in `poebane.d.ts`, it does not exist;
do not rely on it. These pages carry the concepts and examples; the `.d.ts` is the source of truth
for exact names and types.


## Waits and timers

[Execution and waiting primitives](scripting/execution-and-waits.md) explains Sleep without await, independent rules, WaitUntil and Stop. [Waiting recipes](recipes/waiting.md) covers sequential processing and following a leader. These POE-81 features are unreleased; check your installed declarations.
