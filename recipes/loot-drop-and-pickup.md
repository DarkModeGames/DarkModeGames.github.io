# Drop loot on death and let a player claim it

**Problem.** A defeated monster should leave a bag behind, and the first player to walk onto it gets the gold.

**Pattern.** A health [`Trigger`](/reference/entities.html#trigger) on the monster, guarded by "health is 0", spawns a bag actor at its feet and then
removes the monster. The bag's `on_touch` action runs when a player walks into it. The bag is the caller and the
player is the target, so the action pays the target and removes the bag.

```yaml
Resource:
  healthResource:
    name: Health
    default: 30
    min: '0'
    max: '30'
  goldResource:
    name: Gold
    default: 0
    min: '0'
    max: '99999'
  isPlayerResource:
    name: Is Player
    default: 0
    min: '0'
    max: '1'
    reveal: 999999

Condition:
  deadCondition:
    left: '$healthResource'
    operator: '<='
    right: '0'
  targetIsPlayerCondition:
    left: '@isPlayerResource'
    operator: '=='
    right: '1'

Parameter:
  goldParameter:
    key: resource
    value: goldResource
  bagActorParameter:
    key: actor
    value: lootBagActor
  bagGoldParameter:
    key: expression
    value: '2d6+3'

Action:
  wolfDeathAction:
    if: deadCondition
    do: spawn_actor_at_self
    parameters: [bagActorParameter]
    then: wolfRemoveAction
  wolfRemoveAction:
    do: despawn_self
  claimLootAction:
    if: targetIsPlayerCondition
    do: plus_resource_target
    parameters: [goldParameter, bagGoldParameter]
    then: bagRemoveAction
  bagRemoveAction:
    do: despawn_self

Trigger:
  wolfDeathTrigger:
    resource: healthResource
    action: wolfDeathAction

Actor:
  wolfActor:
    name: Grey Wolf
    sprite: wolfSprite
    triggers: [wolfDeathTrigger]
    public: [healthResource]
  lootBagActor:
    name: Loot Bag
    sprite: bagSprite
    base: 6
    salience: 240
    on_touch: claimLootAction
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Trigger`](/reference/entities.html#trigger)

**Gotchas.**

- `on_touch` fires only when the toucher is a player's own character, so monsters never pick up bags.
- The health Trigger fires on every hit. The `if` makes only the killing blow count.
- A despawned deployment never comes back until the host restarts. For respawning monsters, spawn them from
  something that stays (a camp actor with a [Timer](/reference/entities.html#timer)).
- To show a death animation first, swap the monster for a "corpse" actor with
  [`change_actor_self`](/reference/actions.html#change_actor_self) and despawn that after a Timer.
- Use `spawn_actor_at_self_radial` to scatter several drops around the body.
