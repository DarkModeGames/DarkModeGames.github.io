# Skill tooltips with real numbers, and greyed-out skills

**Problem.** A skill's tooltip should say how much it hits for *with this player's stats*, as Diablo does, and the
hotbar should show at a glance which skills can't be used right now.

**Pattern.** In a Skill's `description`, `{% raw %}{{range: <dice>}}{% endraw %}` shows the lowest and highest result with the
reader's own values filled in ("12–17"), and `{% raw %}{{dice: <dice>}}{% endraw %}` shows the formula with the stat-driven part
filled in and highlighted ("1d6+11"). `{% raw %}{{$key}}{% endraw %}` still shows a single value. For the hotbar, `ready` names a
Condition on the player (cost and cooldown) and `in_range` a Condition from the player to the current target.
While `ready` fails the slot is darkened; while a target is held and `in_range` fails it is darkened too.

{% raw %}
```yaml
Resource:
  healthResource:
    name: Health
    default: 100
    min: '0'
    max: '100'
  rageResource:
    name: Rage
    default: 0
    min: '0'
    max: '100'
  strengthResource:
    name: Strength
    default: 10
    min: '0'
    max: '30'
  smiteCooldownResource:
    name: Smite (cooldown)
    default: 0
    min: '0'
    max: '8'
    reveal: 1

Condition:
  smiteReadyCondition:
    left: '$rageResource'
    operator: '>='
    right: '20'
    all: [smiteOffCooldownCondition]
  smiteOffCooldownCondition:
    left: '$smiteCooldownResource'
    operator: '=='
    right: '0'
  smiteInRangeCondition:
    left: '$distance_to_target'
    operator: '<='
    right: '48'

Parameter:
  healthParameter:
    key: resource
    value: healthResource
  smiteDamageParameter:
    key: expression
    value: '2d6+$strengthResource'

Action:
  smiteAction:
    name: Smite
    time: 1.0
    if: smiteReadyCondition
    do: minus_resource_target
    parameters: [healthParameter, smiteDamageParameter]

Skill:
  smiteSkill:
    name: Smite
    description: 'Deals {{range: 2d6+$strengthResource}} damage ({{dice: 2d6+$strengthResource}}). Costs 20 Rage.'
    start: smiteAction
    ready: smiteReadyCondition
    in_range: smiteInRangeCondition

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    skills: [smiteSkill]
```
{% endraw %}

**Entities used.** [`Skill`](/reference/entities.html#skill), [`Condition`](/reference/entities.html#condition), [`Action`](/reference/entities.html#action), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Actor`](/reference/entities.html#actor)

**Gotchas.**

- Copy the same dice into the description as into the damage parameter, so the tooltip never lies.
- `all` on a Condition means every listed Condition must also hold: one `ready` Condition can check both the
  cost and the cooldown.
- `ready` and `in_range` only change how the slot looks. The action's own `if` still decides whether the skill
  fires, so keep the checks there too ([Skill cooldowns](/recipes/skill-cooldowns.html)).
- `$distance_to_target` is in isometric pixels (up and down count double); see
  [Distances and areas](/recipes/distances-and-areas.html).
