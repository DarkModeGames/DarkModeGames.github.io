# Shields and damage reduction with a hurt Trigger

**Problem.** A ward should soak the next 50 damage, and a "guard" stance should halve damage for a few seconds,
no matter which monster or spell deals it.

**Pattern.** Don't touch every attack. Put a [`Trigger`](/reference/entities.html#trigger) on the defender's `healthResource` and compare health with a
**shadow** copy (`lastHealthResource`) to see how much was just lost. If a ward is up, take the loss off the ward
and give the health back; if guarding, give half back. Always finish by copying health into the shadow.

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
  wardResource:
    name: Ward
    description: Absorbs damage until it is spent.
    default: 0
    min: '0'
    max: '200'
    reveal: 1
  guardResource:
    name: Guarded
    description: Seconds of halved damage left.
    default: 0
    min: '0'
    max: '10'
    reveal: 1

Condition:
  hurtCondition:
    left: '$healthResource'
    operator: '<'
    right: '$lastHealthResource'
  hasWardCondition:
    left: '$wardResource'
    operator: '>'
    right: '0'
  hasGuardCondition:
    left: '$guardResource'
    operator: '>'
    right: '0'

Parameter:
  healthParameter:
    key: resource
    value: healthResource
  lastHealthParameter:
    key: resource
    value: lastHealthResource
  wardParameter:
    key: resource
    value: wardResource
  lostParameter:
    key: expression
    value: '$lastHealthResource-$healthResource'
  halfLostParameter:
    key: expression
    value: '($lastHealthResource-$healthResource)/2'
  currentHealthParameter:
    key: expression
    value: '$healthResource'
  zeroValueParameter:
    key: value
    value: '0'

Action:
  hurtAction:
    if: hurtCondition
    then: hurtWardAction
    else: hurtSyncAction
  hurtWardAction:
    if: hasWardCondition
    do: minus_resource_self
    parameters: [wardParameter, lostParameter]
    then: hurtRefundAction
    else: hurtGuardAction
  hurtRefundAction:
    do: plus_resource_self
    parameters: [healthParameter, lostParameter]
    then: hurtSyncAction
  hurtGuardAction:
    if: hasGuardCondition
    do: plus_resource_self
    parameters: [healthParameter, halfLostParameter]
    then: hurtSyncAction
    else: hurtSyncAction
  # Shadow = health (set 0, then add: set takes a plain number).
  hurtSyncAction:
    do: set_resource_self
    parameters: [lastHealthParameter, zeroValueParameter]
    then: hurtSyncAddAction
  hurtSyncAddAction:
    do: plus_resource_self
    parameters: [lastHealthParameter, currentHealthParameter]

Trigger:
  hurtTrigger:
    resource: healthResource
    action: hurtAction

Actor:
  tankActor:
    name: Tank
    sprite: heroSprite
    triggers: [hurtTrigger]
    private: [healthResource, wardResource, guardResource]
```

A skill sets `wardResource` (for example to 50) or `guardResource` (seconds), and a 1-second [Timer](/reference/entities.html#timer) counts the guard
down.

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Trigger`](/reference/entities.html#trigger)

**Gotchas.**

- The refund raises health, which fires the Trigger again. That run sees no loss (health isn't below the shadow)
  and only re-syncs, so it settles.
- Several hits in one frame arrive as one change, so the Trigger sees their total.
- This simple ward refunds the whole blow even if the ward had less left. To split it, compare the loss with the
  ward first.
- A killing blow lands before the refund. If something else reacts to "health is 0" in the same frame, list this
  Trigger **before** it in the actor's `triggers`: sibling Triggers run in list order.
- The same shape gives "rage from damage taken": add rage by the loss instead of refunding.
