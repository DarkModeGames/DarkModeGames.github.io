# Travel between maps through a gate

**Problem.** A gate should take the player to another map, but only after asking "Travel or stay?", whether they
walk into it or right-click it. A locked gate should refuse until the player has earned the way.

**Pattern.** The gate is a targetable actor with its own `dialog` and a two-option `menu`. There are two ways in,
and they differ in who the caller is:

- **Right-click.** The player walks up and the gate's own menu opens. The **player** is the caller and the gate
  is the target, so this menu's Travel uses `*_self` actions and conditions on `$`.
- **Walking into it.** `on_touch` runs with the **gate** as caller and the player as target. Its action opens the
  same dialog plus a second menu (`open_dialog`, then `open_menu`). Those panels go to the target's player, because
  the caller is not a player. The options in it still run with the gate as caller, so this menu's Travel uses
  `*_target` actions and conditions on `@`.

Travel closes the panel, records the zone (no condition can read which map you are on), and changes map.

```yaml
Resource:
  zoneResource:
    name: Zone
    default: 1
    min: '0'
    max: '99'
    reveal: 999999
  moorPassResource:
    name: Moor Pass
    default: 0
    min: '0'
    max: '1'
    reveal: 1

Condition:
  # The same rule twice: "the player holds a pass". $ when the player is the caller, @ when the gate is.
  selfHasPassCondition:
    left: '$moorPassResource'
    operator: '=='
    right: '1'
  targetHasPassCondition:
    left: '@moorPassResource'
    operator: '=='
    right: '1'

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
  moorArrivalParameter:
    key: location
    value: moorArrivalVector
  moorGateDialogParameter:
    key: dialog
    value: moorGateDialog
  moorGateRefusedParameter:
    key: dialog
    value: moorGateRefusedDialog
  moorGateTouchMenuParameter:
    key: menu
    value: moorGateTouchMenu

Dialog:
  moorGateDialog:
    title: The Moor Gate
    text: "Beyond the gate the road runs out onto the moor."
  moorGateRefusedDialog:
    title: The Moor Gate
    text: "The gate-warden shakes his head. No pass, no moor."

Action:
  # Walking into the gate: caller = gate, target = player.
  moorGateTouchAction:
    do: open_dialog
    parameters: [moorGateDialogParameter]
    then: moorGateTouchMenuAction
  moorGateTouchMenuAction:
    do: open_menu
    parameters: [moorGateTouchMenuParameter]
  moorGateTouchTravelAction:
    name: Travel
    if: targetHasPassCondition
    do: close_interaction
    then: moorGateTouchZoneAction
    else: moorGateRefusedAction
  moorGateTouchZoneAction:
    do: set_resource_target
    parameters: [zoneParameter, moorZoneValueParameter]
    then: moorGateTouchMapAction
  moorGateTouchMapAction:
    do: change_map_target
    parameters: [moorMapParameter, moorArrivalParameter]
  # Right-click: caller = player, target = gate.
  moorGateTravelAction:
    name: Travel
    if: selfHasPassCondition
    do: close_interaction
    then: moorGateZoneAction
    else: moorGateRefusedAction
  moorGateZoneAction:
    do: set_resource_self
    parameters: [zoneParameter, moorZoneValueParameter]
    then: moorGateMapAction
  moorGateMapAction:
    do: change_map_self
    parameters: [moorMapParameter, moorArrivalParameter]
  # Shared by both menus.
  moorGateStayAction:
    name: Stay
    do: close_interaction
  moorGateRefusedAction:
    do: open_dialog
    parameters: [moorGateRefusedParameter]

Menu:
  moorGateMenu:
    name: Go out onto the moor?
    actions: [moorGateTravelAction, moorGateStayAction]
  moorGateTouchMenu:
    name: Go out onto the moor?
    actions: [moorGateTouchTravelAction, moorGateStayAction]

Actor:
  moorGateActor:
    name: The Moor Gate
    sprite: gateSprite
    base: 14
    salience: 300
    targetable: true
    dialog: moorGateDialog
    menu: moorGateMenu
    on_touch: moorGateTouchAction

Vector:
  townSpawnVector:
    x: 384
    y: 8
  moorGateVector:
    x: 600
    y: 8
  moorArrivalVector:
    x: 220
    y: 8

Deployment:
  moorGateDeployment:
    actor: moorGateActor
    location: moorGateVector

Map:
  townMap:
    name: Town
    tilemap: groundTilemap
    spawn: townSpawnVector
    deployments: [moorGateDeployment]
  moorMap:
    name: The Moor
    tilemap: groundTilemap
    spawn: moorArrivalVector
```

`groundTilemap` stands for your own [`TileMap`](/reference/entities.html#tilemap). For an open gate, drop the
`if` and `else` from both Travel actions. Build the return gate on the moor the same way.

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Deployment`](/reference/entities.html#deployment), [`Dialog`](/reference/entities.html#dialog), [`Map`](/reference/entities.html#map), [`Menu`](/reference/entities.html#menu), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource), [`Vector`](/reference/entities.html#vector)

**Gotchas.**

- **Write every rule twice**, once with `$` (player is the caller) and once with `@` (gate is the caller). A `$`
  condition in the touch menu would read the gate's own resources.
- `on_touch` fires only when the toucher is a player's own character, so monsters never open the gate's panel.
- `open_dialog`, `open_menu` and `close_interaction` called by a non-player go to the **target's** player. A
  private `echo_self` from the gate shows to nobody, because a gate has no player of its own; use a dialog for
  the refusal.
- Put each arrival point a little way from the return gate, on walkable ground, or the player lands inside it and
  the panel opens again.
- `change_map_*` to the map the actor is already on does nothing. Use `teleport_*_to_vector` within a map.
- [Vector](/reference/entities.html#vector) positions are map-local: the same `x`/`y` means the same spot on any map.
