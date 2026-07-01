# PoEBane scripting — best practices

## Do

- **Read the typed API first.** [`scripting/api/poebane.d.ts`](scripting/api/poebane.d.ts) is
  generated from the runtime bindings and is authoritative. Look up the exact signature there
  instead of guessing.
- **Guard live reads with `PlayerValid()`.** `World.Player` is `Entity | null` and most state
  is only meaningful in-game. Check `PlayerValid()` (and null-check `World.Player`) first.
- **Keep the tick body cheap.** Your script is evaluated on the game loop. Do the minimum per
  tick: read the few fields you need, branch, act. Avoid per-tick allocation and heavy loops.
- **Prefer the accessors.** Read entities/flasks/inventory/UI through the `World.*` accessors —
  they read live native memory correctly. Don't re-derive addresses or offsets yourself.
- **Prefer the high-level toggles.** For stashing/farming/following/pickup, use
  `SetAutoStash` / `SetAutoFarm` / `SetAutoFollow` / `SetAutoPickup` rather than re-implementing
  that automation from primitives.
- **Log at the right level.** `Trace`/`Debug` for detail, `Info` for normal events, `Warn`/
  `Error` for problems. `console.log(...)` works too.

## Don't

- **Don't snapshot the whole world to JSON on the hot loop.** The `World.*` accessors are live
  bindings over native memory. Reading a field is cheap; serializing large accessor graphs (or
  `JSON.stringify`-ing them) every tick is a large allocation + GC hit. Read the specific
  fields you need.
- **Don't block the game loop.** Long sleeps, busy-waits, or synchronous input smoothing stall
  automation. Keep steering instant; do discrete actions and return.
- **Don't hardcode offsets, addresses, or metadata paths from memory.** Use the accessors and
  the `Entity.Path` / `Entity.Metadata` fields. Addresses are per-run.
- **Don't invent APIs.** If a global, accessor, method, or field is not in `poebane.d.ts`, it
  does not exist. The `apiHash` in that file must match the running build's
  `POEBANE_SCRIPT_API_HASH` — a mismatch means the docs are stale, so re-check.
- **Don't assume the reactive-rule layer is complete.** `Group` / `OnTimer` / `OnHotkey` return
  context objects; confirm in the `.d.ts` that the action side you want is wired before relying
  on it. The imperative surface (reads + `PressKey`/`SetAuto*`/`DisplayText`/logging) is stable.
