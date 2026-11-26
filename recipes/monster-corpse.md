# Leave a corpse for a few seconds

**Problem.** A monster that dies should fall over and lie there for a moment, not vanish on the killing blow.

**Pattern.** On death, swap the monster for a **corpse** actor with `change_actor_self`. The corpse uses the
monster's sprite with the death clip as its idle (`loop: false` holds the last frame), has speed `0`, no AI, and
`targetable: false`. A [Timer](/reference/entities.html#timer) on the corpse counts `corpseResource` down and despawns it at 0. The swap keeps the
position and every resource, so loot and credit can still read them.

```yaml
Resource:
  healthResource:
    name: Health
    default: 30
    min: '0'
    max: '30'
  corpseResource:
    name: Corpse Seconds
    default: 6
    min: '0'
    max: '60'
    reveal: 999999

Condition:
  deadCondition:
    left: '$healthResource'
    operator: '<='
    right: '0'
  corpseGoneCondition:
    left: '$corpseResource'
    operator: '<='
    right: '0'

Parameter:
  corpseParameter:
    key: resource
    value: corpseResource
  oneExpressionParameter:
    key: expression
    value: '1'
  asCorpseParameter:
    key: actor
    value: wolfCorpseActor

Action:
  wolfDeathAction:
    if: deadCondition
    do: change_actor_self
    parameters: [asCorpseParameter]
  corpseTickAction:
    do: minus_resource_self
    parameters: [corpseParameter, oneExpressionParameter]
    then: corpseGoneAction
  corpseGoneAction:
    if: corpseGoneCondition
    do: despawn_self

Trigger:
  wolfDeathTrigger:
    resource: healthResource
    action: wolfDeathAction

Timer:
  corpseTimer:
    interval: '1'
    total: '0'
    action: corpseTickAction

Animation:
  wolfDeadAnimation:
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
  wolfCorpseAnimationSet:
    animations: [wolfDeadAnimation]
    idle: wolfDeadAnimation

Sprite:
  wolfCorpseSprite:
    animation_set: wolfCorpseAnimationSet
    texture: assets/actor.png
    size: actorSizeVector
    margin: actorMarginInset

Actor:
  wolfActor:
    name: Grey Wolf
    sprite: wolfSprite
    triggers: [wolfDeathTrigger]
    public: [healthResource]
  wolfCorpseActor:
    name: Grey Wolf (dead)
    sprite: wolfCorpseSprite
    speed: 0.0
    targetable: false
    timers: [corpseTimer]
```

`actorSizeVector` and `actorMarginInset` stand for the wolf sheet's own frame size and margin.

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Animation`](/reference/entities.html#animation), [`AnimationSet`](/reference/entities.html#animationset), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Sprite`](/reference/entities.html#sprite), [`Timer`](/reference/entities.html#timer), [`Trigger`](/reference/entities.html#trigger)

**Gotchas.**

- The corpse needs no health [Trigger](/reference/entities.html#trigger). Its health is 0, and a Trigger fires on the first frame, so a copied death
  Trigger would fire again.
- Drop loot and grant credit in the death action **before** the swap (`spawn_actor_at_self`, area effects), or from
  the corpse's first Timer tick.
- Monsters that targeted something by name keep that name; untargetable corpses aren't picked again, but check
  "is my target an enemy?" in your player skills too.
- A despawned deployment never returns until the host restarts. Respawning camps need a spawner actor.
