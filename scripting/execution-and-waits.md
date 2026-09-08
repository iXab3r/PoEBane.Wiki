---
title: Script execution, waits and timers
description: Sequential actions, independent rules, Sleep, WaitUntil, async callbacks and cancellation
published: true
tags: scripting, sleep, wait, timer, async, cancellation, ai-translated
editor: markdown
date: 2026-09-08T00:00:00.000Z
dateCreated: 2026-09-08T00:00:00.000Z
order: 30
---

# Script execution, waits and timers

Write actions in the order you want them to happen. `Sleep` pauses that sequence while other rules
remain able to run. You do not need `async` or `await` to use it.

```ts
OnTimer(100).Do(() => {
  Info("Starting");
  Sleep(1000);
  Info("One second later");
});
```

These primitives are being introduced in POE-81 and are unreleased until their release version is
announced. This guide applies to builds whose `poebane.d.ts` includes them. Older installations,
including 0.2.140, do not provide them. Check the installed declarations before using them.

## Choose the operation

| Operation | Meaning | Result |
| --- | --- | --- |
| `OnTimer(period).Do(action)` | Repeat a rule, allowing `period` milliseconds after each completed run | Rule builder |
| `Sleep(ms)` or `Wait(ms)` | Continue this sequence after a pause | `void` |
| `WaitUntil(condition, options?)` | Check until true or the timeout expires | `boolean` |
| `setTimeout(callback, ms?, ...args)` | Schedule a separate, one-time callback | Numeric cancellation handle |
| `clearTimeout(handle)` | Cancel a callback that has not started | `void` |
| `await promise` | Ordinary JavaScript Promise waiting inside an async function | Promise's fulfilled value |

Durations are milliseconds. Sleep, WaitUntil options and setTimeout delays must be actual finite,
nonnegative numbers. Fractions are allowed. Strings, negative values, `NaN` and `Infinity` are errors;
the runtime does not silently convert them. Sleep requires its argument; setTimeout's omitted delay
is zero. A delay is a minimum: execution resumes on an available host tick, never before the deadline.

## One sequence stays sequential

Sleep pauses the entire current call chain. Helpers, aliases, loops and ordinary JavaScript collection
callbacks retain their normal order. There is no special waiting version of `forEach` to remember.

```ts
function processItem(item: string) {
  Info(`Starting ${item}`);
  Sleep(1000);
  Info(`Finished ${item}`);
}

OnTimer(500).Do(() => {
  ["first", "second", "third"].forEach(processItem);
  Info("All items finished");
});
```

Each item finishes before the next starts. The final message follows the third item. The same applies
to an ordinary `for` loop and nested helpers. `Wait` is an exact alias of Sleep, not a Promise helper.
`Sleep(0)` yields until at least the next tick. A Sleep does not block the operating-system thread or
spin in a busy loop. A long loop that never waits can still exhaust the script CPU watchdog.

Local variables keep their values across waits. Other sequences may update shared JavaScript objects
while yours waits. Read current game state again after waiting; retained world handles can become
stale after a tick or an area change. A previously computed boolean does not update itself.

## Several rules

Each rule has at most one busy run, covering its predicate, action and any Promise the action returns.
While busy it neither starts again nor builds a queue of missed activations. Other rules are independent.

```ts
OnTimer(100).Do(() => {
  Info("A started");
  Sleep(1000);
  Info("A finished");
});

OnTimer(100).Do(() => {
  Info("B started");
  Sleep(300);
  Info("B finished");
});
```

If both first start at time zero, A finishes around 1000 ms and can next start around 1100 ms.
B finishes around 300 ms and can next start around 400 ms. B can complete several runs while A waits.
Actual times follow available ticks; the example does not promise precise wall-clock timing.

**OnTimer uses delay after completion.** Its period starts after the entire previous run finishes,
not when the previous run starts. Even a zero-period rule waits until a later tick. There are no
catch-up bursts. A hotkey or message arriving while its rule is busy does not create another run.

