# Monsters that come back

**Problem.** A camp of wolves should fill up again some time after the players clear it, but not while a player
is standing right there.

**Pattern.** Set `respawn` on each `Deployment`: seconds after its actor leaves the world for good (it dies and
despawns) before a fresh one spawns at the same spot. The timer waits while a player is near the spot. A monster
that wanders can be walked home with `move_home_self`, for example from a Strategy behaviour when the built-in
`distance_to_home` measure grows too large.

{% raw %}
```yaml
Vector:
  wolfDenVector:
    x: 320
    y: 40

Deployment:
  denWolfDeployment:
    actor: wolfActor
    location: wolfDenVector
    respawn: 120

Condition:
  nearHomeCondition:
    left: '$distance_to_home'
    operator: '<'
    right: '200'

Action:
  goHomeAction:
    do: move_home_self

Behavior:
  stayNearHomeBehavior:
    action: goHomeAction
    goals: [nearHomeCondition]

Strategy:
  wolfStrategy:
    behaviors: [stayNearHomeBehavior]

Actor:
  wolfActor:
    name: Grey Wolf
    sprite: wolfSprite
    strategy: wolfStrategy

Map:
  townMap:
    name: Town
    tilemap: groundTilemap
    spawn: townSpawnVector
    deployments: [denWolfDeployment]
```
{% endraw %}

**Entities used.** [`Deployment`](/reference/entities.html#deployment), [`Strategy`](/reference/entities.html#strategy), [`Behavior`](/reference/entities.html#behavior), [`Condition`](/reference/entities.html#condition), [`Action`](/reference/entities.html#action), [`Vector`](/reference/entities.html#vector)

**Gotchas.**

- `respawn: 0` (the default) means never: deployments spawn once, when the world is built.
- A change of form (`change_actor_*`) is not "leaving for good", so it doesn't start the timer.
- The respawned actor is fresh: every resource at its default, its spawn hook runs again
  ([Spawn hook](/recipes/spawn-hook.html)).
- Respawn timers live on the host; they don't survive a server restart (the world rebuilds every deployment).
