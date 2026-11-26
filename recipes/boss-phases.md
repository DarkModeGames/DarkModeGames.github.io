# Boss phases: a half-health line, adds and an enrage clock

**Problem.** A boss fight should change as it goes: a line at half health, helpers joining, and a hard limit
that punishes a fight that drags on.

**Pattern.** Three independent pieces on the boss:

- **Half line.** A [`Trigger`](/reference/entities.html#trigger) on `healthResource` runs once when health falls to half or less: a `halfDoneResource`
  flag guards it. It speaks a public line and spawns two adds beside the boss with `spawn_actor_at_self_radial`.
- **Fight clock.** A 1-second [`Timer`](/reference/entities.html#timer) adds a second to `fightClockResource` while the boss is engaged
  (`engagedResource` 1, set when it first picks a target).
- **Enrage.** The same tick, once the clock passes 300, adds attack every 10 seconds.

```yaml
Resource:
  healthResource:
    name: Health
    default: 3000
    min: '0'
    max: '3000'
  attackResource:
    name: Attack
    default: 30
    min: '0'
    max: '999'
    reveal: 999999
  halfDoneResource:
    name: Half Line Spoken
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  engagedResource:
    name: Engaged
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  fightClockResource:
    name: Fight Clock
    default: 0
    min: '0'
    max: '9999'
    reveal: 999999

Condition:
  belowHalfCondition:
    left: '$healthResource*2'
    operator: '<='
    right: '3000'
  halfNotDoneCondition:
    left: '$halfDoneResource'
    operator: '=='
    right: '0'
  engagedCondition:
    left: '$engagedResource'
    operator: '=='
    right: '1'
  enragedCondition:
    left: '$fightClockResource'
    operator: '>='
    right: '300'
  enrageBeatCondition:
    left: '$fightClockResource%10'
    operator: '=='
    right: '0'

Parameter:
  halfDoneParameter:
    key: resource
    value: halfDoneResource
  fightClockParameter:
    key: resource
    value: fightClockResource
  attackParameter:
    key: resource
    value: attackResource
  oneValueParameter:
    key: value
    value: '1'
  oneExpressionParameter:
    key: expression
    value: '1'
  enrageStepParameter:
    key: expression
    value: '4'
  halfLineParameter:
    key: message
    value: 'You have not even made me angry yet.'
  halfLineDurationParameter:
    key: duration
    value: '5'
  addActorParameter:
    key: actor
    value: boneHoundActor
  leftParameter:
    key: radial
    value: '90'
  rightParameter:
    key: radial
    value: '270'
  addDistanceParameter:
    key: distance
    value: '56'

Action:
  # Half line (health Trigger): once.
  bossHalfAction:
    if: belowHalfCondition
    then: bossHalfOnceAction
  bossHalfOnceAction:
    if: halfNotDoneCondition
    do: set_resource_self
    parameters: [halfDoneParameter, oneValueParameter]
    then: bossHalfSpeakAction
  bossHalfSpeakAction:
    do: echo_self
    parameters: [halfLineParameter, halfLineDurationParameter]
    then: bossAddLeftAction
  bossAddLeftAction:
    do: spawn_actor_at_self_radial
    parameters: [addActorParameter, leftParameter, addDistanceParameter]
    then: bossAddRightAction
  bossAddRightAction:
    do: spawn_actor_at_self_radial
    parameters: [addActorParameter, rightParameter, addDistanceParameter]
  # Clock (1 s Timer): count while engaged, enrage after 300 s.
  bossClockAction:
    if: engagedCondition
    do: plus_resource_self
    parameters: [fightClockParameter, oneExpressionParameter]
    then: bossEnrageAction
  bossEnrageAction:
    if: enragedCondition
    then: bossEnrageBeatAction
  bossEnrageBeatAction:
    if: enrageBeatCondition
    do: plus_resource_self
    parameters: [attackParameter, enrageStepParameter]

Trigger:
  bossHalfTrigger:
    resource: healthResource
    action: bossHalfAction

Timer:
  bossClockTimer:
    interval: '1'
    total: '0'
    action: bossClockAction

Actor:
  bossActor:
    name: The Warden
    sprite: wolfSprite
    triggers: [bossHalfTrigger]
    timers: [bossClockTimer]
    public: [healthResource]
  boneHoundActor:
    name: Bone Hound
    sprite: wolfSprite
```

Set `engagedResource` to `1` in the boss's acquire action (see [Monster AI](/recipes/mob-ai-strategy.html)), and
back to `0`, with the clock and the flag, when it leashes, so a reset fight starts clean.

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Timer`](/reference/entities.html#timer), [`Trigger`](/reference/entities.html#trigger)

**Gotchas.**

- Don't compare against `max` in a [Condition](/reference/entities.html#condition): write the number, or use a [Measure](/reference/entities.html#measure) for maximum health and compare
  `'$healthResource*2'` to `'$maxHealthMeasure'`.
- `%` is remainder, so `$fightClockResource%10 == 0` is true once every 10 seconds.
- A health Trigger fires on every hit. The flag makes the line run once, even if health dips below half many
  times.
- `spawn_actor_at_self_radial` places the add at an angle from where the boss faces, `distance` iso px away. Adds
  need their own AI and spawn hook.
- A public `echo_self` reaches only players who can see the boss. Give a boss line a longer `duration` so it can
  be read mid-fight.
