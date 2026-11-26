# Minions and formations

This page shows you how to build a summoner: an actor that raises other actors, knows which ones are its own, and leads them about in formation.

## Prerequisites

- [Define actions](/manual/guides/define-actions.html) — spawning is an action.
- [Conditions and logic](/manual/guides/conditions-and-logic.html) — the built-in values below are read in conditions.
- [Add NPC AI](/manual/guides/add-npc-ai.html) — a minion follows and fights through its strategy.

## Masters and minions

Every `spawn_actor_*` action (`spawn_actor_at_self`, `spawn_actor_at_target`, and their `_radial` forms) records its **caller** as the new actor's **master**. The new actor is the caller's **minion**. Nothing else is needed: raise a skeleton with `spawn_actor_at_self` and it belongs to whoever ran the action.

- The link holds on every player's screen, including players who join later.
- `change_actor` keeps it, on both sides: a minion that changes form is still a minion, and a master that changes form keeps its minions.
- Minions are never saved. When a master leaves the world — it despawns, changes map, or its player disconnects — all its minions despawn with it.
- Actors placed by a map's `deployments` have no master.

## Values a minion and its master can read

Three [built-in values](/manual/guides/conditions-and-logic.html) work like resources in any dice expression:

| Value | Reads |
|---|---|
| `$minions` | How many minions the actor has in the world. A minion that despawns stops counting at once. |
| `$has_master` | `1` if the actor's master is still in the world, otherwise `0`. |
| `@mine` | `1` when the target is the caller's own minion, otherwise `0`. `$mine` is always `0`. |

`@mine` is what keeps two summoners apart. A heal that should only touch your own skeletons tests `@mine == 1` in its `if`; an area spell that should spare them tests `@mine == 0` in its sub-action. Another summoner's skeletons are never yours.

A cap on raised minions is one condition on the raise action:

```yaml
Condition:
  roomForMoreCondition:
    left: '$minions'
    operator: '<'
    right: '6'
```

## Actions for minions

| Action | Caller | What it does |
|---|---|---|
| `target_master` | the minion | Targets its master. Does nothing without one. |
| `target_master_target` | the minion | Targets whatever its master is targeting. Does nothing when the master has no target, or is targeting this minion. |
| `each_minion` | the master | Runs its `action` parameter once per minion, with the master as caller and the minion as target. |
| `follow_master` | the minion | Walks to its place in the master's formation. |

`each_minion` is the master's way of giving an order: "all of you, attack my target" is `each_minion` with a sub-action that runs `set_target_target_to_self_target`.

## Formations

`follow_master` takes two parameters:

- `formation` — the shape:
  - `column`: rows of two behind the master;
  - `wedge`: a V opening out behind the master;
  - `ring`: evenly spaced all round the master;
  - `line`: one rank abreast, behind the master.
- `spacing` — the gap between neighbouring minions, and between the master and its first rank, in isometric pixels (a number or dice). Leave room for the minions' sprites: a 64 px sprite looks crowded below about 40.

**Each minion's place** is its place in the order the master raised them. When one is lost, the ones raised after it move up and the gap closes.

**The formation faces the way the master faces.** While the master walks, that is the way it is walking. While it stands, it is the last way it walked, or a direction it turns to face. When the master turns, the formation wheels round to follow it over a moment rather than jumping.

**Pace.** A minion walks at its master's speed while it is near its place, and runs when it falls behind, faster the further behind it is. `follow_master` sets the minion's speed to do this.

Run `follow_master` often, as the action of a behavior whose goal is never met, so the minion keeps its place:

```yaml
Condition:
  neverCondition:
    left: '0'
    operator: '=='
    right: '1'

Parameter:
  columnParameter:
    key: formation
    value: column
  spacingParameter:
    key: spacing
    value: '40'

Action:
  followAction:
    do: follow_master
    parameters: [columnParameter, spacingParameter]

Behavior:
  followBehavior:
    goals: [neverCondition]
    action: followAction
```

Running it every tick is cheap. A minion is only given a new path when its place has moved a little way, or when it stands still away from its place, so a large legion doesn't stutter or flood the pathfinder.

## Fighting for the master

Put a fight behavior **before** the follow behavior in the minion's strategy. While it has something to fight, it fights; when the fight ends, its goal is met again and the minion falls back into formation. See [A summoner with a following legion](/recipes/summoner-legion.html) for a complete strategy.

## Common mistakes

- **Spawning from the minion.** Whoever runs `spawn_actor_*` becomes the master. A Timer on a minion that spawns more actors makes the minion their master, not the summoner.
- **Reading `$mine`.** It is always `0`. The question "is my target mine?" is `@mine`.
- **Two follow behaviors.** A minion with two `follow_master` behaviors switches between two shapes every tick. Give each minion one, and change it with `change_strategy`.
- **Expecting minions to cross maps.** They stay behind and despawn when the master changes map. Raise them again on arrival.
