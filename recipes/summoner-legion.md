# A summoner with a following legion

**Problem.** A necromancer raises skeletons. They should march behind her in formation, fall on whatever she
attacks, come back into line when the fight is over, and never be confused with another necromancer's dead.

**Pattern.** Raising is a plain `spawn_actor_at_self_radial`: the caster becomes each skeleton's **master**
([Minions and formations](/manual/guides/minions-and-formations.html)). A cap of six is one condition on
`$minions`. Each skeleton runs a two-step [strategy](/recipes/mob-ai-strategy.html):

1. **Fight** while it has a target: close in and strike.
2. **Follow** otherwise: `follow_master` to its place in a column, then `target_master_target` to pick up the
   necromancer's target. The moment it has one, the fight step's goal breaks and the skeleton falls back to it.

Two spells use the master link from the necromancer's side: **Onward** (`each_minion`) hands her target to every
skeleton at once, and **Grave Mending** heals only her own skeletons in an area, filtered with `@mine`.

```yaml
Resource:
  healthResource:
    name: Health
    default: 30
    min: '0'
    max: '30'

Actor:
  necromancerActor:
    name: Necromancer
    speed: 1.0
  skeletonActor:
    name: Skeleton
    speed: 1.0
    strategy: skeletonStrategy

Condition:
  roomForMoreCondition:
    left: '$minions'
    operator: '<'
    right: '6'
  noTargetCondition:
    left: '$has_target'
    operator: '=='
    right: '0'
  inReachCondition:
    left: '$distance_to_target'
    operator: '<='
    right: '40'
  neverCondition:
    left: '0'
    operator: '=='
    right: '1'
  targetIsMineCondition:
    left: '@mine'
    operator: '=='
    right: '1'

Parameter:
  skeletonParameter:
    key: actor
    value: skeletonActor
  raiseRadialParameter:
    key: radial
    value: '180'
  raiseDistanceParameter:
    key: distance
    value: '40'
  columnParameter:
    key: formation
    value: column
  spacingParameter:
    key: spacing
    value: '40'
  healthParameter:
    key: resource
    value: healthResource
  strikeParameter:
    key: expression
    value: '1d6'
  mendParameter:
    key: expression
    value: '8'
  onwardEachParameter:
    key: action
    value: onwardEachAction
  mendRadiusParameter:
    key: radius
    value: '200'
  mendEachParameter:
    key: action
    value: mendEachAction

Action:
  # The necromancer's spells (put them on her skills).
  raiseAction:
    if: roomForMoreCondition
    do: spawn_actor_at_self_radial
    parameters: [skeletonParameter, raiseRadialParameter, raiseDistanceParameter]
  onwardAction:
    do: each_minion
    parameters: [onwardEachParameter]
  onwardEachAction:
    do: set_target_target_to_self_target
  mendAction:
    do: area_of_effect_at_self
    parameters: [mendRadiusParameter, mendEachParameter]
  mendEachAction:
    if: targetIsMineCondition
    do: plus_resource_target
    parameters: [healthParameter, mendParameter]

  # The skeleton's steps.
  fightAction:
    if: inReachCondition
    do: minus_resource_target
    parameters: [healthParameter, strikeParameter]
    time: 1.0
    else: closeInAction
  closeInAction:
    do: move_to_target
  followAction:
    do: follow_master
    parameters: [columnParameter, spacingParameter]
    then: assistAction
  assistAction:
    do: target_master_target

Behavior:
  fightBehavior:
    goals: [noTargetCondition]
    action: fightAction
  followBehavior:
    goals: [neverCondition]
    action: followAction

Strategy:
  skeletonStrategy:
    behaviors: [fightBehavior, followBehavior]
```

Give the skeleton a sprite with `walk` and `run` clips, and put `raiseAction`, `onwardAction` and `mendAction` on
the necromancer's [skills](/manual/guides/add-skills.html).

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Behavior`](/reference/entities.html#behavior), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Strategy`](/reference/entities.html#strategy)

**Gotchas.**

- The skeleton is raised 40 px behind the necromancer (`radial: 180`), close to its place in the column, so it
  falls in without a long walk.
- `target_master_target` never hands a skeleton itself as a target, so a necromancer who clicks on her own
  skeleton doesn't set it fighting itself. Add an `if` on `@mine == 0` to the fight step as well if her spells
  can target her own dead.
- Grave Mending's area includes the necromancer and everyone nearby: `@mine` is what keeps it to her skeletons.
  For a spell that should spare them instead, test `@mine == 0`.
- The skeletons leave with her. When she changes map or logs out, they despawn; raise them again on arrival.
- `follow_master` sets each skeleton's speed: its master's, or faster while it catches up. It keeps the speed it
  last had while it fights.
- To change the shape, give the skeletons a second strategy with another `formation` and swap with
  `change_strategy` from `each_minion`.