The interpreter runs one active piece of JavaScript at a time. Independent sequences interleave when
one waits; they do not execute JavaScript simultaneously on different threads. Ready continuations
and timers are serviced before new rule activations. The tick loop processes bounded ready work;
newly scheduled work does not recursively execute in the same pass. This is a host tick loop, not a
promise of browser event-loop or microtask-checkpoint ordering.

## Waiting in a condition

Sleep is allowed in `.If(...)`. The action starts only after the condition returns true.

```ts
OnTimer(100)
  .If(() => {
    Sleep(500);
    return Input.IsKeyDown(Key.F1);
  })
  .Do(() => Info("F1 is held after the pause"));
```

This checks the key after 500 ms. The rule remains busy throughout the condition. If the condition is
false, the run completes without Do, and its next period starts then. Moving the key read before
Sleep would intentionally test an earlier snapshot instead. Rule-scoped values remain associated
with the same rule across waits and async continuations.

## WaitUntil

```ts
OnTimer(100).Do(() => {
  const pressed = WaitUntil(
    () => Input.IsKeyDown(Key.F1),
    { timeoutMs: 5000, pollIntervalMs: 50 },
  );
  Info(pressed ? "F1 pressed" : "Timed out");
});
```

WaitUntil calls the condition immediately. True returns true immediately. False schedules the next
check after the polling interval. The default interval is 50 ms; zero means no more than one check
per tick. There is never more than one condition call in progress.

Omitting `timeoutMs` waits until success or cancellation. Zero timeout performs one immediate check.
At each check, success wins before the deadline is tested: a true result on the boundary tick returns
true. A completed false check at or after the deadline returns false. The next pause is shortened to
the remaining timeout when necessary.

The condition must return a boolean, not a Promise. An exception propagates normally. Stop abandons
the execution; neither an exception nor Stop is reported as the boolean false.

Timeout is checked between completed condition calls. If the condition itself sleeps or performs a
long computation, timeout does not interrupt that call. Keep conditions short and read the relevant
world objects inside the callback. The CPU watchdog still applies to code that keeps computing.

## setTimeout and cancellation handles

```ts
OnTimer(1000).Do(() => {
  setTimeout((message) => {
    Info(String(message));
    Sleep(100);
    Info("Callback finished");
  }, 500, "Callback started");
  Info("Do finished; callback is scheduled separately");
});
```

setTimeout returns immediately. The callback starts no earlier than its deadline and the next tick
after registration. Equal-deadline callbacks preserve registration order. Extra arguments are passed
unchanged, including object identity. Only functions are accepted; string code execution is unsupported.
The callback is called with undefined `this`, subject to ordinary JavaScript function rules.

The callback can Sleep or return a Promise. Scheduling it alone does not keep the calling rule busy.
For example, a rule running every 100 ms can accumulate callbacks with 1000 ms delays. For one
sequential operation, put Sleep directly in the rule instead.

```ts
const handle = setTimeout(() => Info("This will be cancelled"), 1000);
clearTimeout(handle);
```

clearTimeout cancels a callback before it starts, even when its deadline has passed and it is ready
in the queue. After the callback starts, clearTimeout cannot interrupt it, including during its Sleep.
Unknown or already-cleared numeric handles do nothing. Handles are not reused within a runtime.
Cancellation does not settle a Promise waiting for that callback.

The existing `setInterval`/`clearInterval` functions use the same scheduling and cancellation ownership.
An interval schedules its next callback when the current callback starts, so waiting callbacks can
overlap. For recurring automation that must remain sequential, prefer OnTimer's delay after completion.

## Explicit async and await

Ordinary JavaScript Promise semantics remain available. The following wrapper works without changes:

```ts
function pause(ms: number) {
  return new Promise<void>(resolve => setTimeout(resolve, ms));
}

OnTimer(100).Do(async () => {
  Info("Started");
  await pause(1000);
  Sleep(500);
  Info("Finished");
});
```

