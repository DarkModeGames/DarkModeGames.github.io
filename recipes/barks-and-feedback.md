# Ambient barks and failure feedback

**Problem.** Townsfolk should say something now and then when a player walks by, and a refused action should tell
the player why, without filling the screen with text.

**Pattern.** `echo_self` floats a short line above the caller. Its `audience` decides who sees it:

- `public` (the default): every player who can see the speaker. Use it for **barks**. Fire them from `on_view`
  (when a player comes into the NPC's sight), gated by a cooldown resource so they stay rare.
- `private`: only the caller's own player. Use it for **failure feedback** ("Out of range"), so other players
  aren't spammed with your mistakes.

`duration` sets how long the line stays up, in seconds (0.2 to 30; default 1.6, the same as combat numbers). Give
a bark a few seconds so it can be read; keep refusals short.

```yaml
Resource:
  barkCooldownResource:
    name: Bark Cooldown
    default: 0
    min: '0'
    max: '120'
    reveal: 999999

Condition:
  barkReadyCondition:
    left: '$barkCooldownResource'
    operator: '=='
    right: '0'
  inSpellRangeCondition:
    left: '$distance_to_target'
    operator: '<='
    right: '260'

Parameter:
  barkCooldownParameter:
    key: resource
    value: barkCooldownResource
  barkCooldownValueParameter:
    key: value
    value: '60'
  oneExpressionParameter:
    key: expression
    value: '1'
  barkMessageParameter:
    key: message
    value: 'Mind the bog after dark.'
  outOfRangeMessageParameter:
    key: message
    value: 'Out of range'
  privateAudienceParameter:
    key: audience
    value: private
  publicAudienceParameter:
    key: audience
    value: public
  barkDurationParameter:
    key: duration
    value: '4'

Action:
  # Villager: on seeing a player, bark if not barked for a minute.
  villagerBarkAction:
    if: barkReadyCondition
    do: echo_self
    parameters: [barkMessageParameter, publicAudienceParameter, barkDurationParameter]
    then: villagerBarkCooldownAction
  villagerBarkCooldownAction:
    do: set_resource_self
    parameters: [barkCooldownParameter, barkCooldownValueParameter]
  barkTickAction:
    do: minus_resource_self
    parameters: [barkCooldownParameter, oneExpressionParameter]
  # Player skill: a range check with a private refusal.
  boltAction:
    name: Bolt
    time: 1.0
    if: inSpellRangeCondition
    then: boltHitAction
    else: outOfRangeAction
  boltHitAction:
    do: wait
  outOfRangeAction:
    do: echo_self
    parameters: [outOfRangeMessageParameter, privateAudienceParameter]

Timer:
  barkTickTimer:
    interval: '1'
    total: '0'
    action: barkTickAction

Actor:
  villagerActor:
    name: Villager
    sprite: villagerSprite
    perception: 160
    on_view: villagerBarkAction
    timers: [barkTickTimer]
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Timer`](/reference/entities.html#timer)

**Gotchas.**

- `on_view` runs on the NPC (caller) with the player it saw as the target, and only for a player's own character,
  so NPCs seeing each other never bark.
- Keep barks rare (a long cooldown) and short. Never echo on every hit; use
  [combat text](/recipes/combat-text.html) for numbers.
- A floating line belongs to the actor it floats from. If that actor is despawned or changes form, its text goes
  with it.
- A private echo goes to the **caller's** player. When the caller is an NPC (a bark, a gate's `on_touch`), a
  private line reaches nobody. Use public for NPC lines, or a dialog.
- A public line reaches only players who can see the speaker.
- `echo` (no `_self`) only writes to the log. It is useful for debugging.
- `color` takes a hex code (`'#ffd040'`) or a colour name.
