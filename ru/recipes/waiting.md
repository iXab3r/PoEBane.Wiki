---
title: Рецепты ожидания
description: Последовательная обработка, независимые правила и ожидание лидера
published: true
date: 2026-09-08T00:00:00.000Z
tags: scripting, sleep, wait, recipes
editor: markdown
dateCreated: 2026-09-08T00:00:00.000Z
order: 35
---

# Рецепты ожидания

Примитивы POE-81 пока не выпущены. Проверьте наличие функций в объявлениях установленной версии перед использованием. Каждый пример ниже — отдельный скрипт.

[Модель исполнения и полный контракт ожиданий](../scripting/execution-and-waits.md).

## Последовательные функции и независимые правила

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

## Условие и предел времени

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

## Движение за лидером со Sleep

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

## Обёртка Promise и явный await

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

При обычном завершении и исключении JavaScript finally отпускает клавишу. Stop уничтожает исполнение без finally, а отслеживаемый ввод освобождает приложение. Обёртка с Promise сохраняет последовательность только потому, что Do возвращает свой Promise через async. Один setTimeout не удерживает правило занятым.
