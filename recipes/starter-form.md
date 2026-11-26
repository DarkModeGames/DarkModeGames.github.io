# A starter body that picks a class

**Problem.** New players should walk around and look at the classes before choosing, not stand as a bodiless
camera. Once chosen, the class should stick.

**Pattern.** Make `Main.actor` a plain **starter** actor: it can walk and talk, belongs to its own
[`Group`](/reference/entities.html#group) (not the players' group, so monsters ignore it), and its hotbar holds one
skill per class. Each skill runs `change_actor_self` into that class. The class actor's own
[spawn hook](/recipes/spawn-hook.html) records the choice, so offering the same skills on the class actors (or
at a trainer NPC) can refuse anyone already sworn.

```yaml
Main:
  main:
    map: townMap
    actor: starterActor

Group:
  startersGroup:
    name: Newcomers
    color: '#9A9A9A'
  playersGroup:
    name: Adventurers
    color: '#5DA9E9'

Resource:
  classResource:
    name: Class
    description: 0 none, 1 Fighter, 2 Mystic.
    default: 0
    min: '0'
    max: '2'
    reveal: 999999

Condition:
  noClassCondition:
    left: '$classResource'
    operator: '=='
    right: '0'

Parameter:
  classParameter:
    key: resource
    value: classResource
  fighterValueParameter:
    key: value
    value: '1'
  mysticValueParameter:
    key: value
    value: '2'
  asFighterParameter:
    key: actor
    value: fighterActor
  asMysticParameter:
    key: actor
    value: mysticActor
  welcomeMessageParameter:
    key: message
    value: 'Press 1 or 2 to choose a class.'
  welcomeDurationParameter:
    key: duration
    value: '6'
  privateAudienceParameter:
    key: audience
    value: private

Action:
  welcomeAction:
    do: echo_self
    parameters: [welcomeMessageParameter, privateAudienceParameter, welcomeDurationParameter]
  becomeFighterAction:
    name: Become a Fighter
    time: 0.5
    if: noClassCondition
    do: change_actor_self
    parameters: [asFighterParameter]
  becomeMysticAction:
    name: Become a Mystic
    time: 0.5
    if: noClassCondition
    do: change_actor_self
    parameters: [asMysticParameter]
  fighterSpawnAction:
    if: noClassCondition
    do: set_resource_self
    parameters: [classParameter, fighterValueParameter]
  mysticSpawnAction:
    if: noClassCondition
    do: set_resource_self
    parameters: [classParameter, mysticValueParameter]

Trigger:
  fighterSpawnTrigger:
    resource: classResource
    action: fighterSpawnAction
  mysticSpawnTrigger:
    resource: classResource
    action: mysticSpawnAction

Skill:
  becomeFighterSkill:
    name: Fighter
    description: Steel and stubbornness.
    start: becomeFighterAction
  becomeMysticSkill:
    name: Mystic
    description: Old words and cold fire.
    start: becomeMysticAction

Actor:
  starterActor:
    name: Newcomer
    sprite: heroSprite
    group: startersGroup
    perception: 240
    salience: 240
    speed: 1.0
    on_start: welcomeAction
    skills: [becomeFighterSkill, becomeMysticSkill]
  fighterActor:
    name: Fighter
    sprite: heroSprite
    group: playersGroup
    perception: 240
    salience: 240
    speed: 1.0
    triggers: [fighterSpawnTrigger]
  mysticActor:
    name: Mystic
    sprite: heroSprite
    group: playersGroup
    perception: 240
    salience: 240
    speed: 1.0
    triggers: [mysticSpawnTrigger]
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Group`](/reference/entities.html#group), [`Main`](/reference/entities.html#main), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Skill`](/reference/entities.html#skill), [`Trigger`](/reference/entities.html#trigger)

**Gotchas.**

- `change_actor_self` keeps every resource and the position, and takes speed, perception and salience from the new
  actor. Give the starter the same sight as the classes, or the view jumps on the change.
- The class [Trigger](/reference/entities.html#trigger) fires on the new body's first frame, so `classResource` is set the moment the class appears.
- The player's actor key is saved, so a returning player logs in as their class, not the starter.
- `on_start` runs on the player's first login only. It's the right place for a one-time welcome line.
- Keep the starter out of the group monsters hunt, and don't tag it as a player, so gates and monsters treat it as
  nobody.
- Compare [Let players pick their hero](/recipes/pick-your-hero.html), which starts players as a bodiless camera
  instead.
