# Add NPC AI

This page shows you how to make a non-player actor act on its own: patrol a route, spot a player, chase them, and attack. You do this by giving an actor a [strategy](/manual/getting-started/glossary.html#terms).

## Prerequisites

You should already be comfortable with these pages:

- [Create an actor](/manual/guides/create-an-actor.html) — the actor that the AI will drive.
- [Define actions](/manual/guides/define-actions.html) — the actions a behavior runs.
- [Conditions and logic](/manual/guides/conditions-and-logic.html) — the conditions a behavior uses as goals.
- [Build a map](/manual/guides/build-a-map.html) — the map the AI moves across.

## How NPC AI is organized

AI is built from two entity types.

A [Behavior](/manual/getting-started/glossary.html#terms) is one step of a plan: a list of **goals** (Conditions) and one **action**. Think of the goals as "what this step is trying to achieve". The action runs, over and over, **until every goal is true**.

A [Strategy](/manual/getting-started/glossary.html#terms) is an ordered list of behaviors: the whole plan, top to bottom. You attach it to an actor with the actor's `strategy` field.

## How a strategy runs

The engine checks each AI actor's strategy several times a second, on the host or server only (never on a plain client). Each check:

1. **Fall back if something broke.** If a goal of an *earlier* behavior has stopped being true (for example, the actor lost its target), that earlier behavior becomes the current one again.
2. **Skip steps that are done.** Starting at the current behavior, any behavior whose goals are all true is passed over.
3. **Run one action.** The first behavior with an unmet goal runs its action.
4. If every behavior's goals are true, nothing runs: the plan is complete.

So you write a strategy as a checklist: "have a target", then "be close to it", then "it is down". Each behavior does the work that makes its own goal come true.

> Note: A behavior with no goals is never "done", so its action runs on every check. Put one at the end of a list for an action that should repeat forever.

## The goal conditions

In a behavior, the **caller is the AI actor itself** and the target is its current target. So `$` reads the AI, and `@` reads whoever it is targeting.

```yaml
Condition:
  hasTarget:
    left: $has_target
    operator: '=='
    right: '1'
  closeToTarget:
    left: $distance_to_target
    operator: <=
    right: '40'
  targetIsDown:
    left: '@health'
    operator: <=
    right: '0'
```

- `has_target` and `distance_to_target` are built-in values every actor has.
- `distance_to_target` is in pixels on the isometric ground, measured centre to centre. Vertical distance counts double. See [Distances and sizes](/reference/units.html).

## A monster that hunts players

This monster waits until a player comes into view, walks up to them, and bites until they fall.

```yaml
Group:
  playersGroup:
    name: Players
    color: '#4080ff'
  monstersGroup:
    name: Monsters
    color: '#ff4040'

Action:
  findPlayerAction:
    do: target_nearest_in_group
    parameters: [findPlayersParam]
  chaseAction:
    do: move_to_target
  biteAction:
    name: Bite
    do: minus_resource_target
    time: 1.0
    parameters: [biteResourceParam, biteDamageParam]

Parameter:
  findPlayersParam: {key: group, value: playersGroup}
  biteResourceParam: {key: resource, value: health}
  biteDamageParam: {key: expression, value: 1d6}

Behavior:
  findBehavior:
    goals: [hasTarget]
    action: findPlayerAction
  chaseBehavior:
    goals: [closeToTarget]
    action: chaseAction
  attackBehavior:
    goals: [targetIsDown]
    action: biteAction

Strategy:
  hunterStrategy:
    behaviors: [findBehavior, chaseBehavior, attackBehavior]

Actor:
  wolfActor:
    name: Wolf
    sprite: wolfSprite
    group: monstersGroup
    perception: 200
    strategy: hunterStrategy
```

Read it as a checklist:

1. **findBehavior.** Until I have a target, look for the nearest player I can see.
2. **chaseBehavior.** Until I'm within 40 px of my target, walk toward it.
3. **attackBehavior.** Until my target is down, bite it. `time: 1.0` makes each bite take a second.

If the player runs away, `closeToTarget` breaks and the wolf falls back to chasing. If it loses the player completely, `hasTarget` breaks and it goes back to looking. Players must be in `playersGroup` (set `group: playersGroup` on the player actor).

> Tip: `target_nearest_in_group` only finds actors the AI can see, so a bigger `perception` means the monster notices players from farther away.

## A guard that patrols

Put a patrol **first** in the list, with the goal "have a target". The guard walks its route until it spots someone; after that, the chase and attack steps take over. Chain a scan after the patrol so it looks around as it walks:

```yaml
Vector:
  gatePos: {x: 200, y: 40}
  wellPos: {x: 360, y: 120}

Action:
  patrolAction:
    do: use_track
    parameters: [patrolPoint0, patrolPoint1]
    then: findPlayerAction

Parameter:
  patrolPoint0: {key: '0', value: gatePos}
  patrolPoint1: {key: '1', value: wellPos}

Behavior:
  patrolBehavior:
    goals: [hasTarget]
    action: patrolAction

Strategy:
  guardStrategy:
    behaviors: [patrolBehavior, chaseBehavior, attackBehavior]
```

`use_track` walks the Vectors in order of their index keys (`'0'`, `'1'`, …), and back to the first after the last. Each call sends the actor to its current point, or on to the next once it has stopped at it, so the patrol needs calling over and over: a behavior's action does that several times a second, and a Timer with an interval of about a second works too. Calling it again mid-walk carries on rather than starting over; giving the actor a different track starts that track at its first point. A point the actor can't quite reach (off the walkable ground) counts as reached where the actor stops. Behaviors are entities, so `chaseBehavior` and `attackBehavior` are shared with the hunter rather than copied.

## Common mistakes

- **Writing goals as "when to act".** A goal is the state the behavior is trying to *reach*. "Chase while far away" is written as the goal `closeToTarget`, not `farFromTarget`.
- **A goal the action can never achieve.** If nothing the action does can make the goal true, the actor repeats that action forever.
- **Using `@` where you mean "me".** `@has_target` asks whether the *target* has a target. Use `$has_target`.
- **AI that never starts.** Strategies only run on a host or server. Check that the actor has `strategy` and every behavior has an `action`.

For bigger patterns (threat, leashing, packs that call for help) see [Recipes](/recipes/).

## See also

- [Conditions and logic](/manual/guides/conditions-and-logic.html)
- [Define actions](/manual/guides/define-actions.html)
- [Triggers and timers](/manual/guides/triggers-and-timers.html)
- [Recipes](/recipes/)
- [Action function reference](/reference/actions.html)
- [Entity reference](/reference/entities.html)
