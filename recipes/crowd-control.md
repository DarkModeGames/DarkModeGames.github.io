# Stuns and interrupts with a swing lockout

**Problem.** A shield bash should stop a monster attacking for a few seconds.

**Pattern.** A monster that paces its attacks with a swing cooldown (see [Monster AI](/recipes/mob-ai-strategy.html))
only attacks while that cooldown is `0`. So a stun is simply `set_resource_target` on the monster's
`swingCooldownResource`: it can't swing until its own [Timer](/reference/entities.html#timer) counts it back down. A `stunLockResource` on the
target stops the stun being chained forever.

```yaml
Resource:
  isEnemyResource:
    name: Is Enemy
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
  stunLockResource:
    name: Stun Immunity
    default: 0
    min: '0'
    max: '20'
    reveal: 999999
  bashCooldownResource:
    name: Shield Bash (cooldown)
    default: 0
    min: '0'
    max: '12'
    reveal: 1

Condition:
  bashReadyCondition:
    left: '$bashCooldownResource'
    operator: '=='
    right: '0'
  targetIsEnemyCondition:
    left: '@isEnemyResource'
    operator: '=='
    right: '1'
  targetStunnableCondition:
    left: '@stunLockResource'
    operator: '=='
    right: '0'

Parameter:
  swingCooldownParameter:
    key: resource
    value: swingCooldownResource
  stunLockParameter:
    key: resource
    value: stunLockResource
  bashCooldownParameter:
    key: resource
    value: bashCooldownResource
  stunSecondsParameter:
    key: value
    value: '4'
  stunLockSecondsParameter:
    key: value
    value: '12'
  bashCooldownValueParameter:
    key: value
    value: '12'
  oneExpressionParameter:
    key: expression
    value: '1'

Action:
  shieldBashAction:
    name: Shield Bash
    time: 0.8
    if: bashReadyCondition
    then: shieldBashTargetAction
  shieldBashTargetAction:
    if: targetIsEnemyCondition
    do: set_resource_self
    parameters: [bashCooldownParameter, bashCooldownValueParameter]
    then: shieldBashStunAction
  shieldBashStunAction:
    if: targetStunnableCondition
    do: set_resource_target
    parameters: [swingCooldownParameter, stunSecondsParameter]
    then: shieldBashLockAction
  shieldBashLockAction:
    do: set_resource_target
    parameters: [stunLockParameter, stunLockSecondsParameter]
  # Monster's 1 s tick: count both down.
  monsterTickAction:
    do: minus_resource_self
    parameters: [swingCooldownParameter, oneExpressionParameter]
    then: monsterTickLockAction
  monsterTickLockAction:
    do: minus_resource_self
    parameters: [stunLockParameter, oneExpressionParameter]

Timer:
  monsterTickTimer:
    interval: '1'
    total: '0'
    action: monsterTickAction

Skill:
  shieldBashSkill:
    name: Shield Bash
    description: Stun the target for 4 seconds.
    start: shieldBashAction

Actor:
  tankActor:
    name: Tank
    sprite: heroSprite
    skills: [shieldBashSkill]
  ogreActor:
    name: Ogre
    sprite: wolfSprite
    timers: [monsterTickTimer]
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Skill`](/reference/entities.html#skill), [`Timer`](/reference/entities.html#timer)

**Gotchas.**

- This stops attacks, not movement. To root as well, add a [slow](/recipes/slows-that-dont-stack.html) with speed
  `0` for the same time.
- `set` (not `plus`) on the cooldown means a stun never stacks into a longer one.
- The lock is longer than the stun, so a second bash can't land until the monster has had a few swings.
- Bosses can ignore stuns: give them `stunLockResource` a high value in their spawn hook and never count it down.
- A player's skill lock is separate (`Action.time`); this lockout only works on actors whose AI reads the
  cooldown.
