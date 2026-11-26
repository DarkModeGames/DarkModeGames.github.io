# Ranges, sight and area sizes

**Problem.** You set a melee range of 40 and an area of 100, and up-and-down the screen they seem half as big as
side to side.

**Pattern.** Every ground distance is in **isometric pixels**: `sqrt(dx² + (2·dy)²)` between actor centres. A
distance `r` reaches `r` px to either side and `r/2` px up or down the screen, the same ellipse the ground is
drawn on. Melee checks, spell ranges, leashes, sight and area radii all use it, so one number means the same thing
everywhere. Write ranges as conditions on `$distance_to_target`.

```yaml
Condition:
  inMeleeCondition:
    left: '$distance_to_target'
    operator: '<='
    right: '40'
  inSpellRangeCondition:
    left: '$distance_to_target'
    operator: '<='
    right: '260'
  withinLeashCondition:
    left: '$distance_to_target'
    operator: '<='
    right: '360'

Parameter:
  quakeRadiusParameter:
    key: radius
    value: '120'
  quakeEachParameter:
    key: action
    value: quakeEachAction

Action:
  quakeAction:
    do: area_of_effect_at_self
    parameters: [quakeRadiusParameter, quakeEachParameter]
  quakeEachAction:
    do: wait

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    base: 16
    perception: 240
    salience: 240
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter)

**Gotchas.**

- `perception` is how far an actor sees, `salience` how far away it can be seen. One actor sees another when the
  ellipses overlap, so a monster with perception 300 notices a player with salience 240 at about 540 px.
- Keep a monster's sight wider than its leash, or it loses its target at the edge of sight before the leash
  check runs.
- An area of radius `r` hits actors whose **centre** is within `r`. Big actors don't reach further.
- `$distance_to_target` to an actor on another map is huge, so range checks fail safely.
- [`Vector`](/reference/entities.html#vector) positions and sprite sizes are plain pixels, not ground distances. See
  [Distances and sizes](/reference/units.html) for the full table.