At await, Do returns a pending Promise. The rule stays busy until that Promise settles, including the
later Sleep. Its next OnTimer period starts only after completion. Rejection is a script error,
not a successful completion. Returning a Promise directly also retains the busy run:

```ts
OnTimer(100).Do(() => pause(1000));
```

Ignoring a Promise does not wait for it. The runtime does not automatically await arbitrary calls:

```ts
OnTimer(100).Do(() => {
  pause(1000);
  Info("Immediately after scheduling");
});
```

For everyday sequential actions, use Sleep. Explicit async code still needs ordinary JavaScript
understanding: `forEach(async item => { await pause(1000); })` does not wait for callback Promises.
`forEach(item => { Sleep(1000); })` is sequential because Sleep pauses the actual call stack.

### Async does not start a separate sequence

An async function begins synchronously until it reaches await or returns. Sleep inside that
synchronous portion pauses its caller before the function returns its Promise:

```ts
async function work(name: string) {
  Info(name);
  Sleep(1000);
}

OnTimer(100).Do(async () => {
  await Promise.all([work("A"), work("B")]);
});
```

A sleeps first, then B starts and sleeps. The two argument expressions are evaluated in order.
Replacing Sleep with `await pause(1000)` lets each call return a pending Promise promptly, so both
timers can be scheduled together. Use several rules when you want independent recurring activities.

## Stop, errors and lifecycle

Script UI events and React rendering share one serialized renderer. If a script UI event handler or
component calls Sleep, other script UI events and renders wait until that renderer call finishes.
Ordinary rules, timers outside the renderer, and the application's native UI can continue. Do not
expect another script UI component to refresh during that pause. Runtime teardown cancels waiting
script UI executions along with the runtime's other work.

Stopping automation abandons its active executions, sleeping stacks, scheduled callbacks and Promise
continuations. Enabling again creates fresh executions; old ones cannot wake up and send input.
Callback ownership follows its registration origin, not whichever rule later resolves its Promise.
Separate live subscribers to the same Promise retain their own lifetimes.
An empty `.then()` also registers a continuation with that owner. After cancellation, neither its
forwarding step nor custom Promise result-resolution code can revive the abandoned sequence.

Stop does **not** execute JavaScript `finally` on abandoned stacks. The host releases tracked held
input. Normal completion and ordinary JavaScript exceptions still run normal try/finally logic.
Do not rely on finally as an emergency cleanup mechanism when the whole execution is destroyed.

If another Promise depends on a result from an abandoned execution, it may remain pending. Stop does
not manufacture a success value or resume old catch/finally code with a cancellation exception.

Top-level initialization and registered lifecycle callbacks belong to the script runtime; they are
distinct from automation-owned rule executions. OnAutomationInactive runs on the active-to-inactive
transition. Its own waiting continuation can survive while inactive, under the existing restricted
cleanup/input permissions. OnUnload cannot extend runtime lifetime: only each cleanup callback's
synchronous prefix before its first suspension runs, then teardown destroys the runtime.

## Holding a key

```ts
OnTimer(100).Do(() => {
  Input.KeyDown(Key.Space);
  try {
    Sleep(1000);
  } finally {
    Input.KeyUp(Key.Space);
  }
  Sleep(100);
});
```

Normal execution holds Space for at least one second, releases it, then waits another 100 ms. Stop
during the hold releases it through the host, even though the JavaScript finally does not run.

For AI authors: prefer direct Sleep/Wait/WaitUntil in sequential rules; preserve standard JS helpers
and loops. Do not add async to every function, rewrite collection methods, or invent automatic Promise
waiting. Random helpers are unnecessary: ordinary `Math.random()` is available.

Search terms: sleep, wait, delay, pause, foreach, timeout, condition, polling, timer, cancellation,
sequence, ожидание, последовательность.
