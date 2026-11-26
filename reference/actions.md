{% raw %}
# Action Function Reference

All 91 action functions. Each block is a ready-to-copy YAML scaffold wiring the function into an Action (and its Parameter entries). Generated from the engine.

<a id="area_of_effect_at_self"></a>
## area_of_effect_at_self

Executes specified action on all actors within elliptical area centered at caller's position.

```yaml
Action:
  exampleAction:
    do: area_of_effect_at_self
    parameters: [p_action, p_radius]
Parameter:
  p_action:
    # KeyRef to Action entity to execute on each affected actor (required)
    key: action
    value:  # Action
  p_radius:
    # Radius in isometric pixels: hits every actor whose centre is within it, r px sideways and r/2 px up or down on screen (required)
    key: radius
    value:  # Dice
```

<a id="area_of_effect_at_self_radial"></a>
## area_of_effect_at_self_radial

Executes action on all actors in elliptical area at radial position from caller.

```yaml
Action:
  exampleAction:
    do: area_of_effect_at_self_radial
    parameters: [p_action, p_distance, p_radial, p_radius]
Parameter:
  p_action:
    # KeyRef to Action entity to execute on each affected actor (required)
    key: action
    value:  # Action
  p_distance:
    # Distance in isometric pixels from caller's position to AoE center (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to caller's bearing (required)
    key: radial
    value:  # Dice
  p_radius:
    # Radius in isometric pixels: hits every actor whose centre is within it, r px sideways and r/2 px up or down on screen (required)
    key: radius
    value:  # Dice
```

<a id="area_of_effect_at_target"></a>
## area_of_effect_at_target

Executes specified action on all actors within elliptical area centered at target's position.

```yaml
Action:
  exampleAction:
    do: area_of_effect_at_target
    parameters: [p_action, p_radius]
Parameter:
  p_action:
    # KeyRef to Action entity to execute on each affected actor (required)
    key: action
    value:  # Action
  p_radius:
    # Radius in isometric pixels: hits every actor whose centre is within it, r px sideways and r/2 px up or down on screen (required)
    key: radius
    value:  # Dice
```

<a id="area_of_effect_at_target_radial"></a>
## area_of_effect_at_target_radial

Executes action on all actors in elliptical area at radial position from target.

```yaml
Action:
  exampleAction:
    do: area_of_effect_at_target_radial
    parameters: [p_action, p_distance, p_radial, p_radius]
Parameter:
  p_action:
    # KeyRef to Action entity to execute on each affected actor (required)
    key: action
    value:  # Action
  p_distance:
    # Distance in isometric pixels from target's position to AoE center (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to caller's bearing (required)
    key: radial
    value:  # Dice
  p_radius:
    # Radius in isometric pixels: hits every actor whose centre is within it, r px sideways and r/2 px up or down on screen (required)
    key: radius
    value:  # Dice
```

<a id="attach_particle_self"></a>
## attach_particle_self

Attaches a cosmetic particle to the caller.

```yaml
Action:
  exampleAction:
    do: attach_particle_self
    parameters: [p_duration, p_particle]
Parameter:
  p_duration:
    # seconds before auto-remove (optional; 0/absent = until removed)
    key: duration
    value:  # Number
  p_particle:
    # KeyRef to a Particle entity (required)
    key: particle
    value:  # Particle
```

<a id="attach_particle_target"></a>
## attach_particle_target

Attaches a cosmetic particle to the target (e.g. a debuff cloud on an enemy).

```yaml
Action:
  exampleAction:
    do: attach_particle_target
    parameters: [p_duration, p_particle]
Parameter:
  p_duration:
    # number (optional)
    key: duration
    value:  # Number
  p_particle:
    # keyref -> Particle
    key: particle
    value:  # Particle
```

<a id="change_actor_self"></a>
## change_actor_self

Transforms the calling actor into a different actor type while preserving all resources, location, and identity; speed, perception and salience come from the new actor. Useful for polymorph, shapeshift, level up, or evolution mechanics. A map change (change_map_*) in flight for the same actor merges with it: the actor arrives once, in the latest form, at the latest destination.

```yaml
Action:
  exampleAction:
    do: change_actor_self
    parameters: [p_actor]
Parameter:
  p_actor:
    # KeyRef to Actor entity to transform into (required)
    key: actor
    value:  # Actor
```

<a id="change_actor_target"></a>
## change_actor_target

Transforms the target actor into a different actor type while preserving all resources, location, and identity; speed, perception and salience come from the new actor. Useful for polymorph effects cast on other actors. A map change (change_map_*) in flight for the same actor merges with it: the actor arrives once, in the latest form, at the latest destination.

