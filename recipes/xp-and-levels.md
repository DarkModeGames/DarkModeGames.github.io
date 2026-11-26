# XP and level-up

**Problem.** Players should earn XP and level up when they reach a threshold, even when one big reward is worth
two levels, without the level running away.

**Pattern.** A [`Trigger`](/reference/entities.html#trigger) on `xpResource` runs a guarded chain: if XP has reached the threshold **and** the level is
below the cap, spend the threshold's worth of XP, raise the level, and announce it. Spending XP changes it, so the
Trigger fires again on the next frame: a second level-up happens only if there is still enough XP. The guards end
the loop.

```yaml
Resource:
  xpResource:
    name: Experience
    default: 0
    min: '0'
    max: '999999'
  levelResource:
    name: Level
    default: 1
    min: '1'
    max: '10'

Measure:
  xpToLevelMeasure:
    name: Next Level
    description: Experience needed for the next level.
    expression: '$levelResource*100'
    private: true

Condition:
  enoughXpCondition:
    left: '$xpResource'
    operator: '>='
    right: '$xpToLevelMeasure'
  belowCapCondition:
    left: '$levelResource'
    operator: '<'
    right: '10'

Parameter:
  xpParameter:
    key: resource
    value: xpResource
  levelParameter:
    key: resource
    value: levelResource
  spendXpParameter:
    key: expression
    value: '$xpToLevelMeasure'
  oneExpressionParameter:
    key: expression
    value: '1'
  levelUpMessageParameter:
    key: message
    value: 'Level up!'
  levelUpColorParameter:
    key: color
    value: '#ffd040'

Action:
  levelUpAction:
    if: enoughXpCondition
    then: levelUpCapAction
  levelUpCapAction:
    if: belowCapCondition
    do: minus_resource_self
    parameters: [xpParameter, spendXpParameter]
    then: levelUpRaiseAction
  levelUpRaiseAction:
    do: plus_resource_self
    parameters: [levelParameter, oneExpressionParameter]
    then: levelUpAnnounceAction
  levelUpAnnounceAction:
    do: echo_self
    parameters: [levelUpMessageParameter, levelUpColorParameter]

Trigger:
  levelUpTrigger:
    resource: xpResource
    action: levelUpAction

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    triggers: [levelUpTrigger]
    measures: [xpToLevelMeasure]
    private: [levelResource, xpResource]
```

Award XP with `plus_resource_target` on `xpResource` from a monster's death action, filtered to players in range
(see [Tags and area filters](/recipes/tags-and-area-filters.html)).

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Measure`](/reference/entities.html#measure), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Trigger`](/reference/entities.html#trigger)

**Gotchas.**

- Spend XP **before** raising the level. The threshold reads the level, so raising first would spend the next
  level's amount.
- The cap check matters: at the cap, XP keeps arriving and the Trigger keeps firing. Without the guard it would
  level forever.
- A [Condition](/reference/entities.html#condition)'s `right` is dice too, so it can compare against a [Measure](/reference/entities.html#measure).
- The public `echo_self` (the default audience) shows the line to everyone nearby. A level-up is a good use;
  routine events are not.
