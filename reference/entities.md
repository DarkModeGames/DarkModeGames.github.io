{% raw %}
# Entity Reference

Every one of the 36 entity types and its fields. Each block is a ready-to-copy YAML scaffold — required fields carry a default, optional fields are `null` (and dissolve away on load). Generated from the engine.

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
    # Seconds, as a float. On a skill's start action, how long the caster is locked. On a `wait` action, how long before its `then` runs. Elsewhere unused.
    time: 0.0  # Float
```

<a id="actor"></a>
## Actor

Actor Entity A controllable or AI-driven character in the game world.

```yaml
Actor:
  exampleActor:
    # Size of actor's base circle in pixels. An actor with base 0 (the default) has no base collider, so area effects never hit it.
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
    # Vision range in pixels: the radius across of the actor's sight ellipse (half that up and down, the isometric ground). It also sets how far fog of war lifts around the player. Example: 250.
    perception: null  # Integer
    # Private resources visible only on own data plate.
    private: null  # Array[Resource]
    # Public resources visible on target focus plate when other actors view this actor.
    public: null  # Array[Resource]
    # Resource entities associated with this actor.
    resources: null  # Array[Resource]
    # How far away, in pixels, others can notice this actor: the radius across of its own ellipse (half that up and down). Another actor sees it when their sight ellipses overlap. 0 = never seen or drawn. Example: 10.
    salience: null  # Integer
    # Skills available to actor. Maximum 9 skills.
    skills: null  # Array[Skill]
    # Optional Skin entity applying a color remap to this actor's sprite texture at render time. Lets many Actor entities share one Sprite/texture with different recolors. Omit for the sprite's unmodified colors.
    skin: null  # Skin
    # Movement speed multiplier. Defaults to 0: an actor without speed cannot move. 1.0 is normal walking pace, 2.0 twice as fast, 0.5 half.
    speed: 0.0  # Float
    # Reference to Sprite entity for visual representation.
    sprite: null  # Sprite
    # Strategy entity for AI behavior control.
    strategy: null  # Strategy
    # If false, the actor never enters targeting: no tab or click selection, no view-based target_* actions, no on_view for observers. For decorative actors such as walls, trees and torches.
    targetable: true  # Boolean
    # Timer entities that execute actions at intervals or after duration.
    timers: null  # Array[Timer]
    # Trigger entities that monitor resource changes and execute actions.
    triggers: null  # Array[Trigger]
    # Talent trees this actor learns from in the Talents menu (ticket 264).
    trees: null  # Array[Tree]
    # Action run when a player interacts with this actor (right-click, the Interact flag, Enter), with caller = the player and target = this actor, instead of walking up to open its dialog or menu (ticket 267). E.g. start_auto_interact on something the player should keep acting on (an enemy, a crop, a patient).
    on_interact: null  # Action
    # Action this actor runs when another actor acts on it (ticket 302): after any action chain in which another actor's action acted on this one (a `_target` action or transfer_resource; an area effect counts for each actor it reaches), with caller = this actor and target = the actor who acted. Once per chain; what a reaction does never causes reactions. Filter with its `if` (e.g. fight back only at enemies: `@is_enemy == 1`). Players can turn their own off in the options ("React automatically").
    reaction: null  # Action
    # Skill this actor repeats while auto-interacting with its target (its `start` action is one repeat: a sword swing, a heal, a harvest; tickets 268, 301).
    auto_interact: null  # Skill
    # How close the target must be for each repeat, in isometric pixels; farther away the actor walks in. 0 = bases touching.
    auto_interact_range: 0  # Integer
    # Seconds between repeats. Default 2.0.
    auto_interact_interval: 2.0  # Float
    # How fast it repeats, in percent (ticket 303): a dice expression read before each repeat ($ = this actor, @ = its target). The wait between repeats is auto_interact_interval x 100 / speed: '100' (the default) keeps it, '125' is a quarter faster, '50' twice as slow. E.g. '100+$haste' for haste. Floored at 10.
    auto_interact_speed: '100'  # Dice
    # Image shown for this actor on the HUD, e.g. its chip on the minion strip (ticket 273). Optional.
    icon: null  # String
    # The Character page: equipment slots and stat lines (ticket 282). Optional.
    sheet: null  # Sheet
