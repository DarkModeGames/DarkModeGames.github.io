{% raw %}
# Entity Reference

Every one of the 33 entity types and its fields. Each block is a ready-to-copy YAML scaffold — required fields carry a default, optional fields are `null` (and dissolve away on load). Generated from the engine.

<a id="action"></a>
## Action

Action Entity An action used by an actor to effect resources.

```yaml
Action:
  exampleAction:
    # The KeyRef to the animation entity that runs while this action is running. If not null, time must be greater that 0.0
    animation: null  # Animation
    # Function name of action, or empty for an animation-only / chain-only action. When set, must be a defined action function.
    do: null  # String
    # KeyRef to another action. To be run if the `if` condtion fails. Must be a valid action key defined in the campaign.
    else: null  # Action
    # If condition to check if the `do` action should be executed.
    if: null  # Condition
    # Human-readable name of action.
    name: null  # String
    # Array of of parameters. They are expected to match the parameters of the `do` in action.gd
    parameters: null  # Array[Parameter]
    # KeyRef to anther action. To be run if the `if` condition succeeds after the `do` action function is complete.
    then: null  # Action
    # The time this action takes to execute in seconds. Expressed as a float.
    time: 0.0  # Float
```

<a id="actor"></a>
## Actor

Actor Entity A controllable or AI-driven character in the game world.

```yaml
Actor:
  exampleActor:
    # Size of actor's base circle in pixels.
    base: null  # Integer
    # Current facing direction in degrees (0-360). Default north (0).
    bearing: 0  # Integer
    # Dialog entity shown when a player interacts with this actor.
    dialog: null  # Dialog
    # If true, the actor starts flying: it ignores walls and the navmesh (moves straight to its destination) and is stopped only by boundary tiles. Toggle at runtime with the fly_*/land_* actions.
    flying: null  # Boolean
    # Group entity for faction/team membership and outline color.
    group: null  # Group
    # Measure entities associated with this actor.
    measures: null  # Array[Measure]
    # Menu entity for custom interaction options.
    menu: null  # Menu
    # Human-readable name displayed in UI.
    name: null  # String
    # Action triggered when this actor enters a map.
    on_map_entered: null  # Action
    # Action triggered when this actor exits a map.
    on_map_exited: null  # Action
    # Action triggered on the player's first login (login_count == 1).
    on_start: null  # Action
    # Action triggered when another actor touches this actor.
    on_touch: null  # Action
    # Action triggered when this actor is viewed by another actor.
    on_view: null  # Action
    # Vision range in pixels for detecting other actors.
    perception: null  # Integer
    # Private resources visible only on own data plate.
    private: null  # Array[Resource]
    # Public resources visible on target focus plate when other actors view this actor.
    public: null  # Array[Resource]
    # Resource entities associated with this actor.
    resources: null  # Array[Resource]
    # Detection difficulty for other actors (higher = more visible).
    salience: null  # Integer
    # Skills available to actor. Maximum 9 skills.
    skills: null  # Array[Skill]
    # Movement speed in pixels per second.
    speed: null  # Float
    # Reference to Sprite entity for visual representation.
    sprite: null  # Sprite
    # Strategy entity for AI behavior control.
    strategy: null  # Strategy
    # Timer entities that execute actions at intervals or after duration.
    timers: null  # Array[Timer]
    # Trigger entit ies that monitor resource changes and execute actions.
    triggers: null  # Array[Trigger]
```

<a id="animation"></a>
## Animation

Animation Entity 8-directional sprite animation with frame data for each direction.

