---
title: Waiting recipes
description: Sequential processing, independent rules and waiting while following
published: true
date: 2026-09-08T00:00:00.000Z
tags: scripting, sleep, wait, recipes, ai-translated
editor: markdown
dateCreated: 2026-09-08T00:00:00.000Z
order: 35
---

# Waiting recipes

The POE-81 primitives are unreleased. Check the installed declarations for these functions before using them. Each example below is a separate script.

[Execution model and complete waiting contract](../scripting/execution-and-waits.md).

## Sequential helpers and independent rules

```ts
// Ordinary helpers and forEach remain sequential; no async or await is needed.
function showItem(item: string) {
  Info(`Starting ${item}`);
  Sleep(1000);
  Info(`Finished ${item}`);
}

OnTimer(500).Do(() => {
  ["first", "second", "third"].forEach(showItem);
  Info("All finished; the next run starts after another 500 ms");
});

// This independent rule can finish while the first rule is sleeping.
OnTimer(100).Do(() => {
  Wait(300);
  Info("Independent rule completed");
});
```

## Condition and timeout

```ts
OnTimer(100)
  .If(() => {
    Sleep(50);
    return !Input.IsKeyDown(Key.F1);
  })
  .Do(() => {
    const pressed = WaitUntil(
      () => Input.IsKeyDown(Key.F1),
      { timeoutMs: 5000, pollIntervalMs: 50 },
    );
    Info(pressed ? "F1 pressed" : "Timed out");
  });
```

## Follow movement with Sleep

```ts
// Requires AutoFollow and a leader with a known distance. Random uses standard JavaScript.
OnTimer(100)
  .If(() => Ai.IsActive(Poe2Behavior.AutoFollow)
    && (Party.Leader?.DistanceToPlayer ?? 0) > 500)
  .Do(() => {
    Input.KeyDown(Key.Space);
    try {
      Sleep(1000 + Math.floor(Math.random() * 201));
    } finally {
      Input.KeyUp(Key.Space);
    }
    Sleep(50 + Math.floor(Math.random() * 51));
  });

// Stop destroys this sequence without JS finally; the host releases tracked held input.
// No overlapping activation or backlog accumulates during the two pauses.
```

## Promise wrapper and explicit await

```ts
// Explicit Promise wrappers remain supported. Prefer Sleep for simple sequential scripts.
function waitMs(milliseconds: number) {
  return new Promise<void>(resolve => setTimeout(resolve, milliseconds));
}

OnTimer(100)
  .If(() => Ai.IsActive(Poe2Behavior.AutoFollow)
    && (Party.Leader?.DistanceToPlayer ?? 0) > 500)
  .Do(async () => {
    Input.KeyDown(Key.Space);
    try {
      await waitMs(1000 + Math.floor(Math.random() * 201));
    } finally {
      Input.KeyUp(Key.Space);
    }
    await waitMs(50 + Math.floor(Math.random() * 51));
  });

// The returned Promise keeps this rule busy through both waits.
// Stop abandons pending continuation without JS finally; the host releases held input.
```

On normal completion or a JavaScript exception, finally releases the key. Stop destroys the execution without finally; the host releases tracked input. The Promise wrapper preserves the sequence because async Do returns its Promise. Calling setTimeout alone does not keep a rule busy.
