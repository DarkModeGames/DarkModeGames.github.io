# Energy and combo points

**Problem.** A rogue should spend fast-refilling energy on quick jabs that build combo points, then spend all the
points at once on a finisher that hits harder the more it spends.

**Pattern.** `energyResource` refills by 10 every second from a [Timer](/reference/entities.html#timer). A builder costs energy and adds one combo
point (capped at 5 by the bounds). A finisher needs at least one point, deals damage scaled by `$comboResource`, then
sets combo to 0. Every check comes before the first change, so a refused press spends nothing.

```yaml
Resource:
  healthResource:
    name: Health
    default: 100
    min: '0'
    max: '100'
  energyResource:
    name: Energy
    default: 100
    min: '0'
    max: '100'
  comboResource:
    name: Combo
    description: Spent by finishers.
    default: 0
    min: '0'
    max: '5'
    reveal: 1

Condition:
  jabAffordCondition:
    left: '$energyResource'
    operator: '>='
    right: '40'
  finisherAffordCondition:
    left: '$energyResource'
    operator: '>='
    right: '35'
  hasComboCondition:
    left: '$comboResource'
    operator: '>='
    right: '1'

Parameter:
  healthParameter:
    key: resource
    value: healthResource
  energyParameter:
    key: resource
    value: energyResource
  comboParameter:
    key: resource
    value: comboResource
  jabCostParameter:
    key: expression
    value: '40'
  finisherCostParameter:
    key: expression
    value: '35'
  energyRegenParameter:
    key: expression
    value: '10'
  oneExpressionParameter:
    key: expression
    value: '1'
  jabDamageParameter:
    key: expression
    value: '1d6+2'
  finisherDamageParameter:
    key: expression
    value: '$comboResource*8+5'
  zeroValueParameter:
    key: value
    value: '0'

Action:
  jabAction:
    name: Jab
    time: 0.8
    if: jabAffordCondition
    do: minus_resource_self
    parameters: [energyParameter, jabCostParameter]
    then: jabHitAction
  jabHitAction:
    do: minus_resource_target
    parameters: [healthParameter, jabDamageParameter]
    then: jabComboAction
  jabComboAction:
    do: plus_resource_self
    parameters: [comboParameter, oneExpressionParameter]
  finisherAction:
    name: Last Word
    time: 1.0
    if: hasComboCondition
    then: finisherPayAction
  finisherPayAction:
    if: finisherAffordCondition
    do: minus_resource_self
    parameters: [energyParameter, finisherCostParameter]
    then: finisherHitAction
  finisherHitAction:
    do: minus_resource_target
    parameters: [healthParameter, finisherDamageParameter]
    then: finisherSpendAction
  finisherSpendAction:
    do: set_resource_self
    parameters: [comboParameter, zeroValueParameter]
  energyRegenAction:
    do: plus_resource_self
    parameters: [energyParameter, energyRegenParameter]

Timer:
  energyRegenTimer:
    interval: '1'
    total: '0'
    action: energyRegenAction

Skill:
  jabSkill:
    name: Jab
    description: A quick cut. Builds 1 combo point.
    start: jabAction
  finisherSkill:
    name: Last Word
    description: Spends every combo point for 8 damage each.
    start: finisherAction

Actor:
  rogueActor:
    name: Rogue
    sprite: heroSprite
    skills: [jabSkill, finisherSkill]
    timers: [energyRegenTimer]
    private: [healthResource, energyResource, comboResource]
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Skill`](/reference/entities.html#skill), [`Timer`](/reference/entities.html#timer)

**Gotchas.**

- `$` in the damage expression is the caster, so `$comboResource` is the rogue's points, read **before** the next
  step sets them to 0.
- A builder at 5 points still costs energy and adds nothing. Warn with a private "Combo full" if you like.
- Combo points belong to the rogue, not the target. Switching targets keeps them.
- Add target, range and cooldown checks in front of the cost, as in [Give a skill a cooldown](/recipes/skill-cooldowns.html).
