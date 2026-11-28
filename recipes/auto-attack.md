# Auto-attack: right-click an enemy, and fight back when hit

**Problem.** In a WoW-style game you right-click an enemy and your character keeps swinging at it until it dies,
you pick something else, or you walk away. And when something hits you, you want your character to fight back
without a click.

**Pattern.** The engine calls this **auto-interact**: it repeats one skill on a target. It is not only for
fighting (see the last section).
- **Right-click to attack.**
  - Give the player's actor an `auto_interact` skill, plus an `auto_interact_interval` (seconds between swings) and
    an `auto_interact_range` (how close it must be, in isometric pixels; 0 means bases touching).
  - Give each enemy an `on_interact` action that runs `start_auto_interact`.
  - Right-clicking the enemy, the Interact key or its plate icon then runs that action, with the player as caller
    and the enemy as target, instead of walking up to talk. The player walks into range and swings until the
    target changes, dies or leaves, or the player moves by hand.
  - A skill cast in between does not reset the swing clock.
- **Fight back when hit.**
  - Give the player's actor a `reaction`. It runs whenever another actor's action lands on the player, with the
    player as caller and the one who acted as target.
  - Its checks keep it to enemies, and stop it from switching what the player is already doing, pulling a healer
    off the ally they target, or turning a fleeing player around.
  - Its last step starts auto-interact with `chase: false`. The player swings back while the enemy is within
    reach and never walks after it.

{% raw %}
```yaml
Resource:
  healthResource:
    name: Health
    default: 50
    min: '0'
    max: '50'
  isEnemyResource:            # a tag: set it to 1 on enemies at spawn (see the spawn-hook recipe)
    default: 0
    min: '0'
    max: '1'
    reveal: 999999

Parameter:
  healthParameter:
    key: resource
    value: healthResource
  swingDamageParameter:
    key: expression
    value: '1d6+2'
  noChaseParameter:
    key: chase
    value: 'false'

Condition:
  hitByEnemyCondition:        # the actor who acted on me is an enemy
    left: '@isEnemyResource'
    operator: '=='
    right: '1'
  notBusyCondition:           # I'm not already auto-attacking something
    left: '$auto_interacting'
    operator: '=='
    right: '0'
  noTargetCondition:          # I haven't picked a target...
    left: '$has_target'
    operator: '=='
    right: '0'
  isMyTargetCondition:        # ...or the one who hit me IS my target
    left: '@is_target'
    operator: '=='
    right: '1'
  standingStillCondition:     # I'm not walking away
    left: '$moving'
    operator: '=='
    right: '0'

Action:
  swingAction:
    name: Attack
    time: 0.5
    do: minus_resource_target
    parameters: [healthParameter, swingDamageParameter]
  attackItAction:             # an enemy's on_interact: right-click to attack
    do: start_auto_interact
  fightBackAction:            # the player's reaction: four checks, then swing back without chasing
    if: hitByEnemyCondition
    then: fightBackIfFreeAction
  fightBackIfFreeAction:
    if: notBusyCondition
    then: fightBackIfTargetAction
  fightBackIfTargetAction:
    if: noTargetCondition
    then: fightBackIfStillAction
    else: fightBackIfMyTargetAction
  fightBackIfMyTargetAction:
    if: isMyTargetCondition
    then: fightBackIfStillAction
  fightBackIfStillAction:
    if: standingStillCondition
    do: start_auto_interact
    parameters: [noChaseParameter]

Skill:
  swingSkill:
    name: Attack
    description: 'A plain swing: {{range: 1d6+2}} damage.'
    start: swingAction

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    auto_interact: swingSkill
    auto_interact_interval: 2.0
    auto_interact_range: 0
    reaction: fightBackAction
  wolfActor:
    name: Grey Wolf
    sprite: wolfSprite
    on_interact: attackItAction
```
{% endraw %}

