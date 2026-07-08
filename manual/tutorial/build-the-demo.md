# Tutorial: Build the Demo Campaign

This tutorial builds a small example [campaign](/manual/getting-started/glossary.html) called `demo` from scratch, one file at a time, then validates and boots it. By the end you will have written every kind of entity a small game needs and watched the engine load it.

## Introduction

{{ product_name }} games are pure data. You do not write code to make a game; you write YAML [entity](/manual/getting-started/glossary.html) files and package them into a `.zip`. This page walks the whole path: a starting point, a map made of tiles, a playable hero, an action the hero can take, an AI-driven enemy, a waypoint marker, and finally the validate-and-boot step that proves it all hangs together.

Each step shows a real file, so the files you write form a single coherent, working campaign.

## Prerequisites

You should have worked through these pages first:

- [Install and run](/manual/getting-started/install-and-run.html) — get the engine running.
- [Your first campaign](/manual/getting-started/your-first-campaign.html) — the smallest campaign that loads.
- [Campaign structure](/manual/core-concepts/campaign-structure.html) — how a campaign folder is laid out.
- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html) — how entities reference each other.

> Note: Entity YAML files always use **clean keys** — `name`, `min`, `max`, `default`. The engine has internal aliases (`name_` and so on) but those never appear in campaign files. Writing `name_` in a file is an error.

## How a campaign fits together

A campaign is a folder of YAML files. Each file holds one or more entity types, keyed by an [entity key](/manual/getting-started/glossary.html) you choose. Entities point at each other with [KeyRefs](/manual/getting-started/glossary.html): a field whose value is another entity's key.

There are exactly {{ entity_count }} entity types. You will not use all of them here, but you will touch the ones that form the spine of any game: `Main`, `Map`, `Actor`, `Action`, `Strategy`, and a handful of supporting entities they reference. For definitions of each term, see the [glossary](/manual/getting-started/glossary.html).

Create a folder named `demo` and put each file below inside it. Keep file names as shown; the engine reads every `.yaml` in the folder regardless of name, but matching these keeps things findable.

## Step 1: Define the entry point

Every campaign needs exactly one `Main` entity. It names the player's [actor](/manual/getting-started/glossary.html) and the starting [map](/manual/getting-started/glossary.html) — nothing more is required.

`Main.yaml`:

```yaml
Main:
  demoMain:
    actor: heroActor
    map: demoMap
```

Both `actor` and `map` are KeyRefs. They point at entities you have not written yet; you will fill them in over the next steps. The keys `heroActor` and `demoMap` are referenced again later — use them exactly.

## Step 2: Build the tileset and map

A map is drawn from tiles. You define the tiles once in a `TileSet`, lay them out in text grids, wire those grids into a `TileMap`, then wrap it all in a `Map`.

### Define the tiles

A `TileSet` points at one texture and lists the `Tile` entities cut from it. Each `Tile` records its grid `index`, a one-character `symbol` you will use in the layout files, and `navigation`/`obstacle` references to a `Face` entity — which parts of the tile players can walk on and which block them. Here a shared `roof` (the top of the tile) is what grass is walkable on and what the wall blocks against.

`tileset.yaml`:

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

`columns` is how many tiles wide the texture is, used to turn an `index` into a position in the image. Grass is walkable; walls block movement. A `Face` marks which parts of a tile are walkable or solid, using the directions `northeast`, `east`, `southeast`, `southwest`, `west`, and `northwest` (the top of a tile is `northeast` + `northwest`).

### Draw the layers

A `TileMap` stacks `Layer` entities. Each layer's `source` points at a plain-text file under the campaign folder where every character is a tile `symbol` and a space is empty.

`tilemap.yaml`:

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

`ysort` controls whether tiles on that layer sort against actors by depth — leave it off for the flat floor, on for things actors walk behind.

Create the grid files at `demoMap/layer0`, `demoMap/layer1`, and `demoMap/layer2` (no file extension). A floor file is rows of the grass symbol:

```
GGGGGGGG
GGGGGGGG
GGGGGGGG
```

Each `G` becomes one grass tile; spaces leave a gap.

### Assemble the map

