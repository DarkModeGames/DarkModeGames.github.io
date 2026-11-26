# Scale a boss to the party

**Problem.** A boss tuned for four players is a wall for one and a pushover for six of them.

**Pattern.** When the boss engages, it counts the players near it. An `area_of_effect_at_self` runs a sub-action
on every actor in range with the **boss** as caller, so a sub-action that does `plus_resource_self` on
`partyResource` (filtered to players) counts them on the boss. Maximum health is a
[`Measure`](/reference/entities.html#measure) of that count, and a final `set` to a huge number refills health to
the new maximum.

```yaml
Resource:
  isPlayerResource:
    name: Is Player
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  partyResource:
    name: Party Size
    default: 1
    min: '0'
    max: '8'
    reveal: 999999
  healthResource:
    name: Health
    default: 2900
    min: '0'
    max: '$maxHealthMeasure'
  engagedResource:
    name: Engaged
    default: 0
    min: '0'
    max: '1'
    reveal: 999999

Measure:
  maxHealthMeasure:
    name: Maximum Health
    expression: '1000+$partyResource*1900'

Condition:
  targetIsPlayerCondition:
    left: '@isPlayerResource'
    operator: '=='
    right: '1'
  notEngagedCondition:
    left: '$engagedResource'
    operator: '=='
    right: '0'

Parameter:
  partyParameter:
    key: resource
    value: partyResource
  engagedParameter:
    key: resource
    value: engagedResource
  healthParameter:
    key: resource
    value: healthResource
  zeroValueParameter:
    key: value
    value: '0'
  oneValueParameter:
    key: value
    value: '1'
  fullHealthValueParameter:
    key: value
    value: '999999'
  oneExpressionParameter:
    key: expression
    value: '1'
  countRadiusParameter:
    key: radius
    value: '650'
  countEachParameter:
    key: action
    value: countPlayerAction

Action:
  # Run once, from the boss's acquire step.
  bossEngageAction:
    if: notEngagedCondition
    do: set_resource_self
    parameters: [engagedParameter, oneValueParameter]
    then: bossCountResetAction
  bossCountResetAction:
    do: set_resource_self
    parameters: [partyParameter, zeroValueParameter]
    then: bossCountAction
  bossCountAction:
    do: area_of_effect_at_self
    parameters: [countRadiusParameter, countEachParameter]
    then: bossRefillAction
  countPlayerAction:
    if: targetIsPlayerCondition
    do: plus_resource_self
    parameters: [partyParameter, oneExpressionParameter]
  bossRefillAction:
    do: set_resource_self
    parameters: [healthParameter, fullHealthValueParameter]

Actor:
  bossActor:
    name: The Warden
    sprite: wolfSprite
    public: [healthResource]
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Measure`](/reference/entities.html#measure), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource)

**Gotchas.**

- The whole area effect finishes before `then` runs, so the refill sees the final count.
- A [Resource](/reference/entities.html#resource)'s `max` can be dice. Here it reads the Measure, so health is capped by the party count.
- `set_resource_*` takes a plain number, not dice. To set a computed value, set 0 and then `plus` the expression,
  or, as here, let the bounds do the work.
- Count once per pull. Clear `engagedResource` on a leash so the next pull counts again.
- The radius is in isometric pixels: 650 reaches 650 px sideways and 325 px up or down the screen.
