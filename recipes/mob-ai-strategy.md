# Monster AI: acquire, chase, attack, leash

**Problem.** You want a monster that notices a player, runs over, attacks on a rhythm, and gives up if led too far
away.

**Pattern.** A [`Strategy`](/reference/entities.html#strategy) is an ordered list of [`Behavior`](/reference/entities.html#behavior)s. Each tick the engine walks the list from the current
behavior and runs the action of the **first behavior whose goals are not all met**. If an earlier behavior's goals
stop holding, it takes over again. If every goal is met, nothing runs. So write each goal as "what must be true
before we move on", and put the always-running attack last with a goal that is never met.

1. **Evade:** goal "not evading". After a leash, it holds the monster still for a few seconds.
2. **Acquire:** goal "my target is a player". Its action targets the nearest player in view.
3. **Leash:** goal "target within 360 px". Its action drops the target, heals, and starts evading.
4. **Close in:** goal "within 40 px". Its action walks to the target.
5. **Attack:** goal never met. Its action swings when the swing timer is ready.

```yaml
Group:
  playersGroup:
    name: Adventurers
    color: '#5DA9E9'
  enemiesGroup:
    name: Hostile
    color: '#B83A3A'

Resource:
  healthResource:
    name: Health
    default: 100
    min: '0'
    max: '100'
  isPlayerResource:
    name: Is Player
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  swingCooldownResource:
    name: Swing Cooldown
    default: 0
    min: '0'
    max: '10'
    reveal: 999999
  evadeResource:
    name: Evading
    default: 0
    min: '0'
    max: '10'
    reveal: 999999

Condition:
  neverCondition:
    left: '0'
    operator: '=='
    right: '1'
  notEvadingCondition:
    left: '$evadeResource'
    operator: '=='
    right: '0'
  targetIsPlayerCondition:
    left: '@isPlayerResource'
    operator: '=='
    right: '1'
  withinLeashCondition:
    left: '$distance_to_target'
    operator: '<='
    right: '360'
  inMeleeCondition:
    left: '$distance_to_target'
    operator: '<='
    right: '40'
  swingReadyCondition:
    left: '$swingCooldownResource'
    operator: '=='
    right: '0'

Parameter:
  healthParameter:
    key: resource
    value: healthResource
  swingCooldownParameter:
    key: resource
    value: swingCooldownResource
  evadeParameter:
    key: resource
    value: evadeResource
  playersGroupParameter:
    key: group
    value: playersGroup
  biteDamageParameter:
    key: expression
    value: '1d6+2'
  swingValueParameter:
    key: value
    value: '2'
  evadeValueParameter:
    key: value
    value: '5'
  fullHealthValueParameter:
    key: value
    value: '999999'
  oneExpressionParameter:
    key: expression
    value: '1'

Action:
  waitAction:
    do: wait
  acquireAction:
    do: target_nearest_in_group
    parameters: [playersGroupParameter]
  leashAction:
    do: clear_target_self
    then: leashHealAction
  leashHealAction:
    do: set_resource_self
    parameters: [healthParameter, fullHealthValueParameter]
    then: leashEvadeAction
  leashEvadeAction:
    do: set_resource_self
    parameters: [evadeParameter, evadeValueParameter]
  chaseAction:
    do: move_to_target
  biteAction:
    if: swingReadyCondition
    do: minus_resource_target
    parameters: [healthParameter, biteDamageParameter]
    then: biteResetAction
  biteResetAction:
    do: set_resource_self
    parameters: [swingCooldownParameter, swingValueParameter]
  # Every second: count the swing and evade timers down.
  wolfTickAction:
    do: minus_resource_self
    parameters: [swingCooldownParameter, oneExpressionParameter]
    then: wolfTickEvadeAction
  wolfTickEvadeAction:
    do: minus_resource_self
    parameters: [evadeParameter, oneExpressionParameter]

Behavior:
  evadeBehavior:
    goals: [notEvadingCondition]
    action: waitAction
  acquireBehavior:
    goals: [targetIsPlayerCondition]
    action: acquireAction
  leashBehavior:
    goals: [withinLeashCondition]
    action: leashAction
  closeBehavior:
    goals: [inMeleeCondition]
    action: chaseAction
  attackBehavior:
    goals: [neverCondition]
    action: biteAction

Strategy:
  wolfStrategy:
    behaviors: [evadeBehavior, acquireBehavior, leashBehavior, closeBehavior, attackBehavior]

Timer:
  wolfTickTimer:
    interval: '1'
    total: '0'
    action: wolfTickAction

Actor:
  wolfActor:
    name: Grey Wolf
    sprite: wolfSprite
    group: enemiesGroup
    perception: 300
    salience: 240
    speed: 1.1
    strategy: wolfStrategy
    timers: [wolfTickTimer]
    public: [healthResource]
```

Players need `group: playersGroup` and `isPlayerResource` set to `1` by a
[spawn hook](/recipes/spawn-hook.html).

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Behavior`](/reference/entities.html#behavior), [`Condition`](/reference/entities.html#condition), [`Group`](/reference/entities.html#group), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Strategy`](/reference/entities.html#strategy), [`Timer`](/reference/entities.html#timer)

**Gotchas.**

- Strategies run on the host, every physics frame. Pace attacks with a cooldown resource, not with the action's
  `time`; a `time` on a strategy action roots the monster for that long.
- The monster only finds players it can see: its `perception` plus the player's `salience`. That is its aggro
  range.
- Check that the target is really a player (the acquire goal). After the target dies, leaves or changes form, the
  goal fails and the monster looks again.
- `$distance_to_target` is in isometric pixels, so 360 reaches 360 px sideways but only 180 px up or down the
  screen. See [Distances and sizes](/reference/units.html).
- After the evade ends, a player still in sight is picked up again. To make a monster commit only to close
  players, keep an "aware" resource that turns on when a player comes within a short distance or the monster is
  hit, and give the acquire action an `if` on it.
- A monster has no home point. The leash drops the target where it stands. Keep spawn areas apart from
  graveyards and roads.
