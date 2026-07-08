{% raw %}
# Action Function Reference

All 75 action functions. Each block is a ready-to-copy YAML scaffold wiring the function into an Action (and its Parameter entries). Generated from the engine.

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
    # Radius of the elliptical area in pixels (required)
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
    # Distance in pixels from caller's position to AoE center (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to caller's bearing (required)
    key: radial
    value:  # Dice
  p_radius:
    # Radius of the elliptical area in pixels (required)
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
    # Radius of the elliptical area in pixels (required)
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
    # Distance in pixels from target's position to AoE center (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to caller's bearing (required)
    key: radial
    value:  # Dice
  p_radius:
    # Radius of the elliptical area in pixels (required)
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

Transforms the calling actor into a different actor type while preserving all resources, location, and identity. Useful for polymorph, shapeshift, level up, or evolution mechanics.

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

Transforms the target actor into a different actor type while preserving all resources, location, and identity. Useful for polymorph effects cast on other actors.

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

map: KeyRef to Map. location: KeyRef to Vector where the target actor's new position will be.

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

map: KeyRef to Map. location: KeyRef to Vector where the target actor's new position will be.

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

No parameters. Closes the caller's interaction panel (e.g. a "Leave" option).

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

<a id="fly_self"></a>
## fly_self

```yaml
Action:
  exampleAction:
    do: fly_self
    parameters: []
```

<a id="fly_target"></a>
## fly_target

```yaml
Action:
  exampleAction:
    do: fly_target
    parameters: []
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

<a id="land_self"></a>
## land_self

```yaml
Action:
  exampleAction:
    do: land_self
    parameters: []
```

<a id="land_target"></a>
## land_target

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
    # How far to move away (required)
    key: distance
    value:  # Dice
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
    # Distance in pixels from caller's current position (required)
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
    # Distance in pixels from target position (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to caller's bearing, where 0° is forward (required)
    key: radial
    value:  # Dice
```

<a id="open_dialog"></a>
## open_dialog

Shows the dialog's text in the caller's interaction panel (opening it if needed).

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

Shows the menu's options in the caller's interaction panel (opening it if needed). Used to branch a dialog tree to a new set of choices.

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

Plays specified animation on caller. Sets animation state and triggers playback.

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

Plays specified animation on target. Sets animation state and triggers playback.

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
    # How far to pull the target toward caller (required)
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
    # How far to push the target away from caller (required)
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

```yaml
Action:
  exampleAction:
    do: remove_particle_target
    parameters: [p_particle]
Parameter:
  p_particle:
    # keyref -> Particle (optional)
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
    # New speed value in pixels per second (required)
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
    # New speed value in pixels per second (required)
    key: speed
    value:  # Number
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

Spawns a new actor instance at caller's current position.

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

Spawns a new actor instance at radial position from caller.

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
    # Distance in pixels from caller's position (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to caller's bearing (required)
    key: radial
    value:  # Dice
```

<a id="spawn_actor_at_target"></a>
## spawn_actor_at_target

Spawns a new actor instance at target's current position.

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

Spawns a new actor instance at radial position from target.

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
    # Distance in pixels from target's position (required)
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

Targets nearest actor in view that belongs to specified group.

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
    # Distance in pixels to teleport forward (required)
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
    # Distance in pixels from target position (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to caller's bearing, where 0° is forward (required)
    key: radial
    value:  # Dice
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
    # Distance in pixels from caller's position (required)
    key: distance
    value:  # Dice
  p_radial:
    # Angle in degrees (0-360) relative to target's bearing, where 0° is forward (required)
    key: radial
    value:  # Dice
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
    # Distance in pixels from caller's current position (required)
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
    # Temporary speed value in pixels per second (required)
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
    # Temporary speed value in pixels per second (required)
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

Index-numbered params - Vector entity keys keyed by integer index, e.g. "0": pointA, "2": pointB (at least one; indices need not be contiguous). Points are ordered by their integer key. Executes the resulting sequence of points for AI pathfinding behavior.

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

No parameters required. Duration specified via action.time field. Pauses action chain execution. Useful for timing/sequencing effects.

```yaml
Action:
  exampleAction:
    do: wait
    parameters: []
```

{% endraw %}