The `Map` ties the tilemap to a `spawn` point (where the player appears), the NPC `deployments`, and optional scenery. Spawn is a KeyRef to a `Vector`, which you define in the same file.

`map.yaml`:

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
    background:
    - skyParallax
    - starsParallax
    audio:
    - bgMusic
Vector:
  demoSpawn:
    x: 128
    y: 16
```

`background` and `audio` are optional; you can drop them while testing. The `deployments` are written in [Step 5](#step-5-add-an-npc-with-ai). For the full map field list, see [Build a map](/manual/guides/build-a-map.html).

## Step 3: Create the hero actor

The hero is an `Actor` — a character in the world. Actors are the richest entity type, but no field is required; you add only what your character needs. The hero needs a look (a `Sprite`), trackable values (`Resource` entities), and things it can do (`Skill` entities). Its hitbox is derived automatically from the sprite.

### Give it a sprite

A `Sprite` points at a texture, an `AnimationSet`, a `Vector` for `size`, and an `Inset` for `margin`. Note the YAML key is `animation_set`, with an underscore.

`sprite.yaml`:

```yaml
Sprite:
  heroSprite:
    animation_set: heroAnimSet
    texture: /assets/astronaut_sprite.png
    size: spriteSize
    margin: spriteMargin
Vector:
  spriteSize:
    x: 64
    y: 64
Inset:
  spriteMargin:
    top: 16
    right: 8
    bottom: 16
    left: 8
```

`size` is the pixel size of one frame; `margin` is the per-edge padding around the art — it seats the feet on the tile and trims the hitbox to the body.

### Animate it

An `AnimationSet` lists `Animation` entities by name. Each `Animation` has eight compass-direction fields (`N`, `NE`, `E`, `SE`, `S`, `SW`, `W`, `NW`), each an array of frame indices into the sprite sheet. Add an optional `sound` to play when the animation runs.

`animation.yaml`:

```yaml
AnimationSet:
  heroAnimSet:
    name: Hero Animations
    animations:
    - idle
    - run
    - tool
Animation:
  idle:
    S:
    - 30
    N:
    - 35
    E:
    - 5
    W:
    - 0
  run:
    S:
    - 22
    - 23
    - 24
    - 23
  tool:
    sound: attackSound
    S:
    - 21
```

A full character fills in all eight directions; the trimmed example above shows the shape. A single-frame array like `"S": [30]` is a still pose; a multi-frame array like `"S": [22, 23, 24, 23]` cycles.

### Track resources

A [`Resource`](/manual/getting-started/glossary.html) is a numeric value on the actor — health, mana, gold. Each has a `default`, a `min`, and a `max`.

`resources.yaml`:

```yaml
Resource:
  health:
    name: Health
    default: 20
    min: 0
    max: 20
    icon: /assets/icons/health.png
    description: Hit points. Reach 0 and you're done.
    menu: healthMenu
  mana:
    name: Mana
    default: 10
    min: 0
    max: 10
  gold:
    name: Gold
    default: 5
    min: 0
    max: 999
```

For visibility and rolled values such as a `luckMeasure` ([`Measure`](/manual/getting-started/glossary.html), expression `1d20`), see [Add resources and measures](/manual/guides/add-resources-and-measures.html).

### Assemble the hero

Now wire it together. The hero references its sprite, resources, skills, and a [group](/manual/getting-started/glossary.html), and sets movement and vision fields. Its hitbox is derived automatically from the sprite, so there is nothing to author.

`heroActor.yaml`:

```yaml
Actor:
  heroActor:
    name: Hero
    speed: 1.0
    sprite: heroSprite
    base: 8
    public:
    - health
    private:
    - health
    - mana
    - gold
    skills:
    - attackSkill
    - healSkill
    perception: 25
    salience: 1
    menu: heroMenu
    resources:
    - health
    - mana
    - gold
    measures:
    - luckMeasure
    triggers:
    - deathTrigger
    timers:
    - regenTimer
    group: playerGroup
Vector:
  heroStartPos:
    x: 200
    y: 100
