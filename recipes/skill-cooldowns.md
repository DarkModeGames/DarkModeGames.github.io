# Give a skill a cooldown

**Problem.** A skill should be usable once every few seconds, and the player should be told when it isn't ready.

**Pattern.** A cooldown is a [Resource](/reference/entities.html#resource) that counts seconds left. The skill's start action checks it with `if`. On
success it sets the cooldown and does its effect. On failure it shows a short **private** line that only the
caster's own player sees. A 1-second [`Timer`](/reference/entities.html#timer) counts every cooldown down.

```yaml
Resource:
  healthResource:
    name: Health
    default: 100
    min: '0'
    max: '100'
  strikeCooldownResource:
    name: Strike (cooldown)
    description: Seconds until Strike is ready.
    default: 0
    min: '0'
    max: '6'
    reveal: 1

Condition:
  strikeReadyCondition:
    left: '$strikeCooldownResource'
    operator: '=='
    right: '0'

Parameter:
  healthParameter:
    key: resource
    value: healthResource
  strikeCooldownParameter:
    key: resource
    value: strikeCooldownResource
  strikeCooldownValueParameter:
    key: value
    value: '6'
  strikeDamageParameter:
    key: expression
    value: '2d6+3'
  oneExpressionParameter:
    key: expression
    value: '1'
  notReadyMessageParameter:
    key: message
    value: 'Not ready'
  privateAudienceParameter:
    key: audience
    value: private

Action:
  strikeAction:
    name: Strike
    time: 0.8
    if: strikeReadyCondition
    do: set_resource_self
    parameters: [strikeCooldownParameter, strikeCooldownValueParameter]
    then: strikeHitAction
    else: strikeNotReadyAction
  strikeHitAction:
    do: minus_resource_target
    parameters: [healthParameter, strikeDamageParameter]
  strikeNotReadyAction:
    do: echo_self
    parameters: [notReadyMessageParameter, privateAudienceParameter]
  cooldownTickAction:
    do: minus_resource_self
    parameters: [strikeCooldownParameter, oneExpressionParameter]

Timer:
  cooldownTimer:
    interval: '1'
    total: '0'
    action: cooldownTickAction

Skill:
  strikeSkill:
    name: Strike
    description: A hard blow. Ready every 6 seconds.
    start: strikeAction

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    skills: [strikeSkill]
    timers: [cooldownTimer]
    private: [healthResource, strikeCooldownResource]
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Skill`](/reference/entities.html#skill), [`Timer`](/reference/entities.html#timer)

**Gotchas.**

- `total: '0'` repeats the Timer forever. Counting down at `0` does nothing, because a value never goes below
  `min`.
- One Timer can tick several cooldowns: chain more `minus_resource_self` actions with `then`.
- `reveal: 1` shows the cooldown on the plate only while it is counting.
- `time` locks the caster for that many seconds and acts as a global cooldown. It is a float (`0.8`, `1.0`).
- A press whose first `if` fails before any `do` has run is **refused**: the lock and the animation end at once.
  So put the cooldown check first, before anything that changes a resource.
- Keep failure lines short and private ("Not ready", "Out of range"). See
  [Barks and feedback](/recipes/barks-and-feedback.html).
- Put every check first in the chain (has a target, in range, cooldown, cost), so a refused press spends nothing.
