# Let players pick their hero

**Problem.** You want each player to choose a character at the start instead of always
controlling the same one.

**Pattern.** Leave [`Main`](/reference/entities.html#main)'s `actor` unset. When a campaign names no starting actor, each
player begins as a [**null actor**](/manual/core-concepts/campaign-structure.html#the-main-entity) —
a perfect "character select" vantage point. Since the null actor can't walk up and touch
anything, give each hero a `menu` instead of an `on_touch` action: right-clicking a hero
opens its menu from any distance, and the menu's action turns the player into that hero
(`change_actor_self`).

```yaml
# No `actor` on Main -> players start as the null actor.
Main:
  main:
    map: townSquare

# Heroes standing in the world, each ready to be taken over.
Actor:
  knight:
    name: Knight
    sprite: knightSprite
    menu: knightMenu
  mage:
    name: Mage
    sprite: mageSprite
    menu: mageMenu

Menu:
  knightMenu:
    name: Actions
    actions:
    - becomeKnight
  mageMenu:
    name: Actions
    actions:
    - becomeMage

Action:
  becomeKnight:
    do: change_actor_self
    parameters: [asKnight]
  becomeMage:
    do: change_actor_self
    parameters: [asMage]

Parameter:
  asKnight:
    key: actor
    value: knight
  asMage:
    key: actor
    value: mage

# Stand the heroes in the square, near where players start.
Map:
  townSquare:
    name: Town Square
    tilemap: groundTilemap
    spawn: squareSpawn
    deployments: [knightStand, mageStand]

Vector:
  squareSpawn:
    x: 384
    y: 8
  knightSpot:
    x: 344
    y: 8
  mageSpot:
    x: 424
    y: 8

Deployment:
  knightStand:
    actor: knight
    location: knightSpot
  mageStand:
    actor: mage
    location: mageSpot
```

`groundTilemap` stands for your own [`TileMap`](/reference/entities.html#tilemap). Right-click a hero to open its menu and select the action. See the
[action reference](/reference/actions.html#change_actor_self) for `change_actor_self`, and
[Campaign structure](/manual/core-concepts/campaign-structure.html) for `Main`.

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Deployment`](/reference/entities.html#deployment), [`Main`](/reference/entities.html#main), [`Map`](/reference/entities.html#map), [`Menu`](/reference/entities.html#menu), [`Parameter`](/reference/entities.html#parameter), [`Vector`](/reference/entities.html#vector)

**Gotchas.**

- The null actor has unlimited reach: its right-click opens a hero's menu from anywhere. A real character would
  walk up and the menu opens once they are close.
- The null actor holds no resources. After the change, the new hero starts with every resource at its `default`;
  set anything else up with a [spawn hook](/recipes/spawn-hook.html) on the hero.
- The deployed heroes stay where they stand. A player who picks the knight becomes a second knight, and the
  standing knight remains for the next player.
- The choice is saved with the player, so they come back as the hero they picked. To stop a player choosing
  twice, check a resource in the menu action's `if`.
