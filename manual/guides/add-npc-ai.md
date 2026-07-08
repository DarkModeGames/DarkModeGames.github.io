# Add NPC AI

This page shows you how to make a non-player actor act on its own: patrol a route, watch for intruders, or chase whoever comes near. You do this by giving an actor a [strategy](/manual/getting-started/glossary.html#terms), so it is worth understanding before you populate a map with anything other than the player.

## Prerequisites

You should already be comfortable with these pages:

- [Create an actor](/manual/guides/create-an-actor.html) — the actor that the AI will drive.
- [Define actions](/manual/guides/define-actions.html) — the actions a behavior runs.
- [Conditions and logic](/manual/guides/conditions-and-logic.html) — the conditions a behavior uses as goals.
- [Build a map](/manual/guides/build-a-map.html) — the map and tracks the AI moves across.

## How NPC AI is organized

Isometry drives AI with two entity types that work together.

A [Behavior](/manual/getting-started/glossary.html#terms) is a single rule: a set of *goals* and one *action*. The goals are [conditions](/manual/getting-started/glossary.html#terms) that describe when the rule applies. The action is what the actor does when the rule is active and all its goals are true.

A [Strategy](/manual/getting-started/glossary.html#terms) is an ordered list of behaviors. It is the AI controller you attach to an actor. The engine reads the behaviors as a priority list, top to bottom, and the first behavior whose goals are met runs its action.

You wire AI onto an actor by setting that actor's `strategy` field to a Strategy key. An actor without a `strategy` field does nothing on its own. This page builds up three AI patterns: patrol, follow, and guard.

> Note: `strategy`, `behaviors`, `goals`, and `action` are all optional fields. A strategy with no behaviors, or a behavior with no goals, is allowed — a behavior with empty goals always counts as met.

## When behaviors are evaluated

While the game is running on the host or server, the engine keeps checking an actor's strategy. AI never runs on a plain client. The engine resolves the actor's current target, walks the strategy's behaviors in order, and runs the action of the first behavior whose goals are all true.

Goals are combined with logical AND: every condition in a behavior's `goals` must be true for that behavior to match. Because behaviors are ordered, put the most specific or highest-priority behavior first.

## Conditions for AI: targets and distance

AI behaviors read built-in *measure references* that describe the actor's view of the world. Two of them drive every pattern here:

- `@has_target` is `1` when the actor currently has a target, `0` otherwise.
- `@distance_to_target` is how far the target is, in pixels.

A target is another actor the AI has detected. Detection is sized by the actor's `perception` and `salience` fields (see [Create an actor](/manual/guides/create-an-actor.html)); a patroller might use `perception` of `20`, a guard `15`. These are actor fields, not strategy fields — they decide *what* the AI can see, while the strategy decides *what the actor does about it*.

The example defines two goal conditions. The first is true when a target exists; the second is true when the target is more than one pixel away.

```yaml
Condition:
  hasTarget:
    left: '@has_target'
    operator: '='
    right: '1'
  isNearTarget:
    left: '@distance_to_target'
    operator: '>'
    right: '1'
```

`hasTarget` gates behaviors that should run once an actor is spotted. `isNearTarget` gates pursuit, which should stop when the target is reached. See [Conditions and logic](/manual/guides/conditions-and-logic.html) for the full operator list.

## Build a patrol

A patrol walks a fixed route until something interrupts it. The behavior's goal is `hasTarget`, and its action follows a track.

The patrol action uses the `use_track` action function and lists one parameter per waypoint. Each parameter's key is an integer index (`0`, `1`, `2`, …) and its value is a [Vector](/reference/entities.html#vector) key; `use_track` visits the points in ascending index order (gaps are allowed). Each parameter is a flat [KeyRef](/manual/getting-started/glossary.html#terms) to a top-level `Parameter` block, never an inline-nested object.

```yaml
Action:
  patrolAction:
    name: Patrol
    do: use_track
    then: scanAction
    parameters:
    - patrolPoint0
    - patrolPoint1
Parameter:
  patrolPoint0:
    key: '0'
    value: patrolPointA
  patrolPoint1:
    key: '1'
    value: patrolPointB
```

The `then` field chains `scanAction` after the move, so the actor looks for a target as it walks. See [Define actions](/manual/guides/define-actions.html) for how `then`, `do`, and `parameters` fit together.

Wrap that action in a behavior, then in a strategy:

```yaml
Behavior:
  patrolBehavior:
    goals:
    - hasTarget
    action: patrolAction
Strategy:
  patrolStrategy:
    behaviors:
    - patrolBehavior
```

Finally, attach the strategy to an actor through its `strategy` field. The merchant patrols:

```yaml
Actor:
  merchantActor:
    strategy: patrolStrategy
```

## Build a follow

A follow chases a target until it catches up. Its goal is `isNearTarget`, and its action moves toward the target with the `move_to_target` action function.

```yaml
Action:
  pursueAction:
    name: Pursue
    do: move_to_target
Behavior:
  followBehavior:
    goals:
    - isNearTarget
    action: pursueAction
Strategy:
  followStrategy:
    behaviors:
    - followBehavior
```

Because the goal is "target is more than one pixel away," the actor pursues until it closes the gap, then the goal stops being met and the action stops. The patroller actor uses `followStrategy`.

## Build a guard

A guard combines the two patterns into one ordered strategy. It first tries to scan for intruders, then falls back to following one it has found. Order matters: the engine runs the first behavior whose goals are met, so `guardBehavior` comes before `followBehavior`.

The scan action targets the nearest actor with the `target_nearest` action function:

```yaml
Action:
  scanAction:
    name: Scan
    do: target_nearest
```

The guard behavior runs that scan once a target exists, and the strategy lists it ahead of the reused follow behavior:

```yaml
Behavior:
  guardBehavior:
    goals:
    - hasTarget
    action: scanAction
Strategy:
  guardStrategy:
    behaviors:
    - guardBehavior
    - followBehavior
```

The guard actor sets `"strategy": "guardStrategy"`. Notice that `followBehavior` is shared between `followStrategy` and `guardStrategy` — behaviors are entities, so you reference one from many strategies rather than duplicating it.

> Tip: To build new AI, start from these three strategies and swap the actions and goal conditions for your own.

## Verify it loads

A strategy only takes effect after the campaign validates and loads. Boot your campaign on a host to see the actors move, since AI runs on host or server only:

```bash
./isometry --campaign=mygame --network=host --port=5000 \
  --username=p --secret=p --log-level=INFO
```

A clean boot logs `Campaign validation passed` and creates the actors. If a strategy references a missing behavior, condition, or action key, validation fails and names the broken reference.

## See also

- [Create an actor](/manual/guides/create-an-actor.html)
- [Define actions](/manual/guides/define-actions.html)
- [Conditions and logic](/manual/guides/conditions-and-logic.html)
- [Build a map](/manual/guides/build-a-map.html)
- [Triggers and timers](/manual/guides/triggers-and-timers.html)
- [Menus, dialogs, and waypoints](/manual/guides/menus-dialogs-waypoints.html)
- [Entity reference](/reference/entities.html)
- [Action function reference](/reference/actions.html)