```yaml
Animation:
  exampleAnimation:
    # Frame array for east (90°) direction. Default empty.
    E: []  # Array
    # Frame array for north (0°) direction. Default empty.
    N: []  # Array
    # Frame array for northeast (45°) direction. Default empty.
    NE: []  # Array
    # Frame array for northwest (315°) direction. Default empty.
    NW: []  # Array
    # Frame array for south (180°) direction. Default empty.
    S: []  # Array
    # Frame array for southeast (135°) direction. Default empty.
    SE: []  # Array
    # Frame array for southwest (225°) direction. Default empty.
    SW: []  # Array
    # Frame array for west (270°) direction. Default empty.
    W: []  # Array
    # If true, animation repeats continuously.
    loop: null  # Boolean
    # Scaler speed that the animation plays at. Default 1
    rate: 1.0  # Float
    # Sound entity to play with this animation.
    sound: null  # Sound
```

<a id="animationset"></a>
## AnimationSet

AnimationSet Entity A named collection of animations for an actor.

```yaml
AnimationSet:
  exampleAnimationSet:
    # Animation entities in this set (idle, walk, attack, etc).
    animations: null  # Array[Animation]
    # Animation played while a flying actor is moving (the airborne walk/run).
    fly: null  # Animation
    # Which animation in this set fills each engine auto-state. Lets an actor name its animations anything (e.g. barb_ogre_idle) instead of the literal idle/walk/run keys.
    idle: null  # Animation
    # Human-readable animation set name.
    name: null  # String
    run: null  # Animation
    walk: null  # Animation
```

<a id="behavior"></a>
## Behavior

Behavior Entity A goal-action pair for AI decision making.

```yaml
Behavior:
  exampleBehavior:
    # Action entity to execute when goals are satisfied.
    action: null  # Action
    # Condition entities that must be met to activate this behavior.
    goals: null  # Array[Condition]
```

<a id="condition"></a>
## Condition

Condition Entity A boolean comparison used in action if/else logic.

```yaml
Condition:
  exampleCondition:
    # Left operand of comparison. Future: dice expression support.
    left: null  # String
    # Comparison operator (==, !=, <, >, <=, >=). NOT NULL.
    operator: ''  # String
    # Right operand of comparison. Future: dice expression support.
    right: null  # String
```

<a id="deployment"></a>
## Deployment

Deployment Entity Defines an actor's initial placement on a map.

```yaml
Deployment:
  exampleDeployment:
    # Actor entity to deploy at this location.
    actor: null  # Actor
    # Vector entity defining spawn position (x, y).
    location: null  # Vector
```

<a id="dialog"></a>
## Dialog

Dialog Entity A text display with title and body content.

```yaml
Dialog:
  exampleDialog:
    # Body text content.
    text: ''  # String
    # Title text displayed prominently.
    title: ''  # String
```

<a id="face"></a>
## Face

Face Entity The six triangular wedges of a tile's isometric-cube silhouette, each enabled or disabled. Referenced by Tile.navigation and Tile.obstacle to place per-wedge navigation and collision shapes. The hexagon is split into six triangles from its center.

```yaml
Face:
  exampleFace:
    # Right vertical side wedge.
    east: null  # Boolean
    # Upper-right half of the top diamond (roof).
    northeast: null  # Boolean
    # Upper-left half of the top diamond (roof).
    northwest: null  # Boolean
    # Lower-right half of the bottom diamond (floor).
    southeast: null  # Boolean
    # Lower-left half of the bottom diamond (floor).
    southwest: null  # Boolean
    # Left vertical side wedge.
    west: null  # Boolean
```

<a id="floor"></a>
## Floor

Floor Entity A texture placed at a specific location on the map floor.

```yaml
Floor:
  exampleFloor:
    # Location Vector defining floor position (x, y).
    location: null  # Vector
    # Path to texture image file.
    texture: null  # String
```

<a id="group"></a>
## Group

Group Entity A faction or team with visual identification color.

```yaml
Group:
  exampleGroup:
    # Hex color code for actor outline and UI display.
    color: null  # String
    # Human-readable group name.
    name: null  # String
```

<a id="inset"></a>
## Inset

Inset Entity Per-edge padding (in pixels) around a sprite's visible art within its frame.

