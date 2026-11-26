# Derive stats from level and gear with Measures

**Problem.** Attack power should grow with level and with the sword you have equipped, without you updating a
number every time either changes.

**Pattern.** Store the raw facts as Resources (base attack, level, "sword equipped" as 0 or 1) and compute the stat
as a [`Measure`](/reference/entities.html#measure). A Measure is a dice expression worked out fresh every time it is read, so it is always current.
Multiplying by an `..._equipped` flag (0 or 1) switches a bonus on and off.

```yaml
Resource:
  baseAttackResource:
    name: Base Attack
    default: 5
    min: '0'
    max: '999'
    reveal: 999999
  levelResource:
    name: Level
    default: 1
    min: '1'
    max: '20'
  ironSwordEquippedResource:
    name: Iron Sword (equipped)
    default: 0
    min: '0'
    max: '1'
    reveal: 1

Measure:
  attackPowerMeasure:
    name: Attack Power
    description: Added to every weapon blow.
    expression: '$baseAttackResource+$levelResource*2+$ironSwordEquippedResource*4'
    private: true

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    measures: [attackPowerMeasure]
    private: [levelResource]
```

Use it anywhere dice are allowed, for example a hit of `'1d6+$attackPowerMeasure'`.

**Entities used.** [`Actor`](/reference/entities.html#actor), [`Measure`](/reference/entities.html#measure), [`Resource`](/reference/entities.html#resource)

**Gotchas.**

- A Measure shows on the owner's plate only if it has `private: true` **and** is listed in the actor's
  `measures:`.
- Dice work out `*` and `/` before `+`, and `+` before `-`. So `a-b+c` means `a-(b+c)`. Put subtractions last or
  use parentheses.
- A result is never below `0`, and is rounded to a whole number.
- Measures are read, never written. To change the stat, change the Resources behind it.
- See [Items as resources](/recipes/items-as-resources.html) for the menus that set `..._equipped`.
