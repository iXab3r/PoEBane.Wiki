---
title: Начало работы
description: Модель скриптов PoEBane — чтение World, проекция, управление вводом, переключатели SetAuto, оверлей и логирование
published: true
date: 2026-07-07T00:00:00.000Z
tags: PoEBane, Path of Exile 2, скрипты, начало работы, ai-translated
editor: markdown
dateCreated: 2026-07-07T00:00:00.000Z
order: 10
---

# Начало работы со скриптами PoEBane

Скрипт PoEBane — это **TypeScript/JavaScript**, выполняемый на **живом** процессе *Path of Exile 2*.
Вы редактируете его в редакторе Monaco внутри Studio (с типами из `poebane.d.ts`), и его выполняет
встроенный рантайм. Всё ниже основано на сгенерированном `poebane.d.ts` — актуальном API, который
выпускает рабочая сборка. Эта вики описывает прозу; `.d.ts` содержит точные сигнатуры.

## 1. Чтение состояния игры

`World` (тип `Poe2WorldState`) — точка входа для живого состояния:

| Аксессор | Что даёт |
| --- | --- |
| `World.Player` | сущность игрока `Entity` (или `null` вне игры) |
| `World.Entities` | ближайшие сущности (мобы, NPC, объекты) |
| `World.Flasks` | пояс на 5 слотов (флаконы жизни/маны + талисманы) |
| `World.Inventory` | предметы инвентаря |
| `World.ItemsOnGround` | подписи предметов на земле |
| `World.Area` | текущая область / зона |
| `World.Camera` | камера + проекция |
| `World.Party` | участники группы и приглашения |
| `World.Ui` | панели UI (стэш, инвентарь, торговля, подписи на земле) |
| `World.Stash` | состояние панели стэша |
| `World.Osd` | рисование поверх экрана (OSD) |

`Entity` даёт `IsValid`, `Id`, `Type`, `Path`, `Metadata`, `PlayerName`, `IsAlive`, `IsTargetable`,
`WorldPosition` (`Vector3`), `GridPosition` (`Vector2`) и виталы (`HP` / `ES` / `Mana` / `Ward`,
каждый — `Vital` с `Current` / `Max` / `Percent`).

**Всегда защищайте чтение:** сначала вызовите `PlayerValid()` и проверьте `World.Player` на `null`.

## 2. Проекция — мир в экран

- `World.W2S(worldPos)` → экранная точка `Vector2`; `World.TryW2S(worldPos)` → `Vector2 | null`.
- `World.W2C(worldPos)` / `World.TryW2C(worldPos)` → точка в координатах клиента.

Используйте их, чтобы разместить рисование оверлея в мировой позиции.

## 3. Управление вводом

- `PressKey(key)`, `HoldKey(key)`, `ReleaseKey(key)`, `IsKeyPressed(key)` — `key` это член перечисления
  `Key` (см. `poebane.d.ts`).
- `Input` (тип `Poe2InputController`) для более низкоуровневого управления.
- `World.Flasks.Use(index)` использует слот пояса напрямую (`0`/`1` = жизнь/мана, `2..4` = талисманы).

## 4. Высокоуровневые переключатели автоматизации

Предпочитайте их реализации поведения из примитивов:

```ts
SetAutoStash(true);
SetAutoFarm(true);
SetAutoFollow(true);
SetAutoPickup(true);
```

## 5. Оверлей и логирование

- `DisplayText(text, worldPosition, color?)` рисует текст в мировой позиции.
- `Log` / `Trace` / `Debug` / `Info` / `Warn` / `Error` и `console.log(...)` для логирования.

## 6. Реактивные правила (в разработке)

`Group(name)`, `OnTimer(interval_ms)` и `OnHotkey(gesture, scope?)` возвращают билдер-контексты, а
`EnableGroup` / `DisableGroup` включают/выключают именованные группы. Этот реактивный слой ещё
дорабатывается — **проверьте в `poebane.d.ts`, что реально поддерживает возвращаемый контекст**, прежде
чем полагаться на него. Императивная часть выше стабильна уже сегодня.

## Пример — флакон жизни при низком HP

Соответствует текущему API (`PlayerValid`, `World.Player`, `Vital.Percent`,
`Poe2FlasksAccessor.HasItemAt` / `Use`, `Info`):

```ts
// Выполняется на живой игре. Используем слот пояса 0 (флакон жизни), когда HP ниже 50%.
if (PlayerValid()) {
  const player = World.Player;            // Entity | null
  if (player) {
    const hp = player.HP;                 // Vital { Current, Max, Percent }
    if (hp.Percent < 50 && World.Flasks.HasItemAt(0)) {
      World.Flasks.Use(0);
      Info(`использован флакон жизни при ${hp.Percent.toFixed(0)}% HP`);
    }
  }
}
```

Больше готовых скриптов — в [рецептах](../recipes/low-life-flask.md), а перед публикацией загляните в
[лучшие практики](../best-practices.md).