```yaml
Inset:
  exampleInset:
    # Padding below the art (the gap under the feet).
    bottom: null  # Integer
    # Padding left of the art.
    left: null  # Integer
    # Padding right of the art.
    right: null  # Integer
    # Padding above the art.
    top: null  # Integer
```

<a id="layer"></a>
## Layer

Layer Entity A rendering layer in a tilemap defining tile placement data.

```yaml
Layer:
  exampleLayer:
    # Tile placement source data identifying which tiles to place.
    source: null  # String
    # If true, enables Y-sorting for depth-based rendering.
    ysort: null  # Boolean
```

<a id="main"></a>
## Main

Main Entity Campaign configuration defining the starting state. A Main entity is required for each campaign and for each campaign, there can only be one Main entity.

```yaml
Main:
  exampleMain:
    # Main player-controlled Actor entity.
    actor: ''  # Actor
    # Starting Map entity where campaign begins.
    map: ''  # Map
    # Campaign notes or description. Default empty.
    notes: ''  # String
    # Plugin resolution order (nested-zip basenames); earlier = higher priority. The campaign top-level always wins; unlisted plugins rank lowest. Default empty.
    plugins: []  # Array
    # Save-compatibility version. Bump when an entity change would invalidate older player saves; saves from a lower version are migrated, higher are discarded. Default 0.
    version: 0  # Integer
```

<a id="map"></a>
## Map

Map Entity A game level containing terrain, actors, and environmental elements.

```yaml
Map:
  exampleMap:
    # Background audio/music for this map.
    audio: null  # Array[Sound]
    # Parallax background layers.
    background: null  # Array[Parallax]
    # Actor deployments defining initial actor placements.
    deployments: null  # Array[Deployment]
    # Floor textures placed at specific locations.
    floor: null  # Array[Floor]
    # Human-readable name displayed in UI.
    name: null  # String
    # Foreground Overlay effects shown full-screen on map load (rain/fire/fog).
    overlay: null  # Array[Overlay]
    # Spawn point Vector where player enters the map.
    spawn: null  # Vector
    # TileMap entity defining the terrain grid.
    tilemap: null  # TileMap
```

<a id="measure"></a>
## Measure

Measure Entity A calculated value based on dice expressions, associated with an actor.

```yaml
Measure:
  exampleMeasure:
    # Dice expression to calculate value (e.g., "2d6+3").
    expression: null  # Dice
    # Icon path for UI display.
    icon: null  # String
    # Menu entity for interaction options.
    menu: null  # Menu
    # If true, measure is shown on own data plate.
    private: null  # Boolean
    # If true, measure is shown on target focus plate when other actors view this actor.
    public: null  # Boolean
    # Visibility threshold. Measure only visible when value >= reveal. 0 = always visible.
    reveal: null  # Integer
```

<a id="menu"></a>
## Menu

Menu Entity A collection of action options for interaction.

```yaml
Menu:
  exampleMenu:
    # Action entities available in this menu.
    actions: null  # Array[Action]
    # Human-readable menu name.
    name: null  # String
```

<a id="overlay"></a>
## Overlay

Overlay Entity A full-screen animated foreground effect (rain, fire, fog) shown between the game world and the UI, fixed to the camera. Shown/hidden by triggers via the overlay action functions.

```yaml
Overlay:
  exampleOverlay:
    # Frame indices into the sheet, in play order.
    frames: []  # Array
    # If true, the animation repeats; otherwise it holds on the last frame.
    loop: null  # Boolean
    # Starting opacity (0..1); the set_overlay_opacity action can change it at runtime.
    opacity: 1.0  # Float
    # Playback speed scalar over the base FPS (1.0 = unchanged).
    rate: 1.0  # Float
    # Reference to a Vector entity giving one frame's size in the sheet.
    size: null  # Vector
    # Path to the effect's sprite-sheet texture.
    texture: null  # String
    # If true, the texture tiles across the screen (rain/snow); otherwise it stretches to fill.
    tile: null  # Boolean
```

<a id="parallax"></a>
## Parallax

