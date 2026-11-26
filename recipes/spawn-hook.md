# Set up each actor when it spawns

**Problem.** You want a wolf to start at level 3, or every hero to start with a torch, and `on_start` doesn't
seem to fire for them.

**Pattern.** `on_start` runs only for a player's own character, and only on that player's first login. For
everything else use a **spawn hook**: a `spawnedResource` that starts at `0`, and a [`Trigger`](/reference/entities.html#trigger) on it. A Trigger
fires once on the actor's first frame, so its guarded action runs, does the setup, and sets `spawnedResource` to
`1` so it never runs again.

```yaml
Resource:
  spawnedResource:
    name: Spawned
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  levelResource:
    name: Level
    default: 1
    min: '1'
    max: '20'

Condition:
  unspawnedCondition:
    left: '$spawnedResource'
    operator: '=='
    right: '0'

Parameter:
  spawnedParameter:
    key: resource
    value: spawnedResource
  levelParameter:
    key: resource
    value: levelResource
  oneValueParameter:
    key: value
    value: '1'
  wolfLevelValueParameter:
    key: value
    value: '3'

Action:
  wolfSpawnAction:
    if: unspawnedCondition
    do: set_resource_self
    parameters: [levelParameter, wolfLevelValueParameter]
    then: markSpawnedAction
  markSpawnedAction:
    do: set_resource_self
    parameters: [spawnedParameter, oneValueParameter]

Trigger:
  wolfSpawnTrigger:
    resource: spawnedResource
    action: wolfSpawnAction

Actor:
  wolfActor:
    name: Grey Wolf
    sprite: wolfSprite
    triggers: [wolfSpawnTrigger]
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Trigger`](/reference/entities.html#trigger)

**Gotchas.**

- A Trigger fires on the first frame and then whenever its resource changes. Always guard its action with `if`.
- Several changes in one frame fire the Trigger once.
- A player's resources are saved, so `spawnedResource` stays `1` and the hook runs once per character, not once
  per login.
- `set_resource_*` takes a plain number. To set a computed value, set `0` and then `plus_resource_*` the
  expression.
- Give each kind of actor its own spawn action. Chain to a shared one at the end for the common steps.