```

<a id="animation"></a>
## Animation

Animation Entity 8-directional sprite animation with frame data for each direction.

```yaml
Animation:
  exampleAnimation:
    # Frame array for east (90°) direction. Default empty.
    E: []  # Array[Integer]
    # Frame array for north (0°) direction. Default empty.
    N: []  # Array[Integer]
    # Frame array for northeast (45°) direction. Default empty.
    NE: []  # Array[Integer]
    # Frame array for northwest (315°) direction. Default empty.
    NW: []  # Array[Integer]
    # Frame array for south (180°) direction. Default empty.
    S: []  # Array[Integer]
    # Frame array for southeast (135°) direction. Default empty.
    SE: []  # Array[Integer]
    # Frame array for southwest (225°) direction. Default empty.
    SW: []  # Array[Integer]
    # Frame array for west (270°) direction. Default empty.
    W: []  # Array[Integer]
    # If true, animation repeats continuously. If false, it plays once and holds its last frame. Default true.
    loop: true  # Boolean
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
    # Animation played while a flying actor is stationary (the airborne idle).
    hover: null  # Animation
    # Which animation in this set fills each engine auto-state. Lets an actor name its animations anything (e.g. barb_ogre_idle) instead of the literal idle/walk/run keys.
    idle: null  # Animation
    # Human-readable animation set name.
    name: null  # String
    # Animation used for fast ground movement (running).
    run: null  # Animation
    # Animation used for slow ground movement (walking).
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
    # Left operand of comparison. Supports a plain number, a dice expression (e.g. "2d6+3"), or a $/@ resource reference.
    left: null  # String
    # Comparison operator (==, !=, <, >, <=, >=). NOT NULL.
    operator: ''  # String
    # Right operand of comparison. Supports a plain number, a dice expression (e.g. "2d6+3"), or a $/@ resource reference.
    right: null  # String
    # Other Conditions that must all hold too, for the same caller and target (ticket 281): one Condition can then test several things at once, e.g. a menu option shown only while a quest is done AND the next one isn't started.
    all: []  # Array[Condition]
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
    # Seconds after its actor leaves the world for good (despawns; a change of form doesn't count) before the deployment spawns again, fresh, at its location. It waits while a player stands near the spot. 0 (the default) means never.
    respawn: 0  # Integer
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
    # Image shown for the group in the HUD's target-group row above the target plate (a click picks the group for Tab / E / Q). Optional: without one the group shows a dot in its colour.
    icon: null  # String
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
    # Player-controlled Actor entity. Optional — omit it and the player starts as a built-in null actor: invisible, immovable, infinite-reach, and all-seeing: every actor on its map is visible and targetable to it (salience 0 still hides an actor), however far away.
    actor: null  # Actor
    # Starting Map entity where campaign begins.
    map: ''  # Map
    # Campaign notes or description. Default empty.
    notes: ''  # String
    # The campaign's credits, as BBCode ([b], [i], [br], [color], [url] ...). Players read them from the System menu's Credits button. Default empty.
    credits: ''  # String
    # Plugin resolution order (nested-zip basenames); earlier = higher priority. The campaign top-level always wins; unlisted plugins rank lowest. Default empty.
    plugins: []  # Array[String]
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
    # TileMap entity defining the terrain grid. Required: a Map without one fails validation.
    tilemap: ''  # TileMap
```

<a id="mask"></a>
## Mask

Mask Entity Maps one color to another for use in a Skin's color remap.

```yaml
Mask:
  exampleMask:
    # Hex color code to match on the sprite texture (e.g. "#FF0000").
    from_color: null  # String
    # Hex color code to remap matching pixels to.
    to_color: null  # String
