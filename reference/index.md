# Reference

The complete, code-derived reference for Isometry. The entity and action pages are generated
from the engine's own documentation comments, so they stay in sync with the build.

- [Entities](/reference/entities.html) — all {{ entity_count }} entity types and their fields.
- [Action functions](/reference/actions.html) — all {{ action_count }} action functions and their parameters.
- [Distances and sizes](/reference/units.html) — the one distance measure every field and action uses.
- [CLI reference](/reference/cli.html) — every command-line flag and network mode.
- [Troubleshooting](/reference/troubleshooting.html) — symptoms, causes, and fixes.
- [Philosophy](/reference/philosophy.html) — why Isometry is built this way.

<a id="all-entity-types"></a>
## All entity types

One row per entity type. The first link goes to its fields in the generated reference; the second to the
page that shows it in use.

| Entity | What it is for | Learn it |
|---|---|---|
| [`Action`](/reference/entities.html#action) | One step of game logic: an optional `if`, a `do` function with parameters, then `then` / `else`. | [guide](/manual/guides/define-actions.html) |
| [`Actor`](/reference/entities.html#actor) | Anything that stands in the world: players, monsters, NPCs, props. | [guide](/manual/guides/create-an-actor.html) |
| [`Animation`](/reference/entities.html#animation) | One clip: frame lists for all eight directions, speed, looping, an optional sound. | [guide](/manual/guides/visuals-sprites-animations.html) |
| [`AnimationSet`](/reference/entities.html#animationset) | The clips a sprite can play, and which ones play when idle, walking and running. | [guide](/manual/guides/visuals-sprites-animations.html) |
| [`Behavior`](/reference/entities.html#behavior) | One step of a Strategy: goals, and the action to run while they are not met. | [guide](/manual/guides/add-npc-ai.html) |
| [`Condition`](/reference/entities.html#condition) | A comparison of two dice expressions, used as an action's `if` or a behavior's goal. | [guide](/manual/guides/conditions-and-logic.html) |
| [`Deployment`](/reference/entities.html#deployment) | Places an actor on a map at a Vector when the world is built. | [guide](/manual/guides/build-a-map.html) |
| [`Dialog`](/reference/entities.html#dialog) | A titled text panel shown to the player; text can show live values. | [guide](/manual/guides/menus-dialogs-waypoints.html) |
| [`Face`](/reference/entities.html#face) | Which of a tile's six faces count as walkable, wall or boundary. | [guide](/manual/guides/build-a-map.html) |
| [`Floor`](/reference/entities.html#floor) | A picture drawn under a map's tiles at a Vector. | [guide](/recipes/backgrounds-and-floors.html) |
| [`Group`](/reference/entities.html#group) | A faction with a name and outline colour; used for targeting by group. | [guide](/manual/guides/menus-dialogs-waypoints.html) |
| [`Inset`](/reference/entities.html#inset) | Padding on a sprite frame's four edges; seats the art's feet on the ground. | [guide](/manual/guides/visuals-sprites-animations.html) |
| [`Layer`](/reference/entities.html#layer) | One grid of tile symbols in a TileMap. | [guide](/manual/guides/build-a-map.html) |
| [`Main`](/reference/entities.html#main) | The campaign's entry point: the starting map and the players' starting actor. | [guide](/manual/core-concepts/campaign-structure.html) |
| [`Map`](/reference/entities.html#map) | A playable area: its tiles, spawn point, deployments, background, overlays and audio. | [guide](/manual/guides/build-a-map.html) |
| [`Mask`](/reference/entities.html#mask) | Swaps one colour for another inside a Skin. | [guide](/recipes/recolour-with-skins.html) |
| [`Measure`](/reference/entities.html#measure) | A value worked out from a dice expression each time it is read. | [guide](/manual/guides/add-resources-and-measures.html) |
| [`Menu`](/reference/entities.html#menu) | A list of actions offered as options in the interaction panel. | [guide](/manual/guides/menus-dialogs-waypoints.html) |
| [`Nav`](/reference/entities.html#nav) | An extra point where walkers may bend their path, such as a doorway. | [guide](/recipes/corner-pathing-with-nav.html) |
| [`Overlay`](/reference/entities.html#overlay) | A full-screen animated effect between the world and the interface, such as rain. | [guide](/manual/guides/effects-overlays-particles.html) |
| [`Parallax`](/reference/entities.html#parallax) | A scrolling background layer behind a map. | [guide](/manual/guides/audio-and-backgrounds.html) |
| [`Parameter`](/reference/entities.html#parameter) | A key and value passed to an action's function. | [guide](/manual/guides/define-actions.html) |
| [`Particle`](/reference/entities.html#particle) | A small animated sprite attached to one actor, such as a glow. | [guide](/manual/guides/effects-overlays-particles.html) |
| [`Resource`](/reference/entities.html#resource) | A stored number every actor holds: health, gold, an item count, a flag. | [guide](/manual/guides/add-resources-and-measures.html) |
| [`Skill`](/reference/entities.html#skill) | A hotbar ability: the action run on press, an optional one on release, and an optional charge. | [guide](/manual/guides/add-skills.html) |
| [`Skin`](/reference/entities.html#skin) | A list of Masks that recolours an actor's sprite. | [guide](/recipes/recolour-with-skins.html) |
| [`Sound`](/reference/entities.html#sound) | An audio file with looping and pitch, for a map's ambience or an animation. | [guide](/manual/guides/audio-and-backgrounds.html) |
| [`Sprite`](/reference/entities.html#sprite) | A sprite sheet: texture, frame size, margin and its AnimationSet. | [guide](/manual/guides/visuals-sprites-animations.html) |
| [`Strategy`](/reference/entities.html#strategy) | An NPC's AI: an ordered list of Behaviors. | [guide](/manual/guides/add-npc-ai.html) |
| [`Tile`](/reference/entities.html#tile) | One cell type in a TileSet: its symbol, picture and faces. | [guide](/manual/guides/build-a-map.html) |
| [`TileMap`](/reference/entities.html#tilemap) | A map's ground: a TileSet and its Layers. | [guide](/manual/guides/build-a-map.html) |
| [`TileSet`](/reference/entities.html#tileset) | A tile atlas texture and the Tiles cut from it. | [guide](/manual/guides/build-a-map.html) |
| [`Timer`](/reference/entities.html#timer) | Runs an action on an actor every `interval` seconds. | [guide](/manual/guides/triggers-and-timers.html) |
| [`Trigger`](/reference/entities.html#trigger) | Runs an action whenever one of the actor's Resources changes. | [guide](/manual/guides/triggers-and-timers.html) |
| [`Vector`](/reference/entities.html#vector) | A position (x, y) on a map, or a size. | [guide](/manual/guides/build-a-map.html) |
| [`Waypoint`](/reference/entities.html#waypoint) | A discoverable, named place on a map, with an optional menu for fast travel. | [guide](/recipes/fast-travel-waypoints.html) |
