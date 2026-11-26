# Slows that don't stack

**Problem.** A frost bolt should slow its target for a few seconds. Two slows close together leave the target slow
for good.

**Pattern.** `temp_speed_target` sets a speed for `duration` seconds, then puts back the speed it **saw when it
started**. A second slow during the first sees the slowed speed and later restores that, so the target stays
slow. Guard each slow with a lock resource on the target that lasts at least as long as the slow, and count it down
on the target.

```yaml
Resource:
  isEnemyResource:
    name: Is Enemy
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  slowLockResource:
    name: Slow Lock
    default: 0
    min: '0'
    max: '30'
    reveal: 999999

Condition:
  targetIsEnemyCondition:
    left: '@isEnemyResource'
    operator: '=='
    right: '1'
  targetSlowableCondition:
    left: '@slowLockResource'
    operator: '=='
    right: '0'

Parameter:
  slowLockParameter:
    key: resource
    value: slowLockResource
  slowSpeedParameter:
    key: speed
    value: '0.5'
  slowSecondsParameter:
    key: duration
    value: '4'
  slowLockSecondsParameter:
    key: value
    value: '5'
  oneExpressionParameter:
    key: expression
    value: '1'

Action:
  frostBoltAction:
    name: Frost Bolt
    time: 1.0
    if: targetIsEnemyCondition
    then: frostBoltSlowAction
  frostBoltSlowAction:
    if: targetSlowableCondition
    do: temp_speed_target
    parameters: [slowSpeedParameter, slowSecondsParameter]
    then: frostBoltLockAction
  frostBoltLockAction:
    do: set_resource_target
    parameters: [slowLockParameter, slowLockSecondsParameter]
  slowLockTickAction:
    do: minus_resource_self
    parameters: [slowLockParameter, oneExpressionParameter]

Timer:
  slowLockTickTimer:
    interval: '1'
    total: '0'
    action: slowLockTickAction

Skill:
  frostBoltSkill:
    name: Frost Bolt
    description: Slows the target to half speed for 4 seconds.
    start: frostBoltAction

Actor:
  mageActor:
    name: Mage
    sprite: heroSprite
    skills: [frostBoltSkill]
  ogreActor:
    name: Ogre
    sprite: wolfSprite
    speed: 1.0
    timers: [slowLockTickTimer]
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Skill`](/reference/entities.html#skill), [`Timer`](/reference/entities.html#timer)

**Gotchas.**

- `speed` is a multiplier: `1.0` is normal, `0.5` half, `0` stopped.
- Make the lock a second longer than the slow, so the speed is always back to normal before another slow can land.
- Put the lock on the **target**, not the caster. Two mages slowing the same ogre must share one lock.
- `set_speed_*` changes speed for good; use it for a permanent change, not a timed effect.
- Damage goes in the same chain; the slow is only the movement part.