```

<a id="measure"></a>
## Measure

Measure Entity A calculated value based on dice expressions, associated with an actor.

```yaml
Measure:
  exampleMeasure:
    # Description shown in the HUD tooltip. Optional; empty leaves the line out.
    description: null  # String
    # Dice expression to calculate value (e.g., "2d6+3").
    expression: null  # Dice
    # Icon path for UI display.
    icon: null  # String
    # Menu entity for interaction options.
    menu: null  # Menu
    # Display name shown in the HUD tooltip and menus. Optional; empty falls back to the key.
    name: null  # String
    # If true, measure is shown on own data plate.
    private: null  # Boolean
    # If true, measure is shown on target focus plate when other actors view this actor.
    public: null  # Boolean
    # Visibility threshold. Measure only visible when value >= reveal. 0 = always visible.
    reveal: null  # Integer
    # Short words that sort this entry into categories in the resources view, e.g. [gear, head] or [material, ore]. The view shows one filter chip per tag. Optional.
    tags: []  # Array[String]
```

<a id="menu"></a>
## Menu

Menu Entity A collection of action options for interaction.

```yaml
Menu:
  exampleMenu:
    # Action entities available in this menu.
    actions: null  # Array[Action]
    # When true, an option whose Action has an `if` Condition is listed only while that Condition holds for the player (caller) and the menu's target. Options without an `if` are always listed. If every option is hidden, the menu says "Nothing here." The host works the list out and keeps it current while the menu is open, so the Conditions may read any of the target's resources, public or not.
    hide_unavailable: false  # Boolean
    # Human-readable menu name.
    name: null  # String
```

<a id="nav"></a>
## Nav

Nav Entity An authored point for a map's pathfinding graph: walkers may bend their path here. Corners of walls and obstacles are found automatically; place Navs where that isn't enough, e.g. in a doorway or tight custom terrain. A point off the navmesh or inside an obstacle is skipped with a warning when the map is built, and a point too tight for a body is left out of that body's paths.

```yaml
Nav:
  exampleNav:
    # Vector entity giving the point's position on that map (x, y).
    location: ''  # Vector
    # Map entity this point belongs to.
    map: ''  # Map
```

<a id="overlay"></a>
## Overlay

Overlay Entity A full-screen animated foreground effect (rain, fire, fog) shown between the game world and the UI, fixed to the camera. Shown/hidden by triggers via the overlay action functions.

```yaml
Overlay:
  exampleOverlay:
    # Frame indices into the sheet, in play order.
    frames: []  # Array[Integer]
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
    frames: []  # Array[Integer]
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
    # If true, changes to this resource float up as numbers over the actor (combat text) for its own player, the player who caused the change, and players targeting or focusing the actor. Default false.
    combat_text: null  # Boolean
    # Default starting value.
    default: null  # Integer
    # Description text displayed in UI.
    description: null  # String
    # Dialog entity shown when a player interacts with this resource.
    dialog: null  # Dialog
    # Combat text colour for a gain, a hex code or colour name (optional; default green).
    gain_color: null  # String
    # Icon path for UI display.
    icon: null  # String
    # Combat text colour for a loss, a hex code or colour name (optional; default red).
    loss_color: null  # String
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
    # Short words that sort this entry into categories in the resources view, e.g. [gear, head] or [material, ore]. The view shows one filter chip per tag. Optional.
    tags: []  # Array[String]
```

<a id="sheet"></a>
## Sheet

Sheet Entity The Character page of an Actor (ticket 282): equipment Slots laid out on a grid, and Stat lines summing up the character. An Actor names it in `sheet`; players open it with the Character key.

```yaml
Sheet:
  exampleSheet:
    # A line under the title.
    description: null  # String
    # The page's title.
    name: null  # String
    # Equipment cells, each at its own column and row.
    slots: []  # Array[Slot]
    # Summary lines, each at its own column and row.
    stats: []  # Array[Stat]
