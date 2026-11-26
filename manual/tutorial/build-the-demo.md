# Tutorial: Build the Demo Campaign

This tutorial builds a small [campaign](/manual/getting-started/glossary.html) called `demo` from scratch, one file at a time. At the end you package it, validate it and boot it. If you copy every block on this page, you get a campaign that passes validation and loads.

## Introduction

{{ product_name }} games are pure data. You write YAML [entity](/manual/getting-started/glossary.html) files and package them into a `.zip`; you write no code. This page builds:

- a walled map made of tiles;
- a hero you control, with two hotbar skills;
- a guard that hunts the hero;
- a waypoint and a status dialog.

## Prerequisites

You should have worked through these pages first:

- [Install and run](/manual/getting-started/install-and-run.html): get the engine running.
- [Your first campaign](/manual/getting-started/your-first-campaign.html): the smallest campaign that loads.
- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html): how entities point at each other.

> Note: Always write clean field names such as `name`, `min` and `max`. The engine has internal aliases like `name_`, but writing one in a file is an error.

## Step 0: Set up the folder

Make a folder named `demo` with an `assets` folder inside it. Download these three images into `demo/assets/`:

- [ground.png](/assets/tutorial/ground.png): two ground tiles, grass and a stone wall.
- [hero.png](/assets/tutorial/hero.png): the hero, two frames.
- [guard.png](/assets/tutorial/guard.png): the guard, two frames.

Every `.yaml` file below goes directly in `demo/`. The engine reads every `.yaml` file in the folder, whatever its name.

## Step 1: Define the entry point

