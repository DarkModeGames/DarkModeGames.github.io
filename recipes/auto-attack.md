# Auto-attack: right-click an enemy to fight it

**Problem.** In a WoW-style game, right-clicking an enemy starts swinging at it, and the swings keep coming until
you stop, the enemy dies, or you walk away.

**Pattern.** Give the player's actor an `auto_attack` skill, an `auto_attack_interval` (seconds between swings) and
an `auto_attack_range` (how close it must be, in isometric pixels; 0 means bases touching). Give each enemy an
`on_interact` action that runs `start_auto_attack`. Right-clicking the enemy (or the Interact key, or clicking its
plate icon) then runs that action with the player as caller and the enemy as target, instead of walking up to
talk. The player walks into range and swings until the target changes, dies or leaves, or the player moves by
hand. A skill cast in between does not reset the swing clock.

{% raw %}
```yaml
Resource:
  healthResource:
    name: Health
    default: 50
    min: '0'
    max: '50'

Parameter:
  healthParameter:
    key: resource
    value: healthResource
  swingDamageParameter:
    key: expression
    value: '1d6+2'

Action:
  swingAction:
    name: Attack
    time: 0.5
    do: minus_resource_target
    parameters: [healthParameter, swingDamageParameter]
  attackItAction:
    do: start_auto_attack

Skill:
  swingSkill:
    name: Attack
    description: 'A plain swing: {{range: 1d6+2}} damage.'
    start: swingAction

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    auto_attack: swingSkill
    auto_attack_interval: 2.0
    auto_attack_range: 0
  wolfActor:
    name: Grey Wolf
    sprite: wolfSprite
    on_interact: attackItAction
```
{% endraw %}

**Entities used.** [`Actor`](/reference/entities.html#actor), [`Skill`](/reference/entities.html#skill), [`Action`](/reference/entities.html#action), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource)

**Gotchas.**

- `on_interact` replaces the walk-up dialog, so give it only to actors you fight. NPCs keep their `menu`.
- The swing is the skill's `start` action, run as a skill: its `if`, `time` and animation all apply. Put range or
  "is alive" checks in that action's `if` as usual.
- `stop_auto_attack_self` ends it from an action (for example when the player opens a vendor).
- A ranged class can set `auto_attack_range` to its bow's reach so it stops and fires from there.