**Entities used.** [`Actor`](/reference/entities.html#actor), [`Skill`](/reference/entities.html#skill), [`Action`](/reference/entities.html#action), [`Condition`](/reference/entities.html#condition), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource)

**Actions used.** [`start_auto_interact`](/reference/actions.html#start_auto_interact), [`stop_auto_interact_self`](/reference/actions.html#stop_auto_interact_self), [`minus_resource_target`](/reference/actions.html#minus_resource_target)

**How a reaction is counted.**
- It runs once after each action chain in which another actor acted on this one: any `_target` action, or
  `transfer_resource`.
- An area effect counts for every actor it reaches.
- Moving toward someone, or spawning or teleporting next to them, doesn't count.
- An action whose `if` fails lands nothing.
- Nothing a reaction does sets off further reactions, so two actors can't bounce reactions back and forth.

**Gotchas.**

- `on_interact` replaces the walk-up dialog, so give it only to actors you fight. NPCs keep their `menu`.
- The swing is the skill's `start` action, run as a skill: its `if`, `time` and animation all apply. Put range or
  "is alive" checks in that action's `if` as usual.
- A reaction runs on *every* action that lands, heals and buffs from friends included. Always start it with an
  `if`, like `hitByEnemyCondition` above.
- Without `notBusyCondition`, a second enemy biting you would pull your swings off the one you chose.
- Without the target checks, a healer targeting an ally would switch to whatever bit them.
- Without `chase: false` (or `standingStillCondition`), every bite while you run away would turn you around.
- Right-clicking what a reaction started lets it chase again, as a normal right-click does.
- `stop_auto_interact_self` ends it from an action (for example when the player opens a vendor).
- A ranged class can set `auto_interact_range` to its bow's reach so it stops and fires from there.
- Players can turn reactions off for themselves: Options → **React automatically**. Mobs and NPCs always react.
- Mobs can have a `reaction` too, for example to turn on whoever hits them.

## Haste: swing faster or slower

`auto_interact_speed` is a percentage, read before each swing (`$` is the attacker, `@` its target). The wait
between swings is `auto_interact_interval × 100 / speed`, so `'125'` swings a quarter faster and `'50'` half as
fast. Point it at a stat to make haste and slows work:

{% raw %}
```yaml
Resource:
  hasteResource:              # buffs and gear add to it; 25 = a quarter faster
    name: Haste
    default: 0
    min: '0'
    max: '100'

Actor:
  quickHeroActor:
    name: Quick Hero
    sprite: heroSprite
    auto_interact: swingSkill
    auto_interact_interval: 2.0
    auto_interact_speed: '100+$hasteResource'
```
{% endraw %}

With 25 haste the 2.0 s swing comes every 1.6 s. The speed never drops below 10 (ten times slower), so a big
slow can't stop an attack outright.

## Not just for fighting

The same pattern can repeat any interaction. A farmer who keeps tending a crop until it's grown:

{% raw %}
```yaml
Resource:
  growthResource:
    name: Growth
    default: 0
    min: '0'
    max: '10'

Parameter:
  growthParameter:
    key: resource
    value: growthResource
  oneGrowthParameter:
    key: expression
    value: '1'

Condition:
  notYetGrownCondition:
    left: '@growthResource'
    operator: '<'
    right: '10'

Action:
  tendAction:
    name: Tend
    time: 1.0
    if: notYetGrownCondition
    do: plus_resource_target
    parameters: [growthParameter, oneGrowthParameter]
    else: doneTendingAction
  doneTendingAction:
    do: stop_auto_interact_self
  keepTendingAction:
    do: start_auto_interact

Skill:
  tendSkill:
    name: Tend
    start: tendAction

Actor:
  farmerActor:
    name: Farmer
    sprite: villagerSprite
    auto_interact: tendSkill
    auto_interact_interval: 3.0
    auto_interact_range: 0
  cropActor:
    name: Barley
    sprite: oakTreeSprite
    on_interact: keepTendingAction
```
{% endraw %}
