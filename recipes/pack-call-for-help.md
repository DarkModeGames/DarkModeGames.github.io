# Pack call for help

**Problem.** When a player pulls one wolf from a pack, the idle wolves beside it should join in.

**Pattern.** Add one behavior to the monster's [strategy](/recipes/mob-ai-strategy.html), right after acquire:
"call for help". Its goal is "I have called", so it runs once per fight. Its action marks the call as made, then
runs `area_of_effect_at_self` over its packmates. For each one that is an enemy, of the same pack, and still idle,
the sub-action runs `set_target_target_to_self_target`: the helper takes the caller's target.

```yaml
Resource:
  isEnemyResource:
    name: Is Enemy
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  packResource:
    name: Pack
    default: 0
    min: '0'
    max: '99'
    reveal: 999999
  calledResource:
    name: Called for Help
    default: 0
    min: '0'
    max: '1'
    reveal: 999999

Condition:
  calledCondition:
    left: '$calledResource'
    operator: '=='
    right: '1'
  targetIsEnemyCondition:
    left: '@isEnemyResource'
    operator: '=='
    right: '1'
  targetSamePackCondition:
    left: '@packResource'
    operator: '=='
    right: '$packResource'
  targetIdleCondition:
    left: '@calledResource'
    operator: '=='
    right: '0'

Parameter:
  calledParameter:
    key: resource
    value: calledResource
  oneValueParameter:
    key: value
    value: '1'
  callRadiusParameter:
    key: radius
    value: '200'
  callEachParameter:
    key: action
    value: callEachAction

Action:
  callAction:
    do: set_resource_self
    parameters: [calledParameter, oneValueParameter]
    then: callAreaAction
  callAreaAction:
    do: area_of_effect_at_self
    parameters: [callRadiusParameter, callEachParameter]
  callEachAction:
    if: targetIsEnemyCondition
    then: callEachPackAction
  callEachPackAction:
    if: targetSamePackCondition
    then: callEachIdleAction
  callEachIdleAction:
    if: targetIdleCondition
    do: set_target_target_to_self_target

Behavior:
  callBehavior:
    goals: [calledCondition]
    action: callAction
```

Set `packResource` in each pack member's [spawn hook](/recipes/spawn-hook.html) (wolves `1`, bandits `2`, ...),
and set `calledResource` back to `0` in the leash action, so a reset pack can call again.

**Entities used.** [`Action`](/reference/entities.html#action), [`Behavior`](/reference/entities.html#behavior), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource)

**Gotchas.**

- An action with an `if` and a `then` but no `do` is a plain branch. Chaining several branches is how you write
  "and".
- The caller is inside its own area, but it has already set `calledResource`, so the idle check skips it.
- `set_target_target_to_self_target` does nothing when the caller has no target, or when the caller's target is
  the actor being changed.
- The helper then reaches its own call behavior and calls its own neighbours: the alarm spreads through the pack.
- Use `0` for "never calls" (bosses, summons), and test `$packResource > 0` before calling if you mix them in.
- The radius is in isometric pixels: 200 reaches 200 px to the sides but only 100 px up or down.