Parallax Entity A background layer with parallax scrolling effect.

```yaml
Parallax:
  exampleParallax:
    # Parallax scroll speed multiplier. Lower values = slower movement.
    effect: null  # Float
    # Path to texture image file.
    texture: null  # String
```

<a id="parameter"></a>
## Parameter

Parameter Entity A key-value pair passed to action functions.

```yaml
Parameter:
  exampleParameter:
    # Parameter key matching the action function argument (clean JSON key is `key`; `key_` backs it because Entity already owns `key()`/`_key`).
    key: ''  # String
    # Parameter value (string representation).
    value: ''  # String
```

<a id="particle"></a>
## Particle

Particle Entity A cosmetic sprite attached to an actor via the attach_particle actions. It follows the actor and optionally drifts (direction degrees + speed). Purely visual — the attached set replicates, so it shows on every client with no extra networking.

```yaml
Particle:
  exampleParticle:
    # Drift direction as a bearing: degrees relative to the way the actor faces (0 = forward, 90 = clockwise). 0..360.
    direction: 0  # Integer
    # Frame indices into the sheet, in play order.
    frames: []  # Array
    # If true, the drift bearing is captured once when the particle is attached (fixed to the facing at that moment); if false (default), it tracks the actor's current facing each frame.
    lock: null  # Boolean
    # If true, the animation repeats; otherwise it holds on the last frame.
    loop: null  # Boolean
    # Starting opacity (0..1).
    opacity: 1.0  # Float
    # Playback speed scalar over the base FPS (1.0 = unchanged).
    rate: 1.0  # Float
    # Reference to a Vector entity giving one frame's size in the sheet.
    size: null  # Vector
    # Drift speed in pixels/second relative to the actor (0 = static, follows the actor).
    speed: 0.0  # Float
    # Path to the particle's sprite-sheet texture.
    texture: null  # String
```

<a id="resource"></a>
## Resource

Resource Entity A trackable numeric value associated with an actor (health, mana, stamina, etc).

```yaml
Resource:
  exampleResource:
    # Default starting value.
    default: null  # Integer
    # Description text displayed in UI.
    description: null  # String
    # Dialog entity shown when a player interacts with this resource.
    dialog: null  # Dialog
    # Icon path for UI display.
    icon: null  # String
    # Maximum allowed value, as dice notation evaluated per-actor (supports $/@ injection).
    max: null  # Dice
    # Menu entity for interaction options.
    menu: null  # Menu
    # Minimum allowed value, as dice notation evaluated per-actor (supports $/@ injection).
    min: null  # Dice
    # Human-readable name displayed in UI.
    name: null  # String
    # Visibility threshold. Resource only visible when value >= reveal. 0 = always visible.
    reveal: null  # Integer
```

<a id="skill"></a>
## Skill

Skill Entity A skill action available to an actor, typically bound to hotkeys.

```yaml
Skill:
  exampleSkill:
    # Maximum charge value for charging skills. 0 = no charging. Default 0.
    charge: 0  # Integer
    # Description text displayed in UI tooltips. Default empty.
    description: ''  # String
    # Action triggered when skill button is released.
    end: null  # Action
    # Icon path for UI display in skill bar.
    icon: null  # String
    # Human-readable name displayed in UI.
    name: null  # String
    # Action triggered when skill button is pressed.
    start: null  # Action
```

<a id="sound"></a>
## Sound

Sound Entity An audio file with volume control and looping options.

```yaml
Sound:
  exampleSound:
    # If true, audio loops continuously.
    loop: null  # Boolean
    # Volume scale multiplier (dice expression, e.g., "1.0" or "1d4*0.25").
    scale: null  # Dice
    # Path to audio file.
    source: null  # String
```

<a id="sprite"></a>
## Sprite

Sprite Entity Visual representation configuration for actors with animation and texture data.

