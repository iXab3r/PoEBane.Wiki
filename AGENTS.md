# PoEBane scripting — agent guide

You are helping a user write **PoEBane** automation scripts for *Path of Exile 2*. This
repository is your reference for the scripting API, conventions, and examples.

## Priority

1. Follow the user's own project instructions first.
2. Use **this repository** for PoEBane scripting concepts, conventions, best-practices, and examples.
3. Use the generated **`poebane.d.ts`** for exact signatures — it is authoritative and always current
   (see the workflow below). Prefer it and these docs over your own memory: the API surface is
   generated and versioned; your training data is not.
4. Use the live **PoEBane MCP** only for focused runtime lookups (current world state, window
   layout) when it is running — not as a substitute for the docs.

## Workflow

1. Read [`best-practices.md`](best-practices.md) and
   [`scripting/getting-started.md`](scripting/getting-started.md).
2. **Look up the exact API in `poebane.d.ts` before proposing code** — globals, `World`, `Input`, the
   typed accessors and interfaces. In the app, PoeBane mounts this file live (the running build's own
   surface — always current; you are told its path and that it is authoritative). Offline, read it
   from the build at `src/PoeBane/ui/generated/poebane.d.ts`. If a member is not in it, it does not
   exist — do not invent it. See [`scripting/api/README.md`](scripting/api/README.md).
3. Match the style of the user's existing script. Write the smallest change that works.
4. Guard live-state reads with `PlayerValid()`; keep per-tick work cheap (see best-practices).

## What's here

- [`best-practices.md`](best-practices.md) — the do/don'ts that keep scripts correct and cheap.
- [`scripting/getting-started.md`](scripting/getting-started.md) — the scripting model:
  reading `World`, driving `Input`, the `SetAuto*` toggles, logging, and the (WIP) rule layer.
- [`scripting/api/README.md`](scripting/api/README.md) — where the **authoritative** generated API
  reference (`poebane.d.ts` + `poebane.script-api.json`) lives and how to read it. The reference
  itself is **not committed here** — the running build emits it, so it can never drift.

## The model in one paragraph

A PoEBane script is TypeScript/JS run against a live PoE2 process. You **read** live state from
the `World` global (`Player`, `Entities`, `Flasks`, `Inventory`, `Area`, `Camera`, `Party`,
`Ui`, `Stash`, `ItemsOnGround`, `Osd`) — always after checking `PlayerValid()`. You **act**
through `Input` and the globals `PressKey` / `HoldKey` / `ReleaseKey` / `IsKeyPressed`, project
world→screen with `World.W2S` / `W2C`, and draw overlay text with `DisplayText`. High-level
automation is toggled with `SetAutoStash` / `SetAutoFarm` / `SetAutoFollow` / `SetAutoPickup`.
Reactive rules (`Group`, `OnTimer`, `OnHotkey`) are an emerging layer — check `poebane.d.ts` for
what is wired before relying on them. Log with `Log` / `Info` / `Warn` / `Error` or
`console.log(...)`.
