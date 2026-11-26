# Build a Map

This page shows you how to assemble a playable level out of terrain entities: a Map that ties together a tile grid, a spawn point, and the actors placed on it when the level loads. By the end you can author a map and understand why it looks empty until an actor walks across it.

## Prerequisites

You should be comfortable with the campaign data model before you build a map:

- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html) explains how entities reference each other by key.
- [Campaign structure](/manual/core-concepts/campaign-structure.html) explains where YAML files and assets live inside a campaign.
- [Create an actor](/manual/guides/create-an-actor.html) covers the actor you spawn onto the map and its `perception` field.

## Fog of war: why a correct map looks blank

Before you build anything, you need to know that a freshly loaded map renders nothing. This is intended.

Every tile layer starts hidden. Each tile begins fully transparent over a dark, unexplored background. This is [fog of war](/manual/getting-started/glossary.html#terms): the unexplored darkness that lifts only as an actor explores.

A tile reveals when an actor's discovery area overlaps it. That area is sized from the actor's `perception` field (see [Create an actor](/manual/guides/create-an-actor.html)). A hero with a small `perception` sees only the patch of terrain immediately around the spawn point when the level loads. The rest stays dark until the player moves.

Each tile is always in one of three states:

- **Visible** — inside an actor's discovery area right now, drawn at full strength.
- **Discovered** — seen before but not currently in view, drawn dimmed.
- **Undiscovered** — never seen, hidden.

Discovery is tracked per actor and is saved with the game, so explored areas stay explored across sessions.

> Note: If your map appears completely black after loading, that is normal. Move the player actor and the terrain fades in around it. A blank screen is not a sign that your tiles are wrong.

## How a map fits together

A map is a small stack of entities, each with one job. The [Map](/reference/entities.html#map) entity is the top of the stack and points at the rest by [KeyRef](/manual/getting-started/glossary.html#terms):

- A **TileMap** holds the ordered terrain layers and the tile definitions they draw from.
- A **TileSet** names the texture atlas and lists the **Tile** definitions.
- Each **Tile** maps a single symbol character to one cell in that atlas, and declares whether the tile is walkable or solid.
- Each **Layer** points at a plain-text grid file whose characters are tile symbols.
- A **Vector** marks the spawn point where the player enters.
- **Deployment** entities place non-player actors on the map at load time.

> Note: None of the fields below are required by the validator. In practice you set the ones you need for the map to function, and the example values show a working baseline.

## Define the tiles

Start with the look of the terrain. A [TileSet](/reference/entities.html#tileset) names a texture image inside the campaign and lays it out as a grid. The `columns` field tells the engine how wide that grid is, so it can turn each tile's `index` into a position in the atlas.

The example tile set uses an 8-column atlas and defines three tiles:

```yaml
TileSet:
  demoTileSet:
    columns: 8
    texture: /assets/basic.png
    tiles:
    - grass
    - water
    - wall
Face:
  roof:
    northeast: true
    northwest: true
Tile:
  grass:
    symbol: G
    index: 0
    navigation: roof
  water:
    symbol: W
    index: 46
  wall:
    symbol: X
    index: 52
    obstacle: roof
```

Each [Tile](/reference/entities.html#tile) field controls one thing:

- `symbol` is the character matched against the layer grid. It is a string by schema, but the engine reads one character at a time, so use a single character such as `"G"`. Symbols are matched **across the whole campaign**, so two tiles in different tilesets must not share a symbol.
- `index` picks which cell in the grid to use, counting left to right then top to bottom, starting at `0`. With `columns` set to 8, grass `0` is the top-left cell, water `46` is column 6 row 5, and wall `52` is column 4 row 6.
- `navigation` points at a [Face](/reference/entities.html#face) entity that marks the tile as walkable. Here `grass` references `roof`, so actors can walk across grass and avoid everything else.
- `obstacle` points at a `Face` entity that marks the tile as solid. The `wall` references the same `roof`. A tile's faces just say which parts are walkable versus solid.

> Tip: A tile defaults to neither walkable nor solid (both references absent). A floor tile you want actors to cross needs `navigation` pointing at a walkable `Face`, as `grass` shows. Reuse one `Face` entity (like `roof`) across many tiles.

Two more Tile fields exist for special cases. `origin` adjusts what draws in front, and `ghost` makes a tile invisible while it still blocks movement. Leave both off unless you have a specific need; the example uses neither.

> Warning: Do not set `ghost` on your main floor tiles. A ghost tile is never drawn, so a ghosted floor leaves the terrain invisible.

## Draw the layers

A [Layer](/reference/entities.html#layer) is a plain-text file of tile symbols, one character per cell. The file holds your map's shape: where the grass is, where the walls run, where there is nothing at all.

The grid format is deliberately simple. Each non-blank character is matched against a tile's `symbol`. Blank cells — a space, tab, newline, or empty position — place no tile, which is how you leave gaps. A short grass patch with a gap might look like this:

```
GGG
G G
GGG
```

Here the center cell is a space, so no tile is placed there.

The example splits its terrain across three layers, drawn in order from a [TileMap](/reference/entities.html#tilemap):

```yaml
TileMap:
  demoTileMap:
    tileset: demoTileSet
    layers:
    - demoFloor
    - demoWater
    - demoWalls
Layer:
  demoFloor:
    source: /demoMap/layer0
    ysort: false
  demoWater:
    source: /demoMap/layer1
    ysort: true
  demoWalls:
    source: /demoMap/layer2
    ysort: true
```

Each layer's `source` is a path to a grid file, relative to the campaign root. In the example these files live under a `demoMap/` folder inside the campaign and have no file extension: `layer0`, `layer1`, `layer2`. The floor layer draws grass, the walls layer draws the maze border, and the water layer is blank in this map.

The `ysort` field controls what draws in front, so taller terrain like walls draws in front of what is behind it. Leave it off for a flat floor, as `demoFloor` does, and on for upright features.

> Note: Layers draw in the order you list them. Put your floor first and features such as walls after, so the features sit on top.

### How grid characters become tile positions

The engine reads the grid like this:

- **Line number `L` of the file is map column `x = L`** (the first line is `x = 0`).
- **Character number `C` on that line is map row `y = -C`** (the first character is `y = 0`, the next `y = -1`, and so on).

On screen, `x` runs down-right and `y` runs down-left, so the shape in your text file appears **rotated** in game: each line of the file becomes a diagonal running up-right. Expect to check your layout in game rather than trusting the text file's look.

To place things (spawn points, deployments) on a cell, convert the cell to pixels. The centre of the walkable top of cell `(x, y)` is:

```
pixel x = 16 * (x - y) + 16
pixel y = 8 * (x + y)
```

Each cell is 32 px wide and 16 px tall on screen.

## Place the spawn point

The player needs somewhere to enter. The Map's `spawn` field is a [KeyRef](/manual/getting-started/glossary.html#terms) to a [Vector](/reference/entities.html#vector) entity that holds an `x` and `y` position in pixels. It is the fallback location for the player actor when no saved position or action override applies.

The spawn Vector sits beside the Map entity in the same file:

```yaml
Vector:
  demoSpawn:
    x: 128
    y: 16
```

The Map refers to it by key, as the assembled Map below shows.

## Deploy non-player actors

A [Deployment](/reference/entities.html#deployment) places one non-player actor on the map when the level loads. Each deployment pairs an `actor` to spawn with a `location` Vector that says where. The engine spawns each deployed actor as an NPC.

The example places three NPCs:

```yaml
Deployment:
  guardDeploy:
    location: guardPos
    actor: guardActor
  merchantDeploy:
    location: merchantPos
    actor: merchantActor
  patrollerDeploy:
    location: patrollerPos
    actor: patrollerActor
Vector:
  guardPos:
    x: 128
    y: -48
  merchantPos:
    x: 160
    y: 32
  patrollerPos:
    x: 288
    y: 0
```

Each `actor` is a full [Actor](/reference/entities.html#actor) entity defined elsewhere in the campaign. To give a deployed NPC its own behavior, give that actor a strategy; see [Add NPC AI](/manual/guides/add-npc-ai.html).

## Assemble the map

With the tiles, layers, spawn, and deployments in place, the Map entity ties them together:

```yaml
Map:
  demoMap:
    name: Demo Map
    tilemap: demoTileMap
    spawn: demoSpawn
    deployments:
    - guardDeploy
    - merchantDeploy
    - patrollerDeploy
Vector:
  demoSpawn:
    x: 128
    y: 16
```

This Map names itself for the interface, draws its terrain from `demoTileMap`, enters the player at `demoSpawn`, and deploys the three NPCs. To make this map the one a campaign starts on, point the Main entity's map field at it; see [Campaign structure](/manual/core-concepts/campaign-structure.html).

The Map also carries `background` and `audio` fields for parallax scenery and music. Those are covered in [Audio and backgrounds](/manual/guides/audio-and-backgrounds.html).

## Walls, trees, and other tall things

Tiles are for **flat ground**. Anything that stands up (a wall, a tree, a lamp post, a house, a chest) should be an **Actor** with a single still image, placed with a Deployment. Actors are depth-sorted against the player, so the player correctly walks behind a tree and in front of it. A tall tile can't do that.

Set `targetable: false` on these scenery actors so players can't tab-target or click them, and give them `salience` of at least `1` (salience `0` is never drawn).

Actors never block movement, so a solid prop also needs an invisible wall under it: a tile with `obstacle` set to a Face and `ghost: true`, in a layer under the prop's footprint. `ghost` hides the tile while it still blocks movement.

```yaml
Tile:
  blockerTile:
    symbol: '#'
    index: 0
    obstacle: roof
    ghost: true
Actor:
  oakTreeActor:
    name: Oak
    sprite: oakTreeSprite
    targetable: false
    salience: 1
```

> Tip: Don't hide important actors (quest givers, shopkeepers) behind tall props. Players can't see them through the art, so leave them a clear spot in front.

## Help walkers round corners (optional)

Walkers plan their routes over the walkable tiles and bend their path at corners. The corners of walls and
obstacles are found for you. Where a route still looks wrong, such as a narrow doorway or a gap between props, add a
[`Nav`](/reference/entities.html#nav) point: a [`Vector`](/reference/entities.html#vector) on a given
[`Map`](/reference/entities.html#map) where walkers may also bend their path.

```yaml
Nav:
  smithyDoorNav:
    map: townMap
    location: smithyDoorVector
```

Put the point in the middle of the opening, on walkable ground: a point off the walkable area or inside an
obstacle is skipped with a warning when the map is built. See
[Help pathfinding round corners with Nav points](/recipes/corner-pathing-with-nav.html).

## Add floor overlays (optional)

A [Floor](/reference/entities.html#floor) draws a single texture image at a fixed position, beneath the tiles. Use it for a backdrop or a decorative surface that the tile grid sits on top of. A Floor has a `texture` path and a `location` KeyRef to a Vector for its top-left corner. The Map's `floor` field lists the floor overlays to draw. The example map uses none, so you can skip this until you need a painted backdrop.

## Verify the map loads

Zip your campaign and check that the map loads and validates:

```bash
./isometry --validate=/path/to/mygame.zip
./isometry --campaign=mygame --network=host --port=5000 \
  --username=p --secret=p --log-level=INFO
```

A successful load logs `Campaign validation passed` and `Successfully created Entity` lines. Remember the fog of war: a loaded map looks dark until an actor's discovery area reveals the tiles near it.

## See also

- [Create an actor](/manual/guides/create-an-actor.html)
- [Add NPC AI](/manual/guides/add-npc-ai.html)
- [Visuals: sprites and animations](/manual/guides/visuals-sprites-animations.html)
- [Audio and backgrounds](/manual/guides/audio-and-backgrounds.html)
- [Entity reference](/reference/entities.html)
- [Tutorial: build the demo](/manual/tutorial/build-the-demo.html)
