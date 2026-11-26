# Distances and sizes

Every distance on the ground is measured in **isometric pixels**. The distance between two points is
`sqrt(dx² + (2·dy)²)`, where `dx` and `dy` are the screen offsets in world pixels. A step straight up or down the
screen counts double, so a distance `r` reaches `r` pixels to either side and `r/2` pixels up or down: the same
ellipse the ground is drawn on. Sight, touch, `distance_to_target`, area-of-effect radii and every movement distance
use this one measure, between actor centres.

| Field or parameter | Unit | Notes |
|---|---|---|
| `Actor.perception` | isometric px | Radius of what an actor sees. |
| `Actor.salience` | isometric px | Radius within which an actor can be seen. |
| `Actor.base` | isometric px | Radius of the actor's footprint on the ground. |
| Touch and interaction reach | isometric px | Centre to centre, within the two bodies' radii plus a small margin. |
| `distance_to_target`, `distance_to_destination`, `distance_to_home` | isometric px | Measures to compare in conditions, e.g. for a leash. |
| Track and destination arrival | isometric px | How close counts as arrived. |
| `area_of_effect_*` `radius` | isometric px | Hits every actor whose centre is within the radius. |
| `area_of_effect_*_radial` `distance` | isometric px | How far from the caller or target the area's centre sits. |
| `push_target_away`, `pull_target_closer`, `move_away_from_target` `distance` | isometric px | Along the line between the two actors. |
| `move_to_*_radial`, `teleport`, `teleport_*_radial`, `teleport_to_radial`, `spawn_actor_at_*_radial` `distance` | isometric px | At the given angle on the ground. |
| `radial` | degrees | An angle on the ground, not a distance. |
| `speed`, `set_speed_*`, `temp_speed_*` | multiplier | 1.0 is normal speed. |
| `Vector.x`, `Vector.y` | world px | Positions, not distances. |
| `Sprite.size`, `Sprite.margin`, `Overlay.size`, `Particle.size` | texture px | Sizes of images, not ground distances. |
| `Particle.speed` | screen px per second | A visual drift, not a ground distance. |

## Changes in engine 0.401 (ticket 210)

Area-of-effect radii and movement distances used to be measured differently, and now match the rest of the engine.

- **Area of effect.** A radius `r` used to reach about `2r` plus the target's footprint. It now reaches exactly `r`.
  To keep an old area the same size, use `r_new = 2 × r_old + base`, where `base` is the footprint radius of the
  actors it should reach.
- **Pushes, pulls and radial moves.** A distance used to be straight-line screen pixels, so moving up or down the
  screen covered twice the ground of moving sideways. Sideways distances are unchanged. A straight up or down move
  now covers half the screen distance it used to. To keep an old vertical move, double it; diagonals sit in between.