```yaml
Action:
  exampleAction:
    do: change_actor_target
    parameters: [p_actor]
Parameter:
  p_actor:
    # KeyRef to Actor entity to transform target into (required)
    key: actor
    value:  # Actor
```

<a id="change_map_self"></a>
## change_map_self

map: KeyRef to Map. location: KeyRef to Vector where the target actor's new position will be. Moves the actor (a player) to another map. A form change (change_actor_*) made while it is on its way merges with it, and so does one already in flight: the actor arrives once, in the latest form, at the latest destination.

```yaml
Action:
  exampleAction:
    do: change_map_self
    parameters: [p_location, p_map]
Parameter:
  p_location:
    # keyref -> Vector (optional)
    key: location
    value:  # Vector
  p_map:
    # keyref -> Map
    key: map
    value:  # Map
```

<a id="change_map_target"></a>
## change_map_target

map: KeyRef to Map. location: KeyRef to Vector where the target actor's new position will be. Moves the actor (a player) to another map. A form change (change_actor_*) made while it is on its way merges with it, and so does one already in flight: the actor arrives once, in the latest form, at the latest destination.

```yaml
Action:
  exampleAction:
    do: change_map_target
    parameters: [p_location, p_map]
Parameter:
  p_location:
    # keyref -> Vector (optional)
    key: location
    value:  # Vector
  p_map:
    # keyref -> Map
    key: map
    value:  # Map
```

<a id="change_strategy"></a>
## change_strategy

Interrupts current strategy and switches caller to new AI strategy.

```yaml
Action:
  exampleAction:
    do: change_strategy
    parameters: [p_strategy]
Parameter:
  p_strategy:
    # KeyRef to Strategy entity (required)
    key: strategy
    value:  # Strategy
```

<a id="clear_overlays"></a>
## clear_overlays

No parameters required Hides every foreground effect.

```yaml
Action:
  exampleAction:
    do: clear_overlays
    parameters: []
```

<a id="clear_target_self"></a>
## clear_target_self

No parameters required Clears caller's current target, setting it to empty string.

```yaml
Action:
  exampleAction:
    do: clear_target_self
    parameters: []
```

<a id="close_game"></a>
## close_game

No parameters required Closes and exits the game. Currently a placeholder.

```yaml
Action:
  exampleAction:
    do: close_game
    parameters: []
```

<a id="close_interaction"></a>
## close_interaction

No parameters. Closes the caller's interaction panel (e.g. a "Leave" option); the target's, when the caller is not a player.

```yaml
Action:
  exampleAction:
    do: close_interaction
    parameters: []
```

<a id="despawn_self"></a>
## despawn_self

No parameters required Removes calling actor from the game world permanently.

```yaml
Action:
  exampleAction:
    do: despawn_self
    parameters: []
```

<a id="despawn_target"></a>
## despawn_target

No parameters required Removes target actor from the game world permanently.

```yaml
Action:
  exampleAction:
    do: despawn_target
    parameters: []
```

<a id="each_minion"></a>
## each_minion

Runs the action once for each of the caller's minions, with the caller (the master) as caller and the minion as target. Another master's minions are never touched.

```yaml
Action:
  exampleAction:
    do: each_minion
    parameters: [p_action]
Parameter:
  p_action:
    # KeyRef to Action entity to run on each minion (required)
    key: action
    value:  # Action
```

<a id="echo"></a>
## echo

Logs a message. Useful for debugging and displaying information.

```yaml
Action:
  exampleAction:
    do: echo
    parameters: [p_message]
Parameter:
  p_message:
    # Text message to log (required)
    key: message
    value:  # String
```

<a id="echo_self"></a>
## echo_self

Pops a short line of text up out of the caller, like floating combat text. Lines stack. Use sparingly.

```yaml
Action:
  exampleAction:
    do: echo_self
    parameters: [p_audience, p_color, p_duration, p_message]
Parameter:
  p_audience:
    # "public" (every player who can see the caller) or "private" (only the caller's own player) (optional; default public)
    key: audience
    value:  # public | private
  p_color:
    # Text colour, a hex code (e.g. "#ffd040") or a colour name (optional; default white)
    key: color
    value:  # Color
  p_duration:
    # Seconds the text shows, 0.2 to 30 (optional; default 1.6, the same as combat text)
    key: duration
    value:  # Number
  p_message:
    # Text to float up out of the caller; {{$x}} / {{@x}} are resolved (required)
    key: message
    value:  # String
```

<a id="echo_target"></a>
## echo_target

Pops a short line of text up out of the target, like floating combat text. Lines stack. Use sparingly.