```

`base` is the footprint circle in pixels, `speed` the movement rate, [`perception`](/manual/getting-started/glossary.html) the vision range, and [`salience`](/manual/getting-started/glossary.html) how easily others spot this actor. `public` and `private` list which resources other players and the owner can see. The `skills` array is capped at nine entries. For a full walk of actor fields, see [Create an actor](/manual/guides/create-an-actor.html).

## Step 4: Define an action and a skill

An [`Action`](/manual/getting-started/glossary.html) changes the game when it runs. Its only required field is `do`, the name of one of the {{ action_count }} built-in action functions. The attack calls `minus_resource_target`, which subtracts a resource from whatever the hero targets.

### The parameters convention

An action passes data to its function through [`Parameter`](/manual/getting-started/glossary.html) entities. The `parameters` field is a **flat array of Parameter keys**, and you define those parameters in a sibling top-level `Parameter` block. Each parameter is a `key` and a `value`, both strings.

`actions.yaml`:

```yaml
Action:
  attackAction:
    name: Attack
    time: 1.1
    animation: tool
    do: minus_resource_target
    parameters:
    - attackResourceParam
    - attackDamageParam
Parameter:
  attackResourceParam:
    key: resource
    value: health
  attackDamageParam:
    key: expression
    value: (1d6)-1
```

Here `time` is the cast duration in seconds and `animation` names the clip to play. The damage is a [dice expression](/manual/getting-started/glossary.html), `(1d6)-1`.

> Warning: Never nest parameters inline like `"parameters": [{ "Parameter": {...} }]`. The `parameters` field is always a flat array of keys, with the definitions in a sibling `Parameter` block.

Actions can branch. A [`Condition`](/manual/getting-started/glossary.html) compares two values; an action's `if` runs the `then` action on a true result and the `else` action otherwise. The `isDead` condition checks `@health <= 0`:

```yaml
Condition:
  isDead:
    left: '@health'
    operator: <=
    right: '0'
```

The `@` token reads a resource on the acting actor. For the full set of functions you can put in `do`, see the [action function reference](/reference/actions.html) and [Define actions](/manual/guides/define-actions.html).

### Wrap the action in a skill

A `Skill` is what appears on the hero's action bar. It points at a `start` action and an `end` action. The attack skill is named `Punch`.

`skills.yaml`:

```yaml
Skill:
  attackSkill:
    name: Punch
    start: attackAction
    end: attackAction
    icon: /assets/icons/sword.png
```

<a id="step-5-add-an-npc-with-ai"></a>
## Step 5: Add an NPC with AI

An NPC is just an `Actor` with a [`Strategy`](/manual/getting-started/glossary.html) — the AI controller. A strategy holds a list of [`Behavior`](/manual/getting-started/glossary.html) entities; the engine checks each behavior's `goals` (a list of conditions) and runs its `action` when the goals are met.

### Write the AI chain

The guard uses `guardStrategy`, which scans for a target, then pursues it. The behaviors reach back into the conditions and actions you already have.

`strategies.yaml`:

```yaml
Strategy:
  guardStrategy:
    behaviors:
    - guardBehavior
    - followBehavior
  followStrategy:
    behaviors:
    - followBehavior
Behavior:
  guardBehavior:
    goals:
    - hasTarget
    action: scanAction
  followBehavior:
    goals:
    - isNearTarget
    action: pursueAction
```

The supporting actions move the NPC. `scanAction` runs `target_nearest` to pick a target, and `pursueAction` runs `move_to_target`. Add these to `actions.yaml`:

```yaml
Action:
  scanAction:
    name: Scan
    do: target_nearest
  pursueAction:
    name: Pursue
    do: move_to_target
```

The matching goal conditions go in your `Condition` block:

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

> Note: For movement to a fixed point rather than a target, the `set_destination_self` function takes a `destination` parameter — a KeyRef to a `Vector`. The chase AI here does not use it, but it is the building block for patrols toward set points. See [Add NPC AI](/manual/guides/add-npc-ai.html).

### Define the guard actor

The guard is a leaner actor than the hero: one resource, an enemy group, and a `strategy`. It reuses the hero's sprite.

`guardActor.yaml`:

```yaml
Actor:
  guardActor:
    name: Guard
    speed: 0.5
    sprite: heroSprite
    base: 8
    public:
    - health
    perception: 15
    salience: 1
    resources:
    - health
    group: enemyGroup
    strategy: guardStrategy