```

<a id="slot"></a>
## Slot

Slot Entity One equipment cell on a Character page (ticket 282). It shows the first of its `worn` Resources that is above 0 (that Resource's icon and name); empty, it shows its own icon and name. Picking it lists the `choices` the player owns (above 0); each opens that Resource's own menu (Equip, Unequip, Drop: whatever the campaign gave it).

```yaml
Slot:
  exampleSlot:
    # Item Resources that fit this slot; those the player owns are listed when the slot is picked.
    choices: []  # Array[Resource]
    # Column on the page's grid, from 0 at the left.
    column: 0  # Integer
    # Image shown while the slot is empty.
    icon: null  # String
    # The slot's name, shown while it is empty ("Main hand").
    name: null  # String
    # If true, `worn` and `choices` are pairs by position: `worn[i]` is above 0 while `choices[i]` is worn. The page then marks the worn item in the list of choices. Both lists must then be the same length (ticket 298).
    paired: false  # Boolean
    # Row on the page's grid, from 0 at the bottom.
    row: 0  # Integer
    # Resources that mean "something is worn here" (e.g. one `<item>_equipped` flag per item); the first above 0 shows.
    worn: []  # Array[Resource]
```

<a id="stat"></a>
## Stat

Stat Entity One summary line on a Character page (ticket 282): a name and a value read from the player.

```yaml
Stat:
  exampleStat:
    # Column of the stats grid, from 0 at the left.
    column: 0  # Integer
    # What the line explains, shown when it is hovered.
    description: null  # String
    # Image beside the label.
    icon: null  # String
    # The line's label ("Strength").
    name: null  # String
    # Row of the stats grid, from 0 at the top.
    row: 0  # Integer
    # The value: a dice expression over the player's resources and measures, e.g. "$core_strength_measure".
    value: ''  # String
```

<a id="skill"></a>
## Skill

Skill Entity A skill action available to an actor, typically bound to hotkeys.

```yaml
Skill:
  exampleSkill:
    # Maximum charge value for charging skills. 0 = no charging. Default 0.
    charge: 0  # Integer
    # Description text displayed in UI tooltips. Default empty. {{$key}} shows the reader's values; {{dice: <dice>}} shows it as dice plus one number ("1d6+16") and {{range: <dice>}} the lowest and highest result ("17–22") (ticket 288).
    description: ''  # String
    # Action triggered when skill button is released.
    end: null  # Action
    # A Condition on the player (caller and target) that holds when the skill can be cast now (its cost, its cooldown). While it fails, the skill's hotbar slot is greyed (ticket 289). Optional.
    ready: null  # Condition
    # A Condition with the player as caller and the current target as target that holds when the target is close enough. While a target is held and it fails, the slot is darkened too (ticket 289). Optional.
    in_range: null  # Condition
    # Icon path for UI display in skill bar.
    icon: null  # String
    # Human-readable name displayed in UI.
    name: null  # String
    # Action triggered when skill button is pressed.
    start: null  # Action
```

<a id="skin"></a>
## Skin

Skin Entity A named set of color remaps (Masks) applied to an Actor's sprite texture at render time. Masks match the sprite's own colours, unaffected by any tint; a tint (set_modulate_*) then colours the skinned result.

```yaml
Skin:
  exampleSkin:
    # Color remaps applied, in order. Maximum 16 (shader uniform array limit).
    masks: null  # Array[Mask]
    # Display name for this skin.
    title: null  # String
```

<a id="sound"></a>
## Sound

Sound Entity An audio file with pitch, volume, reach and looping options.

```yaml
Sound:
  exampleSound:
    # If true, audio loops continuously.
    loop: null  # Boolean
    # How far a sound placed in the world carries, in pixels (default 2000). Ignored by map audio, which isn't placed.
    range: null  # Integer
    # Pitch in percent, as a dice expression: 100 is normal pitch, 200 an octave up, 50 an octave down. Rolled again each time the sound plays, so '92+1d16' varies it a little. Integers only.
    scale: null  # Dice
    # Path to audio file.
    source: null  # String
    # Volume in percent, a whole number from 0 to 400: 100 plays the file as it is, 50 at half amplitude, 200 at double. Default 100.
    volume: 100  # Integer
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

