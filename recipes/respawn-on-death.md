# Respawn an actor on death

**Problem.** You want a defeated actor to come back after a short delay instead of being
gone for good.

**Pattern.** Watch the actor's `health` resource with a condition, and when it hits zero run
an action chain that restores the resource and moves the actor back to a spawn point. A
`Timer` entity introduces the delay before the restore runs.

The building blocks are all data entities:

- a `Resource` (`health`) with a `min` of `0`,
- an `Action` that sets `health` back to full and teleports the actor to a `Vector` spawn point,
- a `Trigger` (or a `Strategy` condition) that fires the action when `health` reaches `0`,
- a `Timer` to delay the respawn.

```yaml
Action:
  respawnHero:
    do: set_resource_self
    parameters: [respawnResource, respawnValue]
    then: respawnMove
  respawnMove:
    do: set_destination_self
    parameters: [respawnPoint]

Parameter:
  respawnResource: {key: resource, value: health}
  respawnValue: {key: value, value: "100"}
  respawnPoint: {key: destination, value: heroSpawn}

Vector:
  heroSpawn: {x: 240, y: 160}
```

See the [action reference](/reference/actions.html) for the exact parameters of
`set_resource_self` and `set_destination_self`, and the
[entity reference](/reference/entities.html) for `Trigger`, `Timer`, and `Resource` fields.
