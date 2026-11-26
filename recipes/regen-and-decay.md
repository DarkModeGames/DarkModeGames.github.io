# Regenerate mana, decay rage

**Problem.** A caster's mana should refill slowly, and a fighter's rage should build on hits and drain away when
the fighting stops.

**Pattern.** Each class resource gets its own repeating [`Timer`](/reference/entities.html#timer) on the actor: one adds to mana, one takes from
rage. The fighter's hit action adds rage with a `then` step. The resource's bounds keep it in range, so a full
pool or an empty one needs no special case.

```yaml
Resource:
  healthResource:
    name: Health
    default: 100
    min: '0'
    max: '100'
  manaResource:
    name: Mana
    default: 50
    min: '0'
    max: '50'
  rageResource:
    name: Rage
    default: 0
    min: '0'
    max: '100'
  levelResource:
    name: Level
    default: 1
    min: '1'
    max: '20'

Parameter:
  healthParameter:
    key: resource
    value: healthResource
  manaParameter:
    key: resource
    value: manaResource
  rageParameter:
    key: resource
    value: rageResource
  manaRegenParameter:
    key: expression
    value: '2+$levelResource'
  rageDecayParameter:
    key: expression
    value: '3'
  rageGainParameter:
    key: expression
    value: '8'
  hitDamageParameter:
    key: expression
    value: '1d6+2'

Action:
  manaRegenAction:
    do: plus_resource_self
    parameters: [manaParameter, manaRegenParameter]
  rageDecayAction:
    do: minus_resource_self
    parameters: [rageParameter, rageDecayParameter]
  # A fighter's blow: damage, then rage.
  rageHitAction:
    do: minus_resource_target
    parameters: [healthParameter, hitDamageParameter]
    then: rageGainAction
  rageGainAction:
    do: plus_resource_self
    parameters: [rageParameter, rageGainParameter]

Timer:
  manaRegenTimer:
    interval: '2'
    total: '0'
    action: manaRegenAction
  rageDecayTimer:
    interval: '1'
    total: '0'
    action: rageDecayAction

Actor:
  casterActor:
    name: Caster
    sprite: heroSprite
    timers: [manaRegenTimer]
    private: [healthResource, manaResource]
  fighterActor:
    name: Fighter
    sprite: heroSprite
    timers: [rageDecayTimer]
    private: [healthResource, rageResource]
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Timer`](/reference/entities.html#timer)

**Gotchas.**

- A Timer runs with its owner as caller and the owner's current target as target. Use `*_self` actions in timers
  unless you mean the target.
- The first tick comes one `interval` after the actor spawns.
- Timers keep running while the actor casts or is rooted; a skill never pauses them.
- Only actors that list the Timer get it, so a caster never gains rage.
