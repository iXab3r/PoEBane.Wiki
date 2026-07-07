---
title: Флакон при низком HP
description: Автоматически используйте флаконы жизни и маны, когда виталы игрока падают ниже порога
published: true
date: 2026-07-07T00:00:00.000Z
tags: PoEBane, Path of Exile 2, скрипты, флаконы, рецепт, ai-translated
editor: markdown
dateCreated: 2026-07-07T00:00:00.000Z
order: 31
---

# Рецепт — флакон при низком HP

Нажимайте флакон жизни при падении HP и флакон маны при падении маны. Слоты пояса: `0` = жизнь,
`1` = мана, `2..4` = талисманы. `World.Flasks.HasItemAt(i)` проверяет, что в слоте есть готовый флакон;
`World.Flasks.Use(i)` нажимает его.

```ts
// Флакон жизни ниже 50% HP, флакон маны ниже 25% маны.
if (PlayerValid()) {
  const p = World.Player;                 // Entity | null
  if (p) {
    if (p.HP.Percent < 50 && World.Flasks.HasItemAt(0)) {
      World.Flasks.Use(0);
      Info(`флакон жизни при ${p.HP.Percent.toFixed(0)}% HP`);
    }
    if (p.Mana.Percent < 25 && World.Flasks.HasItemAt(1)) {
      World.Flasks.Use(1);
    }
  }
}
```

Замечания:

- `HP` / `Mana` — это объекты `Vital`; читайте `.Percent` (0–100), а не сырой `.Current`.
- `HasItemAt(i)` защищает от нажатия пустого или незаряженного слота.
- Держите это в теле тика: пара дешёвых чтений плюс условное нажатие.
- Талисманы находятся в слотах `2..4` — тот же приём `HasItemAt` / `Use`.
