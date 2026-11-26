# A safe zone monsters won't enter

**Problem.** Monsters that chased a player across the map end up camping the graveyard or the town gate, and kill
players as they arrive.

**Pattern.** Place an invisible **ward** actor at every graveyard and arrival point. Once a second, its [Timer](/reference/entities.html#timer) runs
`area_of_effect_at_self`. Each enemy inside the ring gets `wardedResource` set to `1`. Put a "warded" behavior
first in the monster's [strategy](/recipes/mob-ai-strategy.html): while warded, the monster steps away from its
target, drops it, clears the flag and waits a few seconds.

```yaml
Resource:
  isEnemyResource:
    name: Is Enemy
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  wardedResource:
    name: Warded
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  evadeResource:
    name: Evading
    default: 0
    min: '0'
    max: '10'
    reveal: 999999

Condition:
  targetIsEnemyCondition:
    left: '@isEnemyResource'
    operator: '=='
    right: '1'
  notWardedCondition:
    left: '$wardedResource'
    operator: '=='
    right: '0'

Parameter:
  wardedParameter:
    key: resource
    value: wardedResource
  evadeParameter:
    key: resource
    value: evadeResource
  oneValueParameter:
    key: value
    value: '1'
  zeroValueParameter:
    key: value
    value: '0'
  evadeValueParameter:
    key: value
    value: '5'
  wardRadiusParameter:
    key: radius
    value: '130'
  wardEachParameter:
    key: action
    value: wardEachAction
  retreatDistanceParameter:
    key: distance
    value: '192'

Action:
  # Ward, every second.
  wardPulseAction:
    do: area_of_effect_at_self
    parameters: [wardRadiusParameter, wardEachParameter]
  wardEachAction:
    if: targetIsEnemyCondition
    do: set_resource_target
    parameters: [wardedParameter, oneValueParameter]
  # Monster, while warded.
  wardRetreatAction:
    do: move_away_from_target
    parameters: [retreatDistanceParameter]
    then: wardDropTargetAction
  wardDropTargetAction:
    do: clear_target_self
    then: wardClearAction
  wardClearAction:
    do: set_resource_self
    parameters: [wardedParameter, zeroValueParameter]
    then: wardEvadeAction
  wardEvadeAction:
    do: set_resource_self
    parameters: [evadeParameter, evadeValueParameter]

Timer:
  wardTimer:
    interval: '1'
    total: '0'
    action: wardPulseAction

Behavior:
  wardedBehavior:
    goals: [notWardedCondition]
    action: wardRetreatAction

Actor:
  lampWardActor:
    name: Lamp-Ward
    targetable: false
    base: 0
    perception: 0
    salience: 1
    timers: [wardTimer]

Vector:
  graveyardVector:
    x: 200
    y: 8

Deployment:
  lampWardDeployment:
    actor: lampWardActor
    location: graveyardVector
```

Add `wardedBehavior` as the **first** behavior of each monster strategy, and deploy the ward on your map.

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Behavior`](/reference/entities.html#behavior), [`Condition`](/reference/entities.html#condition), [`Deployment`](/reference/entities.html#deployment), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Timer`](/reference/entities.html#timer), [`Vector`](/reference/entities.html#vector)

**Gotchas.**

- The ward has no sprite and `targetable: false`: it is never drawn as anything and never targeted. A lit lamp
  prop on the same spot shows players where the safe ring is.
- The ring is an ellipse: radius 130 reaches 130 px to the sides and 65 px up or down. Make it a little bigger
  than the area players arrive in.
- `move_away_from_target` needs the target, so it runs before `clear_target_self` in the chain.
- The evade wait (see [Monster AI](/recipes/mob-ai-strategy.html)) keeps the monster from turning straight back.
- Area effects reach every actor, so the `if` on the enemy tag is what keeps players and props out of it.
