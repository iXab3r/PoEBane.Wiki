# PoEBane scripting — agent guide

You are helping a user write **PoEBane** automation scripts for *Path of Exile 2*. This
repository is your reference for the scripting API, conventions, and examples.

## Priority

1. Follow the user's own project instructions first.
2. Use **this repository** for all PoEBane script APIs, accessors, and examples.
3. Prefer these docs and the typed API (`scripting/api/poebane.d.ts`) over your own memory.
   The API surface is generated and versioned — your training data is not.
4. Use the live **PoEBane MCP** only for focused runtime lookups (current world state, window
   layout) when it is running — not as a substitute for the docs.

## Workflow

1. Read [`best-practices.md`](best-practices.md) and
   [`scripting/getting-started.md`](scripting/getting-started.md).
2. **Search [`scripting/api/poebane.d.ts`](scripting/api/poebane.d.ts) for the exact API**
   (globals, `World`, `Input`, the typed accessors and interfaces) *before* proposing code.
   If it is not in that file, it does not exist — do not invent it.
3. Match the style of the user's existing script. Write the smallest change that works.
4. Guard live-state reads with `PlayerValid()`; keep per-tick work cheap (see best-practices).

## What's here

- [`best-practices.md`](best-practices.md) — the do/don'ts that keep scripts correct and cheap.
- [`scripting/getting-started.md`](scripting/getting-started.md) — the scripting model:
  reading `World`, driving `Input`, the `SetAuto*` toggles, logging, and the (WIP) rule layer.
- [`scripting/api/poebane.d.ts`](scripting/api/poebane.d.ts) — the **authoritative** generated
  typed API. Every global, accessor, and interface, with inline doc comments.
- [`scripting/api/README.md`](scripting/api/README.md) — how to read the API reference.

## The model in one paragraph

A PoEBane script is TypeScript/JS run against a live PoE2 process. You **read** live state from
the `World` global (`Player`, `Entities`, `Flasks`, `Inventory`, `Area`, `Camera`, `Party`,
`Ui`, `Stash`, `ItemsOnGround`, `Osd`) — always after checking `PlayerValid()`. You **act**
through `Input` and the globals `PressKey` / `HoldKey` / `ReleaseKey` / `IsKeyPressed`, project
world→screen with `World.W2S` / `W2C`, and draw overlay text with `DisplayText`. High-level
automation is toggled with `SetAutoStash` / `SetAutoFarm` / `SetAutoFollow` / `SetAutoPickup`.
Reactive rules (`Group`, `OnTimer`, `OnHotkey`) are an emerging layer — check the `.d.ts` for
what is wired before relying on them. Log with `Log` / `Info` / `Warn` / `Error` or
`console.log(...)`.
