# Respawn a player on death

**Problem.** When a player's health reaches 0 they should visibly fall, lie still for a few seconds with a
countdown, and then rise at a graveyard, even if they died on another map.

**Pattern.** Three pieces:

1. **Fall.** A [`Trigger`](/reference/entities.html#trigger) on `healthResource`, guarded by "health is 0", untags the player (so monsters, heals and
   pickups ignore them), starts a countdown, and swaps the body for a **fallen** actor with `change_actor_self`.
   The fallen actor uses a non-looping death animation, has speed `0`, no skills, and one [Timer](/reference/entities.html#timer).
2. **Countdown.** The fallen actor's 1-second Timer says "You have fallen." on its first tick, then counts
   `fallenTimerResource` down. When it reaches 0 it refills health and moves to the graveyard:
   `teleport_self_to_vector` if the player is already on the graveyard's map, otherwise `change_map_self`.
3. **Rise.** On the next tick it retags the player and swaps back to the hero with `change_actor_self`.

Resources carry over through `change_actor_*`, so the timer, health and gold survive each swap.

```yaml
Resource:
  healthResource:
    name: Health
    default: 100
    min: '0'
    max: '100'
  isPlayerResource:
    name: Is Player
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  zoneResource:
    name: Zone
    default: 1
    min: '0'
    max: '99'
    reveal: 999999
  justFellResource:
    name: Just Fell
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  fallenTimerResource:
    name: Fallen
    description: Seconds until you rise at the graveyard.
    default: 0
    min: '0'
    max: '60'
    reveal: 1

Condition:
  deadCondition:
    left: '$healthResource'
    operator: '<='
    right: '0'
  justFellCondition:
    left: '$justFellResource'
    operator: '=='
    right: '1'
  fallenTimerRunningCondition:
    left: '$fallenTimerResource'
    operator: '>'
    right: '0'
  fallenTimerDoneCondition:
    left: '$fallenTimerResource'
    operator: '=='
    right: '0'
  inTownCondition:
    left: '$zoneResource'
    operator: '=='
    right: '1'

Parameter:
  healthParameter:
    key: resource
    value: healthResource
  isPlayerParameter:
    key: resource
    value: isPlayerResource
  zoneParameter:
    key: resource
    value: zoneResource
  justFellParameter:
    key: resource
    value: justFellResource
  fallenTimerParameter:
    key: resource
    value: fallenTimerResource
  deathPenaltyValueParameter:
    key: value
    value: '10'
  fullHealthValueParameter:
    key: value
    value: '999999'
  oneValueParameter:
    key: value
    value: '1'
  zeroValueParameter:
    key: value
    value: '0'
  oneExpressionParameter:
    key: expression
    value: '1'
  fallenFormParameter:
    key: actor
    value: fallenHeroActor
  heroFormParameter:
    key: actor
    value: heroActor
  townMapParameter:
    key: map
    value: townMap
  graveyardParameter:
    key: location
    value: graveyardVector
  fallenMessageParameter:
    key: message
    value: 'You have fallen.'
  privateAudienceParameter:
    key: audience
    value: private

Vector:
  graveyardVector:
    x: 200
    y: 8

Action:
  # 1. Fall (the hero's health Trigger).
  heroDeathAction:
    if: deadCondition
    do: set_resource_self
    parameters: [isPlayerParameter, zeroValueParameter]
    then: heroDeathTimerAction
  heroDeathTimerAction:
    do: set_resource_self
    parameters: [fallenTimerParameter, deathPenaltyValueParameter]
    then: heroDeathFlagAction
  heroDeathFlagAction:
    do: set_resource_self
    parameters: [justFellParameter, oneValueParameter]
    then: heroDeathSwapAction
  heroDeathSwapAction:
    do: change_actor_self
    parameters: [fallenFormParameter]
  # 2. Countdown (the fallen actor's Timer).
  fallenTickAction:
    if: justFellCondition
    do: set_resource_self
    parameters: [justFellParameter, zeroValueParameter]
    then: fallenAnnounceAction
    else: fallenCountAction
  fallenAnnounceAction:
    do: echo_self
    parameters: [fallenMessageParameter, privateAudienceParameter]
    then: fallenCountAction
  fallenCountAction:
    if: fallenTimerRunningCondition
    do: minus_resource_self
    parameters: [fallenTimerParameter, oneExpressionParameter]
    then: fallenCountDoneAction
    else: riseAction
  fallenCountDoneAction:
    if: fallenTimerDoneCondition
    do: set_resource_self
    parameters: [healthParameter, fullHealthValueParameter]
    then: toGraveyardAction
  toGraveyardAction:
    if: inTownCondition
    do: teleport_self_to_vector
    parameters: [graveyardParameter]
    else: toGraveyardMapAction
  toGraveyardMapAction:
    do: set_resource_self
    parameters: [zoneParameter, oneValueParameter]
    then: toGraveyardTravelAction
  toGraveyardTravelAction:
    do: change_map_self
    parameters: [townMapParameter, graveyardParameter]
  # 3. Rise (the next tick, at the graveyard).
  riseAction:
    do: set_resource_self
    parameters: [isPlayerParameter, oneValueParameter]
    then: riseSwapAction
  riseSwapAction:
    do: change_actor_self
    parameters: [heroFormParameter]

Trigger:
  heroDeathTrigger:
    resource: healthResource
    action: heroDeathAction

Timer:
  fallenTickTimer:
    interval: '1'
    total: '0'
    action: fallenTickAction

Group:
  playersGroup:
    name: Adventurers
    color: '#5DA9E9'
  fallenGroup:
    name: Fallen
    color: '#6E6A78'

# The fallen body: the hero's sprite sheet, with the death clip as idle, walk and run.
Animation:
  heroDeadAnimation:
    N: [0, 1, 2]
    NE: [0, 1, 2]
    E: [0, 1, 2]
    SE: [0, 1, 2]
    S: [0, 1, 2]
    SW: [0, 1, 2]
    W: [0, 1, 2]
    NW: [0, 1, 2]
    loop: false

AnimationSet:
  fallenHeroAnimationSet:
    animations: [heroDeadAnimation]
    idle: heroDeadAnimation
    walk: heroDeadAnimation
    run: heroDeadAnimation

Sprite:
  fallenHeroSprite:
    animation_set: fallenHeroAnimationSet
    texture: assets/actor.png
    size: actorSizeVector
    margin: actorMarginInset

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    group: playersGroup
    triggers: [heroDeathTrigger]
    private: [healthResource]
  fallenHeroActor:
    name: Hero (fallen)
    sprite: fallenHeroSprite
    group: fallenGroup
    speed: 0.0
    targetable: false
    timers: [fallenTickTimer]
    private: [fallenTimerResource, healthResource]
```

`deathPenaltyValueParameter` is the penalty in seconds. The example has one graveyard, on `townMap` (zone 1). With
a graveyard per map, test the zone and teleport to that map's graveyard. Every map gate must set `zoneResource`
(see [Travel between maps](/recipes/map-gates.html)).

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Animation`](/reference/entities.html#animation), [`AnimationSet`](/reference/entities.html#animationset), [`Condition`](/reference/entities.html#condition), [`Group`](/reference/entities.html#group), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Sprite`](/reference/entities.html#sprite), [`Timer`](/reference/entities.html#timer), [`Trigger`](/reference/entities.html#trigger), [`Vector`](/reference/entities.html#vector)

**Gotchas.**

- **Untag on death.** A monster keeps its target by name, and the fallen body has the same name. Setting
  `isPlayerResource` to `0` makes every "is my target a player?" check let go. Outside `playersGroup` and with
  `targetable: false`, the body is never picked again. Gate heals and potions on the same tag.
- **The fallen actor has no health Trigger.** A Trigger fires on the actor's first frame, and health is 0 then,
  so a copied death Trigger would fire again.
- **Floating text belongs to its actor.** That is why the message comes from the fallen body's first tick, not
  from the hero that is being swapped out.
- **Teleport first, swap a tick later.** For a player on another computer, the teleport travels as a message to
  that computer. Swapping in the same instant can bring the hero back where it fell.
- **`change_map_*` to the map you're already on does nothing**, and no condition can read the current map. That is
  why gates set `zoneResource`.
- `change_actor_*` takes speed, perception and salience from the new actor. Everything else (resources, position,
  name) carries over. A player who quits while fallen logs back in fallen, and the countdown carries on.
- Regen and other Timers run only on actors that list them, so the fallen body regenerates nothing.
- `loop: false` plays the fall once and holds the last frame.
