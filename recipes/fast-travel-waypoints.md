# Fast travel with waypoints

**Problem.** Once a player has found the shrine on the moor, they should be able to travel back to it from
anywhere.

**Pattern.** A [`Waypoint`](/reference/entities.html#waypoint) marks a named place on a map. It appears on the
player's map once **discovered**: the first time the player opens their map while within three quarters of their
`perception` of it. Give the waypoint a `menu`; choosing the waypoint on the map opens it with the player as both
caller and target, so a "Travel here" option uses `*_self` actions.

```yaml
Resource:
  zoneResource:
    name: Zone
    default: 1
    min: '0'
    max: '99'
    reveal: 999999

Parameter:
  zoneParameter:
    key: resource
    value: zoneResource
  moorZoneValueParameter:
    key: value
    value: '2'
  moorMapParameter:
    key: map
    value: moorMap
  shrineLocationParameter:
    key: location
    value: shrineVector

Vector:
  townSpawnVector:
    x: 384
    y: 8
  shrineVector:
    x: 300
    y: 40

Action:
  travelToShrineAction:
    name: Travel here
    do: close_interaction
    then: travelToShrineZoneAction
  travelToShrineZoneAction:
    do: set_resource_self
    parameters: [zoneParameter, moorZoneValueParameter]
    then: travelToShrineMapAction
  travelToShrineMapAction:
    do: change_map_self
    parameters: [moorMapParameter, shrineLocationParameter]

Menu:
  shrineMenu:
    name: The Standing Shrine
    actions: [travelToShrineAction]

Waypoint:
  shrineWaypoint:
    name: The Standing Shrine
    description: Seven stones on the moor road.
    map: moorMap
    location: shrineVector
    menu: shrineMenu

Map:
  townMap:
    name: Town
    tilemap: groundTilemap
    spawn: townSpawnVector
  moorMap:
    name: The Moor
    tilemap: groundTilemap
    spawn: shrineVector
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Map`](/reference/entities.html#map), [`Menu`](/reference/entities.html#menu), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Vector`](/reference/entities.html#vector), [`Waypoint`](/reference/entities.html#waypoint)

**Gotchas.**

- Discovery is saved with the player, so a found waypoint stays found.
- `change_map_self` to the map the player is already on does nothing. For a waypoint on the same map, use
  `teleport_self_to_vector` instead, or offer both options with an `if` on a zone resource.
- The waypoint's `location` must be on walkable ground, since the player lands there.
- A waypoint without a `menu` is only a marker.
- To charge for travel, put an `if` on gold in front of the chain, as in
  [A vendor with a gold check](/recipes/vendor-with-gold-check.html).