<a id="quest"></a>
## Quest

Quest Entity A quest in the player's Quests page (ticket 275). Its state is one Resource the campaign already sets: 0 means not started (not listed); the Step with the highest `value` at or below the resource's value is the current one; a Step with `done: true` moves the quest to Completed. No actions of its own.

```yaml
Quest:
  exampleQuest:
    # A line about the quest, shown under its name.
    description: null  # String
    # Image shown beside the quest in the list.
    icon: null  # String
    # The quest's name in the Quests page and the tracker.
    name: null  # String
    # Where the quest sits in the lists (lowest first; then by name). Default 0.
    order: 0  # Integer
    # Resource holding the quest's progress on the player (0 = not started).
    resource: ''  # Resource
    # The quest's steps, each shown while the resource is at or past its value.
    steps: []  # Array[Step]
```

<a id="step"></a>
## Step

Step Entity One step of a Quest (ticket 275): the text the player reads while the quest's resource is at `value` (or past it, until a later step's value is reached).

```yaml
Step:
  exampleStep:
    # True when reaching this step finishes the quest (it moves to Completed; its text is the ending).
    done: false  # Boolean
    # What the player must do now, in a line or two. {{$key}} shows the player's resources (e.g. a kill count).
    text: ''  # String
    # The quest resource's value from which this step is the current one.
    value: 0  # Integer
```

<a id="talent"></a>
## Talent

Talent Entity One node of a talent Tree (ticket 264). Its rank is an ordinary Resource, so skills, measures, conditions and Triggers read it like any other value. Points go in from the bottom row up: a talent opens once every talent in `requires` has a rank.

```yaml
Talent:
  exampleTalent:
    # Column in the tree's grid, from 0 at the left.
    column: 0  # Integer
    # Points spent for each rank. Default 1.
    cost: 1  # Integer
    # What the talent does, shown in its tooltip.
    description: null  # String
    # Image for the talent's tile.
    icon: null  # String
    # Condition on the player (caller and target are the player) that must pass to learn a rank, e.g. a level gate.
    if: null  # Condition
    # How many ranks the talent has. Default 1.
    max: 1  # Integer
    # Human-readable talent name.
    name: null  # String
    # Action run on the player (caller and target) after each rank is learned, e.g. set_skill_self to grant a skill.
    on_learn: null  # Action
    # Talents that need at least one rank before this one opens; the menu draws a line from each to this one.
    requires: []  # Array[Talent]
    # Resource holding this talent's rank (0 = not learned). Give it min 0 and a max of at least `max`.
    resource: ''  # Resource
    # Row in the tree's grid, from 0 at the bottom.
    row: 0  # Integer
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
    # Layer entities defining tile placement and rendering order. Required (an empty list is allowed).
    layers: []  # Array[Layer]
    # TileSet entity containing tile definitions. Required: a TileMap without one fails validation.
    tileset: ''  # TileSet
```

<a id="tileset"></a>
## TileSet

TileSet Entity A collection of tiles from a texture atlas arranged in a grid.

```yaml
TileSet:
  exampleTileSet:
    # Number of columns in the tile grid (at least 1). Required.
    columns: 1  # Integer
    # Path to tileset texture file. Required.
    texture: ''  # String
    # Tile entities defining individual tiles in the set. Required (an empty list is allowed).
    tiles: []  # Array[Tile]
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

<a id="tree"></a>
## Tree

Tree Entity A talent tree (ticket 264): talents laid out in a grid, learned from the bottom row up with the points in one Resource. An Actor lists its trees in `trees`; players open them in the Talents menu.

```yaml
Tree:
  exampleTree:
    # What the tree is about, shown under its name.
    description: null  # String
    # Image for the tree's tab.
    icon: null  # String
    # Human-readable tree name (the tab and the panel title).
    name: null  # String
    # Resource holding the points this tree spends: the campaign grants them (e.g. a Trigger on level-up).
    points: ''  # Resource
    # The talents in this tree.
    talents: []  # Array[Talent]
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