```

This example uses three NPCs that show different strategies: `guardActor` uses `guardStrategy`, `merchantActor` uses `patrolStrategy`, and `patrollerActor` uses `followStrategy`. Match those assignments — they are not interchangeable.

### Place the NPCs

A `Deployment` puts an actor on the map at a `Vector` location. The map's `deployments` array (Step 2) references these.

`deployments.yaml`:

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

## Step 6: Add a waypoint

A `Waypoint` is a named marker on a map, shown to the player to point at a place of interest. It uses the clean key `name` — never the internal alias `name_`.

`waypoints.yaml`:

```yaml
Waypoint:
  townCenter:
    name: Town Center
    location: townCenterPos
    icon: /assets/icons/sun.png
    map: demoMap
    description: The center of the demo town.
Vector:
  townCenterPos:
    x: 300
    y: 200
```

`location` is a `Vector` KeyRef and `map` ties the marker to a specific map. For menus and dialogs that round out the player-facing layer, see [Menus, dialogs, and waypoints](/manual/guides/menus-dialogs-waypoints.html).

### A note on dialogs

A [`Dialog`](/manual/getting-started/glossary.html) is a titled text panel. Its `text` can show live actor values: `{% raw %}{{@resource}}{% endraw %}` reads the viewed actor, `{% raw %}{{$resource}}{% endraw %}` reads the caller. A status dialog:

{% raw %}
```yaml
Dialog:
  statusDialog:
    title: Status Report
    text: 'Health: {{@health}} / 20

      Mana: {{@mana}} / 10

      Gold: {{$gold}}'
```
{% endraw %}

Both `title` and `text` are required.

## Step 7: Validate and boot

Before a campaign loads, the engine validates every entity against its schema and resolves every KeyRef. A typo in a key or a missing required field stops the load with a clear message. See [Validation](/manual/core-concepts/validation.html) for how to read the output.

### Validate the folder

You can check the campaign before packaging it. Point the engine at the `demo` folder with `--validate`; it runs the full validator and exits, printing the result. (A folder is validated without an archive, so asset-path checks are skipped — those run when you validate a `.zip`.)

```bash
isometry --validate=demo
```

A passing run prints `Campaign validation passed` and exits `0`. A failure prints a summary of errors and exits `1`.

### Package the campaign

Package the folder into a `.zip` whose single root folder is named `demo/`, matching the campaign name:

```bash
zip -r demo.zip demo
```

The packaged `.zip` is the unit the engine loads; it must sit beside the binary at runtime.

### Boot it headless

Place `demo.zip` next to the binary, then boot directly into a hosted session. The `--network=host` mode runs the full load-and-validate pipeline; `--network=none` is currently a no-op and loads no campaign.

```bash
isometry --headless --campaign=demo --network=host --port=5000 --username=p --secret=p --log-level=INFO
```

Logs write to `log.txt` in the binary's directory. A clean boot shows two markers:

```
Campaign validation passed
Successfully created Entity
```

`Campaign validation passed` means every schema and KeyRef checked out. `Successfully created Entity` lines confirm the engine built your actors and map from the data. If you see a validation error instead, it names the entity and field at fault — fix it and re-run. For hosting and the network flags in depth, see [Hosting](/manual/multiplayer/hosting.html).

You have now built and booted a full example campaign: an entry point, a tiled map, a playable hero, an action, an AI-driven enemy, and a waypoint — all in data.

## See also

- [Create an actor](/manual/guides/create-an-actor.html)
- [Add resources and measures](/manual/guides/add-resources-and-measures.html)
- [Define actions](/manual/guides/define-actions.html)
- [Build a map](/manual/guides/build-a-map.html)
- [Add NPC AI](/manual/guides/add-npc-ai.html)
- [Menus, dialogs, and waypoints](/manual/guides/menus-dialogs-waypoints.html)
- [Validation](/manual/core-concepts/validation.html)
- [Hosting](/manual/multiplayer/hosting.html)
- [Glossary](/manual/getting-started/glossary.html)
- [Action function reference](/reference/actions.html)
