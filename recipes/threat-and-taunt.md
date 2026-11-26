# Threat-based targeting and taunt

**Problem.** A monster should attack whoever is doing the most to it, and a tank should be able to pull it back.

**Pattern.** Each player has a `threatResource` that their attacks raise and a [Timer](/reference/entities.html#timer) slowly lowers. Every few
seconds the monster runs `target_highest_measure` on a **retarget score**. A plain "highest threat" pick swaps to
the first of any tied players. So the monster briefly marks its current target with a `holdResource`, and the
score is `threat*2 + hold`: a challenger must have strictly more threat to win. A taunt uses
`set_target_target_to_self`: the monster now targets the tank.

```yaml
Resource:
  threatResource:
    name: Threat
    default: 0
    min: '0'
    max: '999999'
    reveal: 999999
  holdResource:
    name: Retarget Hold
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  isPlayerResource:
    name: Is Player
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  isEnemyResource:
    name: Is Enemy
    default: 0
    min: '0'
    max: '1'
    reveal: 999999

Measure:
  retargetScoreMeasure:
    name: Retarget Score
    expression: '$threatResource*2+$holdResource'
    reveal: 999999

Condition:
  targetIsPlayerCondition:
    left: '@isPlayerResource'
    operator: '=='
    right: '1'
  targetIsEnemyCondition:
    left: '@isEnemyResource'
    operator: '=='
    right: '1'

Parameter:
  threatParameter:
    key: resource
    value: threatResource
  holdParameter:
    key: resource
    value: holdResource
  retargetScoreParameter:
    key: measure
    value: retargetScoreMeasure
  oneValueParameter:
    key: value
    value: '1'
  zeroValueParameter:
    key: value
    value: '0'
  threatDecayParameter:
    key: expression
    value: '$threatResource/10+1'
  tauntThreatParameter:
    key: expression
    value: '50'

Action:
  # Monster, every 2 s: hold the current target, pick the best score, release the hold.
  retargetAction:
    if: targetIsPlayerCondition
    do: set_resource_target
    parameters: [holdParameter, oneValueParameter]
    then: retargetPickAction
  retargetPickAction:
    do: target_highest_measure
    parameters: [retargetScoreParameter]
    then: retargetReleaseAction
  retargetReleaseAction:
    do: set_resource_target
    parameters: [holdParameter, zeroValueParameter]
  # Player, every 2 s: threat fades.
  threatDecayAction:
    do: minus_resource_self
    parameters: [threatParameter, threatDecayParameter]
  # Tank skill: "Look at me", plus a lump of threat so the next retarget agrees.
  tauntAction:
    name: Taunt
    time: 0.5
    if: targetIsEnemyCondition
    do: set_target_target_to_self
    then: tauntThreatAction
  tauntThreatAction:
    do: plus_resource_self
    parameters: [threatParameter, tauntThreatParameter]

Timer:
  retargetTimer:
    interval: '2'
    total: '0'
    action: retargetAction
  threatDecayTimer:
    interval: '2'
    total: '0'
    action: threatDecayAction

Skill:
  tauntSkill:
    name: Taunt
    description: Force the target to attack you.
    start: tauntAction

Actor:
  tankActor:
    name: Tank
    sprite: heroSprite
    skills: [tauntSkill]
    timers: [threatDecayTimer]
  ogreActor:
    name: Ogre
    sprite: wolfSprite
    timers: [retargetTimer]
```

Every player attack should end with a `plus_resource_self` on `threatParameter`, for example the damage dealt.

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Measure`](/reference/entities.html#measure), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Skill`](/reference/entities.html#skill), [`Timer`](/reference/entities.html#timer)

**Gotchas.**

- `target_highest_measure` looks only at targetable actors in the monster's view, and works the [Measure](/reference/entities.html#measure) out on
  each candidate, so `$` in it means "that candidate".
- In a `then` chain the target stays the one the chain started with. So the release step clears the hold on the
  **old** target, even though the pick just changed it. The new target is used from the next action.
- Threat here is one number per player, shared by every monster that sees them. That is simple and usually good
  enough. Zero it when the player dies.
- `set_target_target_to_self_target` is the sibling action: the target takes the **caller's** target. Use it for
  assists. See [Pack call for help](/recipes/pack-call-for-help.html).