```yaml
Sprite:
  exampleSprite:
    # Reference to AnimationSet entity containing available animations.
    animation_set: null  # AnimationSet
    # Reference to Inset entity: per-edge padding (top/right/bottom/left) around the visible art.
    margin: null  # Inset
    # Reference to Vector entity defining sprite dimensions (width, height).
    size: null  # Vector
    # Path to texture/sprite sheet file.
    texture: null  # String
```

<a id="strategy"></a>
## Strategy

Strategy Entity AI behavior strategy defining goal-based decision making.

```yaml
Strategy:
  exampleStrategy:
    # Behavior entities evaluated to determine AI actions.
    behaviors: null  # Array[Behavior]
```

<a id="tile"></a>
## Tile

Tile Entity A single tile definition with navigation and rendering properties.

```yaml
Tile:
  exampleTile:
    # Reference to a Face entity: which cube faces are a hard boundary that blocks ALL actors, including flying ones (unlike obstacle/walls, which flyers ignore). Absent = none.
    boundary: null  # Face
    # If true, tile is invisible but still functional for pathfinding/collision.
    ghost: null  # Boolean
    # Index in the tileset texture grid.
    index: null  # Integer
    # If true, this tile's navigation faces are kept in their own navigation region (as-is) instead of being merged into the map's combined navmesh — finer pathing control on custom maps.
    isolated: null  # Boolean
    # Reference to a Face entity: which cube faces are walkable. Absent = none.
    navigation: null  # Face
    # Reference to a Face entity: which cube faces collide (walls). Absent = none.
    obstacle: null  # Face
    # Y-sort origin offset for rendering layering.
    origin: null  # Integer
    # Symbol identifier for this tile type.
    symbol: null  # String
```

<a id="tilemap"></a>
## TileMap

TileMap Entity A grid-based map using tiles from a tileset arranged in layers.

```yaml
TileMap:
  exampleTileMap:
    # Layer entities defining tile placement and rendering order.
    layers: null  # Array[Layer]
    # TileSet entity containing tile definitions.
    tileset: null  # TileSet
```

<a id="tileset"></a>
## TileSet

TileSet Entity A collection of tiles from a texture atlas arranged in a grid.

```yaml
TileSet:
  exampleTileSet:
    # Number of columns in the tile grid.
    columns: null  # Integer
    # Path to tileset texture file.
    texture: null  # String
    # Tile entities defining individual tiles in the set.
    tiles: null  # Array[Tile]
```

<a id="timer"></a>
## Timer

Timer Entity Executes an action at intervals or after a total duration.

```yaml
Timer:
  exampleTimer:
    # Action entity to execute at intervals or completion.
    action: null  # Action
    # Interval between action executions in seconds (dice expression).
    interval: null  # Dice
    # Total duration in seconds (dice expression, e.g., "60" or "2d6").
    total: null  # Dice
```

<a id="trigger"></a>
## Trigger

Trigger Entity Monitors a resource and executes an action when the resource changes.

```yaml
Trigger:
  exampleTrigger:
    # Action entity to execute when resource changes.
    action: null  # Action
    # Resource entity to monitor for changes.
    resource: null  # Resource
```

<a id="vector"></a>
## Vector

Vector Entity A 2D coordinate point used for positions, dimensions, and polygon vertices.

```yaml
Vector:
  exampleVector:
    # X coordinate in pixels.
    x: null  # Integer
    # Y coordinate in pixels.
    y: null  # Integer
```

<a id="waypoint"></a>
## Waypoint

Waypoint Entity A discoverable fast-travel location on a map.

```yaml
Waypoint:
  exampleWaypoint:
    # Description text displayed in UI.
    description: null  # String
    # Icon path for map and UI display.
    icon: null  # String
    # Vector entity defining waypoint position (x, y).
    location: null  # Vector
    # Map entity this waypoint belongs to.
    map: null  # Map
    # Menu entity for waypoint interaction options.
    menu: null  # Menu
    # Human-readable waypoint name.
    name: null  # String
```

{% endraw %}
