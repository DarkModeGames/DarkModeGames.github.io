# Health regeneration only out of combat

**Problem.** Health should refill on its own, but only after a few seconds without taking damage, so fights still
need healers and potions.

**Pattern.** `combatResource` is a countdown: 5 means "just fought", 0 means "safe". A health [`Trigger`](/reference/entities.html#trigger) sets it to
5 whenever health drops below a shadow copy (`lastHealthResource`), and always re-syncs the shadow. A 1-second
[Timer](/reference/entities.html#timer) counts combat down while it is above 0, and heals only once it reaches 0.

```yaml
Resource:
  healthResource:
    name: Health
    default: 100
    min: '0'
    max: '100'
  lastHealthResource:
    name: Last Health
    default: 100
    min: '0'
    max: '99999'
    reveal: 999999
  combatResource:
    name: In Combat
    default: 0
    min: '0'
    max: '5'
    reveal: 999999

Condition:
  tookDamageCondition:
    left: '$healthResource'
    operator: '<'
    right: '$lastHealthResource'
  inCombatCondition:
    left: '$combatResource'
    operator: '>'
    right: '0'

Parameter:
  healthParameter:
    key: resource
    value: healthResource
  lastHealthParameter:
    key: resource
    value: lastHealthResource
  combatParameter:
    key: resource
    value: combatResource
  combatValueParameter:
    key: value
    value: '5'
  zeroValueParameter:
    key: value
    value: '0'
  oneExpressionParameter:
    key: expression
    value: '1'
  currentHealthParameter:
    key: expression
    value: '$healthResource'
  regenAmountParameter:
    key: expression
    value: '5'

Action:
  # Health Trigger: a drop starts (or restarts) the countdown.
  regenHurtAction:
    if: tookDamageCondition
    do: set_resource_self
    parameters: [combatParameter, combatValueParameter]
    then: regenSyncAction
    else: regenSyncAction
  regenSyncAction:
    do: set_resource_self
    parameters: [lastHealthParameter, zeroValueParameter]
    then: regenSyncAddAction
  regenSyncAddAction:
    do: plus_resource_self
    parameters: [lastHealthParameter, currentHealthParameter]
  # 1 s Timer: count down, or heal once safe.
  regenTickAction:
    if: inCombatCondition
    do: minus_resource_self
    parameters: [combatParameter, oneExpressionParameter]
    else: regenHealAction
  regenHealAction:
    do: plus_resource_self
    parameters: [healthParameter, regenAmountParameter]

Trigger:
  regenHurtTrigger:
    resource: healthResource
    action: regenHurtAction

Timer:
  regenTickTimer:
    interval: '1'
    total: '0'
    action: regenTickAction

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    triggers: [regenHurtTrigger]
    timers: [regenTickTimer]
    private: [healthResource]
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Timer`](/reference/entities.html#timer), [`Trigger`](/reference/entities.html#trigger)

**Gotchas.**

- The heal is a rise, so it never restarts the countdown. Only a drop does.
- Healing at full health does nothing: the `max` bound clamps it.
- The first damage after a long rest shows the order: hit at second 0, countdown 5…1, first heal at second 5.
- To also count dealing damage as combat, add a second Trigger on a `threatResource` that the player's attacks
  raise, with the same countdown.
- Put this Trigger **before** any Trigger that refunds damage (shields) in `triggers`, so it sees the blow first.
