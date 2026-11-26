# Talents unlocked at a level

**Problem.** At level 6 the player should choose one of two talents, and the choice should put a new skill on
their hotbar and stay chosen.

**Pattern.** Two resources per talent tier:

- `tier6OpenResource` (shown with `reveal: 1`, with a `menu`) means "you may choose now";
- `tier6ChoiceResource` (hidden) records the choice: 0 none, 1 or 2.

A [`Trigger`](/reference/entities.html#trigger) on `levelResource` opens the tier when the level is reached and nothing is chosen. The player opens
the resource from their inventory. Each menu option records the choice, closes the tier, and uses
`set_skill_self` to put the granted [`Skill`](/reference/entities.html#skill) into hotbar slot 8.

```yaml
Resource:
  levelResource:
    name: Level
    default: 1
    min: '1'
    max: '10'
  tier6OpenResource:
    name: 'Talent: choose one'
    description: You have earned a talent. Open this to choose.
    default: 0
    min: '0'
    max: '1'
    reveal: 1
    menu: tier6Menu
  tier6ChoiceResource:
    name: Level 6 Talent
    default: 0
    min: '0'
    max: '2'
    reveal: 999999

Condition:
  reachedLevel6Condition:
    left: '$levelResource'
    operator: '>='
    right: '6'
  tier6UnchosenCondition:
    left: '$tier6ChoiceResource'
    operator: '=='
    right: '0'
  tier6OpenCondition:
    left: '$tier6OpenResource'
    operator: '=='
    right: '1'

Parameter:
  tier6OpenParameter:
    key: resource
    value: tier6OpenResource
  tier6ChoiceParameter:
    key: resource
    value: tier6ChoiceResource
  oneValueParameter:
    key: value
    value: '1'
  twoValueParameter:
    key: value
    value: '2'
  zeroValueParameter:
    key: value
    value: '0'
  whirlwindSkillParameter:
    key: skill
    value: whirlwindSkill
  shieldWallSkillParameter:
    key: skill
    value: shieldWallSkill
  slotEightParameter:
    key: slot
    value: '8'

Action:
  # Level Trigger: open tier 6 once it is reached and still unchosen.
  tier6UnlockAction:
    if: reachedLevel6Condition
    then: tier6UnlockCheckAction
  tier6UnlockCheckAction:
    if: tier6UnchosenCondition
    do: set_resource_self
    parameters: [tier6OpenParameter, oneValueParameter]
  # Menu option 1.
  chooseWhirlwindAction:
    name: 'Fury: Whirlwind'
    if: tier6OpenCondition
    do: close_interaction
    then: chooseWhirlwindRecordAction
  chooseWhirlwindRecordAction:
    do: set_resource_self
    parameters: [tier6ChoiceParameter, oneValueParameter]
    then: chooseWhirlwindCloseTierAction
  chooseWhirlwindCloseTierAction:
    do: set_resource_self
    parameters: [tier6OpenParameter, zeroValueParameter]
    then: chooseWhirlwindGrantAction
  chooseWhirlwindGrantAction:
    do: set_skill_self
    parameters: [whirlwindSkillParameter, slotEightParameter]
  # Menu option 2.
  chooseShieldWallAction:
    name: 'Guard: Shield Wall'
    if: tier6OpenCondition
    do: close_interaction
    then: chooseShieldWallRecordAction
  chooseShieldWallRecordAction:
    do: set_resource_self
    parameters: [tier6ChoiceParameter, twoValueParameter]
    then: chooseShieldWallCloseTierAction
  chooseShieldWallCloseTierAction:
    do: set_resource_self
    parameters: [tier6OpenParameter, zeroValueParameter]
    then: chooseShieldWallGrantAction
  chooseShieldWallGrantAction:
    do: set_skill_self
    parameters: [shieldWallSkillParameter, slotEightParameter]
  whirlwindAction:
    name: Whirlwind
    time: 1.0
    do: wait
  shieldWallAction:
    name: Shield Wall
    time: 1.0
    do: wait

Trigger:
  tier6Trigger:
    resource: levelResource
    action: tier6UnlockAction

Menu:
  tier6Menu:
    name: Choose a level 6 talent
    actions: [chooseWhirlwindAction, chooseShieldWallAction]

Skill:
  whirlwindSkill:
    name: Whirlwind
    description: Strike everything around you.
    start: whirlwindAction
  shieldWallSkill:
    name: Shield Wall
    description: Halve the damage you take for a while.
    start: shieldWallAction

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    triggers: [tier6Trigger]
    private: [levelResource]
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Menu`](/reference/entities.html#menu), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Skill`](/reference/entities.html#skill), [`Trigger`](/reference/entities.html#trigger)

**Gotchas.**

- `set_skill_self` hot-swaps one hotbar slot (1–9). The swap is saved with the player and kept through
  `change_actor`, for players on a client as well as the host, so the talent skill is still there after a restart.
  A swap made on the actor's first frame (a spawn hook, `on_start`) shows on the hotbar straight away.
- The level Trigger also fires on the actor's first frame. With the "unchosen" guard, a player who logs in at
  level 8 without having chosen gets the tier opened then.
- Re-check `tier6OpenCondition` in each option. The menu doesn't refresh, so a second click must do nothing.
- Passive talents need no skill: record the choice and branch on it with `if` inside the skills it changes.
- Replace `do: wait` with the real effect; it is only a placeholder here.