```yaml
Action:
  exampleAction:
    do: echo_target
    parameters: [p_audience, p_color, p_duration, p_message]
Parameter:
  p_audience:
    # "public" (every player who can see the target) or "private" (only the caller's own player) (optional; default public)
    key: audience
    value:  # public | private
  p_color:
    # Text colour, a hex code (e.g. "#ffd040") or a colour name (optional; default white)
    key: color
    value:  # Color
  p_duration:
    # Seconds the text shows, 0.2 to 30 (optional; default 1.6, the same as combat text)
    key: duration
    value:  # Number
  p_message:
    # Text to float up out of the target; {{$x}} / {{@x}} are resolved (required)
    key: message
    value:  # String
```

<a id="fly_self"></a>
## fly_self

Makes the caller fly: it ignores walls and the navmesh (moves straight to its destination) until landed.

```yaml
Action:
  exampleAction:
    do: fly_self
    parameters: []
```

<a id="fly_target"></a>
## fly_target

Makes the target fly (see fly_self).

```yaml
Action:
  exampleAction:
    do: fly_target
    parameters: []
```

<a id="follow_master"></a>
## follow_master

Moves the caller to its slot in its master's formation, turned with the way the master faces: its direction of travel while it moves, or a bearing it turns to while it stands. A minion's slot is its place among its master's minions, in the order they were spawned, so a gap closes when one goes. It walks at its master's speed near its slot and faster the further it is from it, to catch up (this sets the caller's speed). Run it often, e.g. as a Strategy behaviour's action: a new path is only planned when the slot has moved a little way, or the minion stands off its slot.

```yaml
Action:
  exampleAction:
    do: follow_master
    parameters: [p_formation, p_spacing]
Parameter:
  p_formation:
    # The formation's shape: column (rows of two behind the master), wedge (a V opening behind), ring (evenly round the master) or line (abreast, one rank behind) (required)
    key: formation
    value:  # column | wedge | ring | line
  p_spacing:
    # The gap between neighbouring minions, and between the master and the first rank, in isometric pixels (required)
    key: spacing
    value:  # Dice
```

<a id="hide_overlay"></a>
## hide_overlay

Hides a single foreground effect.

```yaml
Action:
  exampleAction:
    do: hide_overlay
    parameters: [p_overlay]
Parameter:
  p_overlay:
    # Overlay entity key (required)
    key: overlay
    value:  # Overlay
```

<a id="interrupt_self"></a>
## interrupt_self

Immediately stops all in-progress animations/actions on the caller and snaps it to idle.

```yaml
Action:
  exampleAction:
    do: interrupt_self
    parameters: []
```

<a id="interrupt_target"></a>
## interrupt_target

Immediately stops all in-progress animations/actions on the target and snaps it to idle.

```yaml
Action:
  exampleAction:
    do: interrupt_target
    parameters: []
```

<a id="land_self"></a>
## land_self

Makes the caller land, ending the fly_self ignoring-collision movement. Nothing happens while the caller is not over walkable ground (a wall, water, off the map): it stays flying. $can_land tells whether it could land now.

```yaml
Action:
  exampleAction:
    do: land_self
    parameters: []
```

<a id="land_target"></a>
## land_target

Makes the target land (see land_self). Nothing happens while the target is not over walkable ground.

```yaml
Action:
  exampleAction:
    do: land_target
    parameters: []
```

<a id="minus_resource_self"></a>
## minus_resource_self

Subtracts from caller's resource value. Useful for self-damage, costs, debuffs.

```yaml
Action:
  exampleAction:
    do: minus_resource_self
    parameters: [p_expression, p_resource]
Parameter:
  p_expression:
    # Dice algebra expression (e.g., "2d6+3") (required)
    key: expression
    value:  # Dice
  p_resource:
    # KeyRef to Resource entity (required)
    key: resource
    value:  # Resource
```

<a id="minus_resource_target"></a>
## minus_resource_target

Subtracts from target's resource value. Useful for damage, debuffs, resource costs.

```yaml
Action:
  exampleAction:
    do: minus_resource_target
    parameters: [p_expression, p_resource]
Parameter:
  p_expression:
    # Dice algebra expression (e.g., "2d6+3") (required)
    key: expression
    value:  # Dice
  p_resource:
    # KeyRef to Resource entity (required)
    key: resource
    value:  # Resource
```

<a id="move_away_from_target"></a>
## move_away_from_target

Sets caller's destination to position away from target by specified distance.

```yaml
Action:
  exampleAction:
    do: move_away_from_target
    parameters: [p_distance]
Parameter:
  p_distance:
    # How far to move away, in isometric pixels (required)
    key: distance
    value:  # Dice
```

<a id="move_home_self"></a>
## move_home_self

No parameters. Walks the caller back to where it spawned: its Deployment's location, or where a spawn_actor_* action put it (a change of form keeps it). Does nothing while the caller is on another map. See the built-in measure distance_to_home.

```yaml
Action:
  exampleAction:
    do: move_home_self
```

<a id="start_auto_attack"></a>
## start_auto_attack

No parameters. The caller starts auto-attacking the target: it walks into its Actor's auto_attack_range and repeats its auto_attack skill every auto_attack_interval seconds until the target changes, dies, leaves, or the caller moves by hand. Typically an enemy's on_interact (right-click to attack). Needs an Actor.auto_attack.

```yaml
Action:
  exampleAction:
    do: start_auto_attack
```

<a id="stop_auto_attack_self"></a>
## stop_auto_attack_self

No parameters. The caller stops auto-attacking.

```yaml
Action:
  exampleAction:
    do: stop_auto_attack_self
```

<a id="move_to_self_radial"></a>
## move_to_self_radial

Moves caller to a position at specified radial angle and distance from current location.

```yaml
Action:
  exampleAction:
    do: move_to_self_radial
    parameters: [p_distance, p_radial]
Parameter:
  p_distance:
    # Distance in isometric pixels from caller's current position (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to caller's bearing, where 0° is forward (required)
    key: radial
    value:  # Dice
```

<a id="move_to_target"></a>
## move_to_target

Sets caller's destination to target actor's position. No parameters required.

```yaml
Action:
  exampleAction:
    do: move_to_target
    parameters: []
```

<a id="move_to_target_radial"></a>
## move_to_target_radial

Moves caller to a position at specified radial angle and distance from target.

```yaml
Action:
  exampleAction:
    do: move_to_target_radial
    parameters: [p_distance, p_radial]
Parameter:
  p_distance:
    # Distance in isometric pixels from target position (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to caller's bearing, where 0° is forward (required)
    key: radial
    value:  # Dice
```

<a id="open_dialog"></a>
## open_dialog

Shows the dialog's text in the caller's interaction panel (opening it if needed). When the caller is not a player (an NPC or a door's on_touch), it goes to the target's player instead.

```yaml
Action:
  exampleAction:
    do: open_dialog
    parameters: [p_dialog]
Parameter:
  p_dialog:
    # KeyRef to Dialog entity (required)
    key: dialog
    value:  # Dialog
```

<a id="open_menu"></a>
## open_menu

Shows the menu's options in the caller's interaction panel (opening it if needed). Used to branch a dialog tree to a new set of choices. When the caller is not a player, it goes to the target's player.

```yaml
Action:
  exampleAction:
    do: open_menu
    parameters: [p_menu]
Parameter:
  p_menu:
    # KeyRef to Menu entity (required)
    key: menu
    value:  # Menu
```

<a id="open_options"></a>
## open_options

No parameters required Opens the options/settings menu. Currently a placeholder.

```yaml
Action:
  exampleAction:
    do: open_options
    parameters: []
```

<a id="play_keyframe_self"></a>
## play_keyframe_self

Plays specified animation on caller once (one cycle, even for a looping clip), then the caller returns to idle. The caller can't move while it plays.

```yaml
Action:
  exampleAction:
    do: play_keyframe_self
    parameters: [p_animation]
Parameter:
  p_animation:
    # KeyRef to Animation entity (required)
    key: animation
    value:  # Animation
```

<a id="play_keyframe_target"></a>
## play_keyframe_target

Plays specified animation on target once (one cycle, even for a looping clip), then the target returns to idle. The target can't move while it plays.

```yaml
Action:
  exampleAction:
    do: play_keyframe_target
    parameters: [p_animation]
Parameter:
  p_animation:
    # KeyRef to Animation entity (required)
    key: animation
    value:  # Animation
```

<a id="play_sound_self"></a>
## play_sound_self

Plays a Sound once at the caller, with its pitch, volume and range. It plays on if the caller despawns or changes form straight after.

```yaml
Action:
  exampleAction:
    do: play_sound_self
    parameters: [p_audience, p_sound]
Parameter:
  p_audience:
    # "public" (every player who can see the caller) or "private" (only the caller's own player; when the caller isn't a player, the target's player) (optional; default public)
    key: audience
    value:  # public | private
  p_sound:
    # KeyRef to the Sound to play (required)
    key: sound
    value:  # Sound
```

<a id="play_sound_target"></a>
## play_sound_target

Plays a Sound once at the target, with its pitch, volume and range. It plays on if the target despawns or changes form straight after.

```yaml
Action:
  exampleAction:
    do: play_sound_target
    parameters: [p_audience, p_sound]
Parameter:
  p_audience:
    # "public" (every player who can see the target) or "private" (only the caller's own player; when the caller isn't a player, the target's player) (optional; default public)
    key: audience
    value:  # public | private
  p_sound:
    # KeyRef to the Sound to play (required)
    key: sound
    value:  # Sound
```

<a id="plus_resource_self"></a>
## plus_resource_self

Adds to caller's resource value. Useful for self-healing, gaining resources, buffs.

```yaml
Action:
  exampleAction:
    do: plus_resource_self
    parameters: [p_expression, p_resource]
Parameter:
  p_expression:
    # Dice algebra expression (e.g., "2d6+3") (required)
    key: expression
    value:  # Dice
  p_resource:
    # KeyRef to Resource entity (required)
    key: resource
    value:  # Resource
```

<a id="plus_resource_target"></a>
## plus_resource_target

Adds to target's resource value. Useful for healing, buffs, giving resources to allies.

```yaml
Action:
  exampleAction:
    do: plus_resource_target
    parameters: [p_expression, p_resource]
Parameter:
  p_expression:
    # Dice algebra expression (e.g., "2d6+3") (required)
    key: expression
    value:  # Dice
  p_resource:
    # KeyRef to Resource entity (required)
    key: resource
    value:  # Resource
```

<a id="pull_target_closer"></a>
## pull_target_closer

Pulls target closer to caller by specified distance. Direction calculated from target to caller.

```yaml
Action:
  exampleAction:
    do: pull_target_closer
    parameters: [p_distance]
Parameter:
  p_distance:
    # How far to pull the target toward caller, in isometric pixels (required)
    key: distance
    value:  # Dice
```

<a id="push_target_away"></a>
## push_target_away

Pushes target away from caller by specified distance. Direction calculated from caller to target.

```yaml
Action:
  exampleAction:
    do: push_target_away
    parameters: [p_distance]
Parameter:
  p_distance:
    # How far to push the target away from caller, in isometric pixels (required)
    key: distance
    value:  # Dice
```

<a id="remove_particle_self"></a>
## remove_particle_self

```yaml
Action:
  exampleAction:
    do: remove_particle_self
    parameters: [p_particle]
Parameter:
  p_particle:
    # Particle key to remove (optional; absent = clear all)
    key: particle
    value:  # Particle
```

<a id="remove_particle_target"></a>
## remove_particle_target

Removes a previously attached particle from the target.

```yaml
Action:
  exampleAction:
    do: remove_particle_target
    parameters: [p_particle]
Parameter:
  p_particle:
    # Particle key to remove (optional; absent = clear all)
    key: particle
    value:  # Particle
```

<a id="set_destination_self"></a>
## set_destination_self

Sets caller's movement destination to specified vertex coordinates.

```yaml
Action:
  exampleAction:
    do: set_destination_self
    parameters: [p_destination]
Parameter:
  p_destination:
    # KeyRef to Vector entity (required)
    key: destination
    value:  # Vector
```

<a id="set_modulate_self"></a>
## set_modulate_self

Sets caller's color modulation/tint. Useful for visual status effects, team colors, highlighting.

```yaml
Action:
  exampleAction:
    do: set_modulate_self
    parameters: [p_color]
Parameter:
  p_color:
    # Hex color code (e.g., "#ff0000" for red) or color name (required)
    key: color
    value:  # Color
```

<a id="set_modulate_target"></a>
## set_modulate_target

Sets target's color modulation/tint. Useful for visual status effects, team colors, highlighting.

```yaml
Action:
  exampleAction:
    do: set_modulate_target
    parameters: [p_color]
Parameter:
  p_color:
    # Hex color code (e.g., "#ff0000" for red) or color name (required)
    key: color
    value:  # Color
```

<a id="set_overlay_opacity"></a>
## set_overlay_opacity

Sets the opacity of a shown foreground effect.

```yaml
Action:
  exampleAction:
    do: set_overlay_opacity
    parameters: [p_opacity, p_overlay]
Parameter:
  p_opacity:
    # Opacity 0..1 (required)
    key: opacity
    value:  # Number
  p_overlay:
    # Overlay entity key (required)
    key: overlay
    value:  # Overlay
```

<a id="set_resource_self"></a>
## set_resource_self

Sets caller's resource to exact value (still respects min/max bounds).

```yaml
Action:
  exampleAction:
    do: set_resource_self
    parameters: [p_resource, p_value]
Parameter:
  p_resource:
    # KeyRef to Resource entity (required)
    key: resource
    value:  # Resource
  p_value:
    # Absolute value to set (required)
    key: value
    value:  # Number
```

<a id="set_resource_target"></a>
## set_resource_target

Sets target's resource to exact value (still respects min/max bounds).

```yaml
Action:
  exampleAction:
    do: set_resource_target
    parameters: [p_resource, p_value]
Parameter:
  p_resource:
    # KeyRef to Resource entity (required)
    key: resource
    value:  # Resource
  p_value:
    # Absolute value to set (required)
    key: value
    value:  # Number
```

<a id="set_skill_self"></a>
## set_skill_self

Assigns a skill into one of the caller's hotbar slots at runtime. The swap is kept in the player's save and through change_actor, and one made on the actor's first frame stays on the hotbar.

```yaml
Action:
  exampleAction:
    do: set_skill_self
    parameters: [p_skill, p_slot]
Parameter:
  p_skill:
    # KeyRef to a Skill entity (required)
    key: skill
    value:  # Skill
  p_slot:
    # Hotbar slot to assign, 1-9 (required)
    key: slot
    value:  # Number
```

<a id="set_skill_target"></a>
## set_skill_target

Assigns a skill into one of the target's hotbar slots at runtime. The swap is kept in the player's save and through change_actor.

```yaml
Action:
  exampleAction:
    do: set_skill_target
    parameters: [p_skill, p_slot]
Parameter:
  p_skill:
    # KeyRef to a Skill entity (required)
    key: skill
    value:  # Skill
  p_slot:
    # Hotbar slot to assign, 1-9 (required)
    key: slot
    value:  # Number
```

<a id="set_speed_self"></a>
## set_speed_self

Permanently sets caller's movement speed.

```yaml
Action:
  exampleAction:
    do: set_speed_self
    parameters: [p_speed]
Parameter:
  p_speed:
    # New speed multiplier; 1.0 is normal pace (required)
    key: speed
    value:  # Number
```

<a id="set_speed_target"></a>
## set_speed_target

Permanently sets target's movement speed.

```yaml
Action:
  exampleAction:
    do: set_speed_target
    parameters: [p_speed]
Parameter:
  p_speed:
    # New speed multiplier; 1.0 is normal pace (required)
    key: speed
    value:  # Number
```

<a id="set_target_target_to_self"></a>
## set_target_target_to_self

No parameters. The target now targets the caller (a taunt: "look at me").

```yaml
Action:
  exampleAction:
    do: set_target_target_to_self
    parameters: []
```

<a id="set_target_target_to_self_target"></a>
## set_target_target_to_self_target

No parameters. The target takes the caller's current target (an assist or rally: inside an area-of-effect sub-action, a pack leader passes its target to each helper). Nothing changes when the caller has no target, or when its target is the target itself.

```yaml
Action:
  exampleAction:
    do: set_target_target_to_self_target
    parameters: []
```

<a id="show_connection_info"></a>
## show_connection_info

No parameters required Displays connection and network information. Currently a placeholder.

```yaml
Action:
  exampleAction:
    do: show_connection_info
    parameters: []
```

<a id="show_overlay"></a>
## show_overlay

Shows a full-screen foreground effect (rain/fire/fog) between the world and the UI.

```yaml
Action:
  exampleAction:
    do: show_overlay
    parameters: [p_overlay]
Parameter:
  p_overlay:
    # Overlay entity key (required)
    key: overlay
    value:  # Overlay
```

<a id="spawn_actor_at_self"></a>
## spawn_actor_at_self

Spawns a new actor instance at caller's current position. The caller becomes the new actor's master (see target_master, follow_master and the minions value).

```yaml
Action:
  exampleAction:
    do: spawn_actor_at_self
    parameters: [p_actor]
Parameter:
  p_actor:
    # KeyRef to Actor entity (required)
    key: actor
    value:  # Actor
```

<a id="spawn_actor_at_self_radial"></a>
## spawn_actor_at_self_radial

Spawns a new actor instance at radial position from caller. The caller becomes the new actor's master (see target_master, follow_master and the minions value).

```yaml
Action:
  exampleAction:
    do: spawn_actor_at_self_radial
    parameters: [p_actor, p_distance, p_radial]
Parameter:
  p_actor:
    # KeyRef to Actor entity (required)
    key: actor
    value:  # Actor
  p_distance:
    # Distance in isometric pixels from caller's position (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to caller's bearing (required)
    key: radial
    value:  # Dice
```

<a id="spawn_actor_at_target"></a>
## spawn_actor_at_target

Spawns a new actor instance at target's current position. The caller becomes the new actor's master (see target_master, follow_master and the minions value).

```yaml
Action:
  exampleAction:
    do: spawn_actor_at_target
    parameters: [p_actor]
Parameter:
  p_actor:
    # KeyRef to Actor entity (required)
    key: actor
    value:  # Actor
```

<a id="spawn_actor_at_target_radial"></a>
## spawn_actor_at_target_radial

Spawns a new actor instance at radial position from target. The caller becomes the new actor's master (see target_master, follow_master and the minions value).

```yaml
Action:
  exampleAction:
    do: spawn_actor_at_target_radial
    parameters: [p_actor, p_distance, p_radial]
Parameter:
  p_actor:
    # KeyRef to Actor entity (required)
    key: actor
    value:  # Actor
  p_distance:
    # Distance in isometric pixels from target's position (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to caller's bearing (required)
    key: radial
    value:  # Dice
```

<a id="swap_positions"></a>
## swap_positions

No parameters required Swaps positions of caller and target. Both actors instantly teleport to each other's location.

```yaml
Action:
  exampleAction:
    do: swap_positions
    parameters: []
```

<a id="target_furthest"></a>
## target_furthest

Sets caller's target to the furthest actor in view. No parameters required.

```yaml
Action:
  exampleAction:
    do: target_furthest
    parameters: []
```

<a id="target_highest_measure"></a>
## target_highest_measure

Targets the actor in view with the highest value of specified measure.

```yaml
Action:
  exampleAction:
    do: target_highest_measure
    parameters: [p_measure]
Parameter:
  p_measure:
    # KeyRef to Measure entity (required)
    key: measure
    value:  # Measure
```

<a id="target_highest_resource"></a>
## target_highest_resource

Targets the actor in view with the highest value of specified resource.

```yaml
Action:
  exampleAction:
    do: target_highest_resource
    parameters: [p_resource]
Parameter:
  p_resource:
    # KeyRef to Resource entity (required)
    key: resource
    value:  # Resource
```

<a id="target_lowest_measure"></a>
## target_lowest_measure

Targets the actor in view with the lowest value of specified measure.

```yaml
Action:
  exampleAction:
    do: target_lowest_measure
    parameters: [p_measure]
Parameter:
  p_measure:
    # KeyRef to Measure entity (required)
    key: measure
    value:  # Measure
```

<a id="target_lowest_resource"></a>
## target_lowest_resource

Targets the actor in view with the lowest value of specified resource.

```yaml
Action:
  exampleAction:
    do: target_lowest_resource
    parameters: [p_resource]
Parameter:
  p_resource:
    # KeyRef to Resource entity (required)
    key: resource
    value:  # Resource
```

<a id="target_master"></a>
## target_master

No parameters. The caller targets its master: the actor whose spawn_actor_* action spawned it. Nothing changes when the caller has no master, or its master is gone.

```yaml
Action:
  exampleAction:
    do: target_master
    parameters: []
```

<a id="target_master_target"></a>
## target_master_target

No parameters. The caller targets whatever its master is targeting ("attack what my master attacks"). Nothing changes when the caller has no master, the master has no target, or the master is targeting the caller.

```yaml
Action:
  exampleAction:
    do: target_master_target
    parameters: []
```

<a id="target_nearest"></a>
## target_nearest

Sets caller's target to the nearest actor in view. No parameters required.

```yaml
Action:
  exampleAction:
    do: target_nearest
    parameters: []
```

<a id="target_nearest_in_group"></a>
## target_nearest_in_group

Targets the nearest actor in view that belongs to the specified group, measured in isometric pixels like distance_to_target.

```yaml
Action:
  exampleAction:
    do: target_nearest_in_group
    parameters: [p_group]
Parameter:
  p_group:
    # KeyRef to Group entity (required)
    key: group
    value:  # Group
```

<a id="target_random"></a>
## target_random

Sets caller's target to a random actor in view. No parameters required.

```yaml
Action:
  exampleAction:
    do: target_random
    parameters: []
```

<a id="target_random_in_group"></a>
## target_random_in_group

Targets a random actor in view that belongs to specified group.

```yaml
Action:
  exampleAction:
    do: target_random_in_group
    parameters: [p_group]
Parameter:
  p_group:
    # KeyRef to Group entity (required)
    key: group
    value:  # Group
```

<a id="teleport"></a>
## teleport

Teleports caller forward in current bearing direction by specified distance.

```yaml
Action:
  exampleAction:
    do: teleport
    parameters: [p_distance]
Parameter:
  p_distance:
    # Distance in isometric pixels to teleport forward (required)
    key: distance
    value:  # Dice
```

<a id="teleport_self_to_target"></a>
## teleport_self_to_target

No parameters required Instantly teleports caller to target's current position.

```yaml
Action:
  exampleAction:
    do: teleport_self_to_target
    parameters: []
```

<a id="teleport_self_to_target_radial"></a>
## teleport_self_to_target_radial

Teleports caller to position at specified radial angle and distance from target.

```yaml
Action:
  exampleAction:
    do: teleport_self_to_target_radial
    parameters: [p_distance, p_radial]
Parameter:
  p_distance:
    # Distance in isometric pixels from target position (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to caller's bearing, where 0° is forward (required)
    key: radial
    value:  # Dice
```

<a id="teleport_self_to_vector"></a>
## teleport_self_to_vector

Instantly teleports caller to a fixed position on its current map (e.g. a respawn point).

```yaml
Action:
  exampleAction:
    do: teleport_self_to_vector
    parameters: [p_location]
Parameter:
  p_location:
    # KeyRef to Vector entity (required)
    key: location
    value:  # Vector
```

<a id="teleport_target_to_self"></a>
## teleport_target_to_self

No parameters required Instantly teleports target to caller's current position.

```yaml
Action:
  exampleAction:
    do: teleport_target_to_self
    parameters: []
```

<a id="teleport_target_to_self_radial"></a>
## teleport_target_to_self_radial

Teleports target to position at specified radial angle and distance from caller.

```yaml
Action:
  exampleAction:
    do: teleport_target_to_self_radial
    parameters: [p_distance, p_radial]
Parameter:
  p_distance:
    # Distance in isometric pixels from caller's position (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to target's bearing, where 0° is forward (required)
    key: radial
    value:  # Dice
```

<a id="teleport_target_to_vector"></a>
## teleport_target_to_vector

Instantly teleports target to a fixed position on its current map (e.g. a respawn point).

```yaml
Action:
  exampleAction:
    do: teleport_target_to_vector
    parameters: [p_location]
Parameter:
  p_location:
    # KeyRef to Vector entity (required)
    key: location
    value:  # Vector
```

<a id="teleport_to_radial"></a>
## teleport_to_radial

Teleports caller to position at specified radial angle and distance from current location.

```yaml
Action:
  exampleAction:
    do: teleport_to_radial
    parameters: [p_distance, p_radial]
Parameter:
  p_distance:
    # Distance in isometric pixels from caller's current position (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to caller's bearing, where 0° is forward (required)
    key: radial
    value:  # Dice
```

<a id="temp_speed_self"></a>
## temp_speed_self

Temporarily changes caller's speed, then restores original speed after duration.

```yaml
Action:
  exampleAction:
    do: temp_speed_self
    parameters: [p_duration, p_speed]
Parameter:
  p_duration:
    # Duration in seconds for the speed change (required)
    key: duration
    value:  # Number
  p_speed:
    # Temporary speed multiplier; 1.0 is normal pace (required)
    key: speed
    value:  # Number
```

<a id="temp_speed_target"></a>
## temp_speed_target

Temporarily changes target's speed, then restores original speed after duration.

```yaml
Action:
  exampleAction:
    do: temp_speed_target
    parameters: [p_duration, p_speed]
Parameter:
  p_duration:
    # Duration in seconds for the speed change (required)
    key: duration
    value:  # Number
  p_speed:
    # Temporary speed multiplier; 1.0 is normal pace (required)
    key: speed
    value:  # Number
```

<a id="transfer_resource"></a>
## transfer_resource

Subtracts from target's resource and adds same amount to caller's resource. Example: Drain 10 health from enemy and heal self for same amount.

```yaml
Action:
  exampleAction:
    do: transfer_resource
    parameters: [p_expression, p_resource]
Parameter:
  p_expression:
    # Dice algebra for amount to transfer (required)
    key: expression
    value:  # Dice
  p_resource:
    # KeyRef to Resource entity (required)
    key: resource
    value:  # Resource
```

<a id="use_track"></a>
## use_track

Index-numbered params - Vector entity keys keyed by integer index, e.g. "0": pointA, "2": pointB (at least one; indices need not be contiguous). Points are ordered by their integer key. Walks the caller round the points in order, back to the first after the last. Each call sends the caller to its current point, and on to the next once it has stopped there (within 8 px, or at the nearest spot it can reach), so call it repeatedly: from a Strategy behavior, or a Timer about once a second. Calling it again with the same track carries on from where it is; a different track starts at its own first point.

```yaml
Action:
  exampleAction:
    do: use_track
    # one or more index-numbered Vector keys, in order
    parameters: [point0, point1]
Parameter:
  point0:
    key: "0"
    value:  # Vector
  point1:
    key: "1"
    value:  # Vector
```

<a id="wait"></a>
## wait

No parameters required. The wait is the action's `time`, in seconds. Runs this action's `then`, and so the rest of its chain, `time` seconds later. Nothing else waits: the caller can still move and act, and other actions, Triggers, Timers and Strategies go on (a Timer or Strategy that fires again meanwhile starts another chain). With `time: 0` the `then` runs at once. An `if` that fails runs the `else` at once, without waiting. The rest of the chain is dropped, quietly, if by then the caller or the target has despawned or moved to another map; a caller that changed form in place carries on in its new form. On a skill's start action, `time` also locks the caster for those seconds, as it does for any start action.

```yaml
Action:
  exampleAction:
    do: wait
    parameters: []
```

{% endraw %}
