# Items as resources: equip, use, drop

**Problem.** You want an inventory: swords to equip (one weapon at a time), potions to drink, and anything can be
dropped.

**Pattern.** Every item is its own [`Resource`](/reference/entities.html#resource) holding how many you own, with `reveal: 1` so it appears in the
inventory only while you have one. Gear also gets an `..._equipped` resource (0 or 1). A Resource's `menu` gives it
options in the inventory. Each option first closes the panel (`close_interaction`), then does the work:

- **Equip:** clear every other item in the same slot, then set this one's flag.
- **Use:** take one away, then apply the effect.
- **Drop:** take one away, and clear the equipped flag if none are left.

```yaml
Resource:
  healthResource:
    name: Health
    default: 100
    min: '0'
    max: '100'
  ironSwordResource:
    name: Iron Sword
    description: Plain, heavy, honest.
    default: 0
    min: '0'
    max: '1'
    reveal: 1
    menu: ironSwordMenu
  ironSwordEquippedResource:
    name: Iron Sword (equipped)
    default: 0
    min: '0'
    max: '1'
    reveal: 1
    menu: ironSwordEquippedMenu
  oakStaffEquippedResource:
    name: Oak Staff (equipped)
    default: 0
    min: '0'
    max: '1'
    reveal: 1
  healingDraughtResource:
    name: Healing Draught
    description: Restores 30 health.
    default: 0
    min: '0'
    max: '5'
    reveal: 1
    menu: healingDraughtMenu

Condition:
  ownsIronSwordCondition:
    left: '$ironSwordResource'
    operator: '>='
    right: '1'
  noIronSwordCondition:
    left: '$ironSwordResource'
    operator: '=='
    right: '0'
  ownsDraughtCondition:
    left: '$healingDraughtResource'
    operator: '>='
    right: '1'

Parameter:
  healthParameter:
    key: resource
    value: healthResource
  ironSwordParameter:
    key: resource
    value: ironSwordResource
  ironSwordEquippedParameter:
    key: resource
    value: ironSwordEquippedResource
  oakStaffEquippedParameter:
    key: resource
    value: oakStaffEquippedResource
  healingDraughtParameter:
    key: resource
    value: healingDraughtResource
  oneValueParameter:
    key: value
    value: '1'
  zeroValueParameter:
    key: value
    value: '0'
  oneExpressionParameter:
    key: expression
    value: '1'
  draughtHealParameter:
    key: expression
    value: '30'

Action:
  # Equip: one weapon at a time.
  equipIronSwordAction:
    name: Equip
    do: close_interaction
    then: equipIronSwordRunAction
  equipIronSwordRunAction:
    if: ownsIronSwordCondition
    do: set_resource_self
    parameters: [oakStaffEquippedParameter, zeroValueParameter]
    then: equipIronSwordSetAction
  equipIronSwordSetAction:
    do: set_resource_self
    parameters: [ironSwordEquippedParameter, oneValueParameter]
  unequipIronSwordAction:
    name: Unequip
    do: close_interaction
    then: unequipIronSwordRunAction
  unequipIronSwordRunAction:
    do: set_resource_self
    parameters: [ironSwordEquippedParameter, zeroValueParameter]
  # Drop: lose one; if none are left, it can't stay equipped.
  dropIronSwordAction:
    name: Drop
    do: close_interaction
    then: dropIronSwordRunAction
  dropIronSwordRunAction:
    if: ownsIronSwordCondition
    do: minus_resource_self
    parameters: [ironSwordParameter, oneExpressionParameter]
    then: dropIronSwordCleanupAction
  dropIronSwordCleanupAction:
    if: noIronSwordCondition
    do: set_resource_self
    parameters: [ironSwordEquippedParameter, zeroValueParameter]
  # Use a consumable.
  drinkDraughtAction:
    name: Drink
    do: close_interaction
    then: drinkDraughtRunAction
  drinkDraughtRunAction:
    if: ownsDraughtCondition
    do: minus_resource_self
    parameters: [healingDraughtParameter, oneExpressionParameter]
    then: drinkDraughtHealAction
  drinkDraughtHealAction:
    do: plus_resource_self
    parameters: [healthParameter, draughtHealParameter]

Menu:
  ironSwordMenu:
    name: Iron Sword
    actions: [equipIronSwordAction, dropIronSwordAction]
  ironSwordEquippedMenu:
    name: Iron Sword
    actions: [unequipIronSwordAction]
  healingDraughtMenu:
    name: Healing Draught
    actions: [drinkDraughtAction]
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Condition`](/reference/entities.html#condition), [`Menu`](/reference/entities.html#menu), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource)

**Gotchas.**

- An inventory menu runs with the player as both caller and target, so `*_self` actions act on the player.
- The engine does not refresh a menu after an option runs. Close it first, or the old options stay on screen.
- Always re-check ownership (`if`) in the run step. A stale menu or a double click must not equip or drink
  something you no longer have.
- With several items per slot, the equip step clears every other item in that slot. A generator script keeps this
  tidy.
- If gear raises **maximum** health (health `max: '$maxHealthMeasure'`), taking it off doesn't lower the current
  value by itself. End each equip, unequip and drop chain with `minus_resource_self` on health with expression
  `'0'`: any change re-applies the bounds, so health drops to the new maximum.
- Stats follow the flags through [Measures](/recipes/derived-stats-with-measures.html).