Every campaign needs exactly one [`Main`](/reference/entities.html#main) entity. It names the starting map and the actor the player controls.

`main.yaml`:

```yaml
Main:
  demoMain:
    actor: heroActor
    map: demoMap
```

`actor` and `map` are [KeyRefs](/manual/getting-started/glossary.html): the keys of entities you write in the next steps.

## Step 2: Build the map

A map is drawn from tiles. You define the tiles in a [`TileSet`](/reference/entities.html#tileset), lay them out in text files, stack those files in a [`TileMap`](/reference/entities.html#tilemap), then wrap it all in a [`Map`](/reference/entities.html#map).

### Define the tiles

A `TileSet` points at one image and lists the [`Tile`](/reference/entities.html#tile) entities cut from it. Each tile is a 32×32 square of the image, counted left to right from `index` 0. Each `Tile` has a one-character `symbol` for the layout files, and a [`Face`](/reference/entities.html#face) that says which part of it is walkable (`navigation`) or solid (`obstacle`). The top of a tile is its `northeast` and `northwest` faces.

`tileset.yaml`:

```yaml
TileSet:
  demoTileSet:
    columns: 2
    texture: /assets/ground.png
    tiles: [grassTile, wallTile]
Face:
  topFace:
    northeast: true
    northwest: true
Tile:
  grassTile:
    symbol: G
    index: 0
    navigation: topFace
  wallTile:
    symbol: X
    index: 1
    obstacle: topFace
```

`columns` is how many tiles wide the image is. Grass is walkable and walls block movement.

### Draw the layers

A `TileMap` stacks [`Layer`](/reference/entities.html#layer) entities. Each layer's `source` is a plain-text file in the campaign. The path must start with `/`.

`tilemap.yaml`:

```yaml
TileMap:
  demoTileMap:
    tileset: demoTileSet
    layers: [floorLayer, wallLayer]
Layer:
  floorLayer:
    source: /maps/floor
    ysort: false
  wallLayer:
    source: /maps/walls
    ysort: true
```

`ysort` decides whether the layer's tiles sort in depth against actors. Leave it off for a flat floor.

Now make a `maps` folder in `demo/` and create two files in it, **with no file extension**. Each character is one tile `symbol`, and a space leaves the cell empty. Each **line** of the file is one step along the map's x axis, and each **character** along a line is one step along y (y counts down from 0, so the map runs up and to the right on screen).

`maps/floor`, ten lines of grass:

```
GGGGGGGGGG
GGGGGGGGGG
GGGGGGGGGG
GGGGGGGGGG
GGGGGGGGGG
GGGGGGGGGG
GGGGGGGGGG
GGGGGGGGGG
GGGGGGGGGG
GGGGGGGGGG
```

`maps/walls`, a ring of wall around the edge (the middle lines are an `X`, eight spaces and an `X`):

```
XXXXXXXXXX
X        X
X        X
X        X
X        X
X        X
X        X
X        X
X        X
XXXXXXXXXX
```

### Assemble the map

The `Map` ties the tilemap to a `spawn` point, where the player appears, and a list of `deployments`, the NPCs placed on it. Positions are [`Vector`](/reference/entities.html#vector) entities in world pixels. On this map the middle of the floor is about `(176, 0)`.

`map.yaml`:

```yaml
Map:
  demoMap:
    name: Demo Map
    tilemap: demoTileMap
    spawn: heroStart
    deployments: [guardDeploy]
Vector:
  heroStart:
    x: 160
    y: -24
```

You write `guardDeploy` in [Step 5](#step-5-add-an-npc-with-ai). For every map field, see [Build a map](/manual/guides/build-a-map.html).

## Step 3: Create the hero

The hero is an [`Actor`](/reference/entities.html#actor). It needs a look (a [`Sprite`](/reference/entities.html#sprite)), some values to track ([`Resource`](/reference/entities.html#resource) entities), and a [group](/manual/getting-started/glossary.html).

### Sprites and animations

A `Sprite` points at an image, an [`AnimationSet`](/reference/entities.html#animationset), and a `Vector` giving the size of one frame. An [`Animation`](/reference/entities.html#animation) lists frame numbers for each of the eight compass directions (`N`, `NE`, `E`, `SE`, `S`, `SW`, `W`, `NW`); all eight are needed. One frame is a still pose, and several frames cycle. The hero and the guard images have the same layout, so they share one `AnimationSet`.

`sprites.yaml`:

```yaml
Sprite:
  heroSprite:
    texture: /assets/hero.png
    animation_set: personAnimSet
    size: frameSize
  guardSprite:
    texture: /assets/guard.png
    animation_set: personAnimSet
    size: frameSize
Vector:
  frameSize:
    x: 32
    y: 32
AnimationSet:
  personAnimSet:
    name: Person
    animations: [standAnim, walkAnim]
    idle: standAnim
    walk: walkAnim
Animation:
  standAnim:
    N: [0]
    NE: [0]
    E: [0]
    SE: [0]
    S: [0]
    SW: [0]
    W: [0]
    NW: [0]
  walkAnim:
    N: [0, 1]
    NE: [0, 1]
    E: [0, 1]
    SE: [0, 1]
    S: [0, 1]
    SW: [0, 1]
    W: [0, 1]
    NW: [0, 1]
```

Note the field is `animation_set`, with an underscore.

### Resources and groups

A [`Resource`](/manual/getting-started/glossary.html) is a number an actor holds, such as health or gold. Give each one a `default`, a `min` and a `max`, and write `min` and `max` as quoted strings. **Every actor automatically has every Resource** at its default, so you never list resources on an actor.

A [`Group`](/reference/entities.html#group) is a team. The guard looks for actors in the players' group.

`resources.yaml`:

```yaml
Resource:
  health:
    name: Health
    default: 20
    min: '0'
    max: '20'
  gold:
    name: Gold
    default: 5
    min: '0'
    max: '999'
Group:
  playerGroup:
    name: Players
    color: '#4080ff'
  enemyGroup:
    name: Enemies
    color: '#ff4040'
```

### Assemble the hero

`hero.yaml`:

```yaml
Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    group: playerGroup
    base: 8
    speed: 1.0
    perception: 250
    salience: 40
    public: [health]
    private: [health, gold]
    skills: [punchSkill, statusSkill]
```

- `base` is the size of the actor's footprint, in pixels.
- `speed` is a multiplier: `1.0` is normal pace. Write the decimal point.
- [`perception`](/manual/getting-started/glossary.html) is how far the actor sees, in pixels, and [`salience`](/manual/getting-started/glossary.html) is how far away others can notice it.
- `public` resources show to other players; `private` ones show only to you.
- `skills` fill the hotbar, keys 1 to 9 in order. You write them next.

For every actor field, see [Create an actor](/manual/guides/create-an-actor.html).

## Step 4: Define actions and skills

An [`Action`](/manual/getting-started/glossary.html) changes the game when it runs. Its `do` field names one of the {{ action_count }} built-in action functions. Data goes in through [`Parameter`](/manual/getting-started/glossary.html) entities: `parameters` is a flat list of [Parameter](/reference/entities.html#parameter) keys, and each Parameter is a `key` and a `value`.

The punch takes health from whoever the hero has targeted. The status action opens a dialog.

`actions.yaml`:

{% raw %}
```yaml
Action:
  punchAction:
    name: Punch
    time: 1.0
    do: minus_resource_target
    parameters: [healthParam, punchDamageParam]
  statusAction:
    name: Status
    do: open_dialog
    parameters: [statusDialogParam]
Parameter:
  healthParam:
    key: resource
    value: health
  punchDamageParam:
    key: expression
    value: 1d6
  statusDialogParam:
    key: dialog
    value: statusDialog
Skill:
  punchSkill:
    name: Punch
    start: punchAction
    end: punchAction
  statusSkill:
    name: Status
    start: statusAction
    end: statusAction
Dialog:
  statusDialog:
    title: Status
    text: "Health: {{$health}} of 20\nGold: {{$gold}}"
```
{% endraw %}

- `time` is how long the action takes, in seconds.
- `1d6` is a [dice expression](/manual/getting-started/glossary.html): one six-sided die.
- A [`Skill`](/reference/entities.html#skill) is a hotbar button. It runs its `start` action.
- A [`Dialog`](/reference/entities.html#dialog) needs a `title` and a `text`. In the text, {% raw %}`{{$health}}`{% endraw %} shows a value of the actor who opened it.
- The engine's YAML reader has no multi-line strings. Keep each text on one line, in double quotes, and write `\n` for a line break.

Press `1` to punch your target and `2` to see your status. For more, see [Define actions](/manual/guides/define-actions.html).

<a id="step-5-add-an-npc-with-ai"></a>
## Step 5: Add an NPC with AI

An NPC is an `Actor` with a [`Strategy`](/manual/getting-started/glossary.html). A strategy is an ordered list of [`Behavior`](/manual/getting-started/glossary.html) entities. Each behavior has `goals` (Conditions) and an `action`, and **the action runs until all its goals are true**. The engine runs the first behavior whose goals are not yet met.

Read the guard's plan as a checklist:

1. Until I have a target, look for the nearest player.
2. Until I am within 40 pixels of it, walk toward it.
3. Until it is down, hit it.

A [`Condition`](/manual/getting-started/glossary.html) compares two values. In a behavior, `$` reads the guard itself and `@` reads its target. `has_target` and `distance_to_target` are built-in values every actor has.

`guard.yaml`:

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
Action:
  findHeroAction:
    do: target_nearest_in_group
    parameters: [playerGroupParam]
  chaseAction:
    do: move_to_target
  hitAction:
    name: Hit
    time: 1.0
    do: minus_resource_target
    parameters: [healthParam, hitDamageParam]
Parameter:
  playerGroupParam:
    key: group
    value: playerGroup
  hitDamageParam:
    key: expression
    value: 1d4
Behavior:
  findBehavior:
    goals: [hasTarget]
    action: findHeroAction
  chaseBehavior:
    goals: [closeToTarget]
    action: chaseAction
  hitBehavior:
    goals: [targetIsDown]
    action: hitAction
Strategy:
  guardStrategy:
    behaviors: [findBehavior, chaseBehavior, hitBehavior]
Actor:
  guardActor:
    name: Guard
    sprite: guardSprite
    group: enemyGroup
    base: 8
    speed: 0.5
    perception: 150
    salience: 40
    public: [health]
    strategy: guardStrategy
Deployment:
  guardDeploy:
    actor: guardActor
    location: guardStart
Vector:
  guardStart:
    x: 176
    y: 32
```

`hitAction` reuses `healthParam` from Step 4; parameters are shared by key. A [`Deployment`](/reference/entities.html#deployment) places an actor at a `Vector` when the world is built. If you run away, `closeToTarget` breaks and the guard chases again. For patrols and more, see [Add NPC AI](/manual/guides/add-npc-ai.html).

## Step 6: Add a waypoint

A [`Waypoint`](/reference/entities.html#waypoint) is a named marker on a map that points the player at a place.

`waypoints.yaml`:

```yaml
Waypoint:
  middleWaypoint:
    name: Middle
    description: The middle of the demo map.
    map: demoMap
    location: middlePos
Vector:
  middlePos:
    x: 176
    y: 0
```

For menus, dialogs and waypoints in depth, see [Menus, dialogs, and waypoints](/manual/guides/menus-dialogs-waypoints.html).

## Step 7: Package, validate and boot

### Package

The engine loads a `.zip` whose single top folder is named after the campaign. From the folder that contains `demo/`:

```bash
zip -r demo.zip demo -x "*.DS_Store"
```

**Repackage the zip after every edit.** The engine only reads the zip.

### Validate

```bash
isometry --validate=/full/path/to/demo.zip
```

The validator checks every field, every KeyRef, every dice expression, and that every image and layer file exists in the zip. A pass prints `Campaign validation passed` and exits `0`. A failure names the entity and field at fault and exits `1`. See [Validation](/manual/core-concepts/validation.html).

### Boot

Put `demo.zip` in the `campaigns` folder next to the engine, or point the engine at its folder with `--dir=/full/path/to/folder`. Then host a game:

```bash
isometry --campaign=demo --network=host --port=5000 --username=p --secret=p
```

You appear on the grass inside the walls, and the guard walks over to fight you. Target it and press `1` to punch.

The engine writes `log.txt` in its data folder (on macOS, `~/Library/Application Support/isometry/`). `isometry --print-paths` shows where the data and campaigns folders are. A clean boot logs `Campaign validation passed` and then `Successfully created Entity` lines. Add `--headless` to boot without a window, for example on a server. See [Hosting](/manual/multiplayer/hosting.html).

You have built a campaign with a tiled map, a hero with skills, a guard with AI, a waypoint and a dialog, all in data.

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
