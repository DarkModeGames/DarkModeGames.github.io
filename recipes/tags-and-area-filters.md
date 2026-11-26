# Tag actors and filter area effects

**Problem.** Your fire nova hurts the caster, your friends and the scenery, not just the enemies.

**Pattern.** `area_of_effect_*` runs its sub-action on **every** actor whose centre is inside the radius: the
caster, allies, enemies and decorations alike. Conditions can't test a [`Group`](/reference/entities.html#group), so give actors **tag resources**
(`isPlayerResource`, `isEnemyResource`) set by a [spawn hook](/recipes/spawn-hook.html), and start the sub-action
with an `if` on the tag of `@` (each actor hit).

```yaml
Resource:
  healthResource:
    name: Health
    default: 100
    min: '0'
    max: '100'
  isEnemyResource:
    name: Is Enemy
    default: 0
    min: '0'
    max: '1'
    reveal: 999999

Condition:
  untaggedCondition:
    left: '$isEnemyResource'
    operator: '=='
    right: '0'
  targetIsEnemyCondition:
    left: '@isEnemyResource'
    operator: '=='
    right: '1'

Parameter:
  healthParameter:
    key: resource
    value: healthResource
  isEnemyParameter:
    key: resource
    value: isEnemyResource
  oneValueParameter:
    key: value
    value: '1'
  novaDamageParameter:
    key: expression
    value: '2d6+4'
  novaRadiusParameter:
    key: radius
    value: '100'
  novaHitParameter:
    key: action
    value: novaHitAction

Action:
  # The caster's skill: hit everything in 100 px...
  fireNovaAction:
    name: Fire Nova
    time: 1.0
    do: area_of_effect_at_self
    parameters: [novaRadiusParameter, novaHitParameter]
  # ...but only enemies take damage.
  novaHitAction:
    if: targetIsEnemyCondition
    do: minus_resource_target
    parameters: [healthParameter, novaDamageParameter]
  # Spawn hook for enemies (see the spawn-hook recipe).
  tagEnemyAction:
    if: untaggedCondition
    do: set_resource_self
    parameters: [isEnemyParameter, oneValueParameter]

Trigger:
  tagEnemyTrigger:
    resource: isEnemyResource
    action: tagEnemyAction

Skill:
  fireNovaSkill:
    name: Fire Nova
    description: Burns every enemy close around you.
    start: fireNovaAction

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    skills: [fireNovaSkill]
  wolfActor:
    name: Grey Wolf
    sprite: wolfSprite
    triggers: [tagEnemyTrigger]
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Skill`](/reference/entities.html#skill), [`Trigger`](/reference/entities.html#trigger)

**Gotchas.**

- Inside the sub-action, `$` is the caster and `@` is the actor being hit. Always filter on `@`.
- The caster is inside its own area. An unfiltered "heal everyone" heals the caster too, which may be what you
  want.
- `targetable: false` decorations are still hit. They hold every resource at its default, so tag `0` filters them
  out.
- The tag [Trigger](/reference/entities.html#trigger) above is the smallest possible hook: it fires on the first frame, and its guard lets it set
  the tag only once.
- A `radius` of `r` reaches `r` px to the sides and `r/2` px up or down the screen, between centres. See
  [Distances and sizes](/reference/units.html).
