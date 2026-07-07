---
title: Оверлей HP
description: Рисуйте проценты HP и ES игрока как текст на экране, привязанный к персонажу
published: true
date: 2026-07-07T00:00:00.000Z
tags: PoEBane, Path of Exile 2, скрипты, оверлей, рецепт, ai-translated
editor: markdown
dateCreated: 2026-07-07T00:00:00.000Z
order: 33
---

# Рецепт — оверлей HP

`DisplayText(text, worldPosition, color?)` рисует текст оверлея в мировой позиции — ручная проекция не
нужна. Привяжите его к игроку, чтобы получить плавающий индикатор витала.

```ts
if (PlayerValid()) {
  const p = World.Player;
  if (p) {
    const hp = p.HP.Percent.toFixed(0);
    const es = p.ES.Percent.toFixed(0);
    DisplayText(`HP ${hp}%  ES ${es}%`, p.WorldPosition);
  }
}
```

Замечания:

- `WorldPosition` — это `Vector3`; `DisplayText` принимает мировую точку напрямую и сам размещает
  подпись.
- Вызывайте это из тела тика, чтобы подпись следовала за персонажем каждый кадр.
- Передайте необязательный `color` третьим аргументом, чтобы задать цвет текста — тип цвета смотрите в
  `poebane.d.ts`.
