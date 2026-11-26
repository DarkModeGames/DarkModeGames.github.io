# Give every resource bounds, and hide internal state

**Problem.** Some of your numbers are for the player to see (health, gold), and some are bookkeeping (a quest
flag, a cooldown, a "has been set up" marker). You want the right ones on screen and every value kept in range.

**Pattern.** Give **every** [`Resource`](/reference/entities.html#resource) an explicit `min` and `max`, written as quoted strings (they are dice
expressions). Use `reveal` to decide when a value appears: the HUD plate and the inventory show a resource only
while its value is at least `reveal`. Internal state gets a `reveal` so high it never shows.

```yaml
Resource:
  healthResource:
    name: Health
    description: Reach 0 and you fall.
    default: 100
    min: '0'
    max: '100'
  goldResource:
    name: Gold
    default: 0
    min: '0'
    max: '99999'
  # An item: shown only while you own at least one.
  ropeResource:
    name: Coil of Rope
    default: 0
    min: '0'
    max: '3'
    reveal: 1
  # Internal bookkeeping: never shown.
  metFerrymanResource:
    name: Met the Ferryman
    default: 0
    min: '0'
    max: '1'
    reveal: 999999

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    private: [healthResource, goldResource]
    public: [healthResource]
```

**Entities used.** [`Actor`](/reference/entities.html#actor), [`Resource`](/reference/entities.html#resource)

**Gotchas.**

- An unset `max` counts as `0`, so the value is clamped to `0` and the resource looks broken. Values are also never
  below `0`.
- `min` and `max` must be quoted strings. A bare number fails validation with "Expected DICE_EXPRESSION".
- Bounds are dice: `max: '$maxHealthMeasure'` works. `$` is the owner. Don't use `@` in bounds; it reads whatever
  the owner happens to be targeting.
- Every actor carries every resource at its `default`. `private:` puts a resource on the owner's own plate,
  `public:` on other players' view of the actor.
- `reveal` is a fixed number compared with the value. It can't read another resource.
- A resource the player sees should have an `icon`, or the plate shows a placeholder.

See [Distances and sizes](/reference/units.html) and the [entity reference](/reference/entities.html) for every
`Resource` field.
