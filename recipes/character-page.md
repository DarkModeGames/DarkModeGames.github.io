# A character page with equipment slots

**Problem.** Players want one page that shows what they wear and their key stats, and lets them swap gear by
picking a slot, like a WoW or Diablo character screen.

**Pattern.** A `Sheet` lays out `Slot`s and `Stat`s on a grid; the actor names it in `sheet` and players open it
with **U**. Items stay ordinary Resources ([Items as resources](/recipes/items-as-resources.html)): a count, an
`<item>_equipped` flag, and a `Menu` with Equip and Unequip. A slot shows the first of its `worn` flags that is
above 0. Picking it lists the `choices` the player owns, and each opens that item's own menu, so equipping works
exactly as it does from the inventory. A `Stat` is a label and a dice expression over the player's values.

{% raw %}
```yaml
Resource:
  ironSwordResource:
    name: Iron Sword
    description: Plain, heavy, honest.
    default: 0
    min: '0'
    max: '1'
    reveal: 1
    menu: ironSwordMenu
  ironSwordEquippedResource:
    name: Iron Sword (worn)
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  ashStaffResource:
    name: Ash Staff
    default: 0
    min: '0'
    max: '1'
    reveal: 1
    menu: ashStaffMenu
  ashStaffEquippedResource:
    name: Ash Staff (worn)
    default: 0
    min: '0'
    max: '1'
    reveal: 999999
  strengthResource:
    name: Strength
    default: 10
    min: '0'
    max: '30'

Measure:
  attackPowerMeasure:
    name: Attack Power
    expression: '$strengthResource*2+$ironSwordEquippedResource*8+$ashStaffEquippedResource*3'

Condition:
  ownsIronSwordCondition:
    left: '$ironSwordResource'
    operator: '>'
    right: '0'
  ownsAshStaffCondition:
    left: '$ashStaffResource'
    operator: '>'
    right: '0'

Parameter:
  ironSwordEquippedParameter:
    key: resource
    value: ironSwordEquippedResource
  ashStaffEquippedParameter:
    key: resource
    value: ashStaffEquippedResource
  onParameter:
    key: value
    value: '1'
  offParameter:
    key: value
    value: '0'

Action:
  equipIronSwordAction:
    name: Equip
    if: ownsIronSwordCondition
    do: set_resource_self
    parameters: [ironSwordEquippedParameter, onParameter]
    then: dropAshStaffAction
  dropAshStaffAction:
    do: set_resource_self
    parameters: [ashStaffEquippedParameter, offParameter]
  unequipIronSwordAction:
    name: Unequip
    do: set_resource_self
    parameters: [ironSwordEquippedParameter, offParameter]
  equipAshStaffAction:
    name: Equip
    if: ownsAshStaffCondition
    do: set_resource_self
    parameters: [ashStaffEquippedParameter, onParameter]
    then: dropIronSwordAction
  dropIronSwordAction:
    do: set_resource_self
    parameters: [ironSwordEquippedParameter, offParameter]
  unequipAshStaffAction:
    name: Unequip
    do: set_resource_self
    parameters: [ashStaffEquippedParameter, offParameter]

Menu:
  ironSwordMenu:
    name: Iron Sword
    actions: [equipIronSwordAction, unequipIronSwordAction]
  ashStaffMenu:
    name: Ash Staff
    actions: [equipAshStaffAction, unequipAshStaffAction]

Slot:
  mainHandSlot:
    name: Main hand
    worn: [ironSwordEquippedResource, ashStaffEquippedResource]
    choices: [ironSwordResource, ashStaffResource]
    paired: true   # worn[i] goes with choices[i]: the page marks the worn item in its list
    column: 0
    row: 1

Stat:
  strengthStat:
    name: Strength
    value: '$strengthResource'
    description: Adds to Attack Power.
    row: 0
  attackPowerStat:
    name: Attack Power
    value: '$attackPowerMeasure'
    description: How hard your blows land.
    row: 1

Sheet:
  heroSheet:
    name: Character
    slots: [mainHandSlot]
    stats: [strengthStat, attackPowerStat]

Actor:
  heroActor:
    name: Hero
    sprite: heroSprite
    sheet: heroSheet
    measures: [attackPowerMeasure]
```
{% endraw %}

**Entities used.** [`Sheet`](/reference/entities.html#sheet), [`Slot`](/reference/entities.html#slot), [`Stat`](/reference/entities.html#stat), [`Resource`](/reference/entities.html#resource), [`Measure`](/reference/entities.html#measure), [`Menu`](/reference/entities.html#menu), [`Condition`](/reference/entities.html#condition), [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor)

**Gotchas.**

- A slot's grid counts rows from the bottom; the stats grid counts rows from the top.
- Equipping one item must unequip the others for the same slot (the `then` chains above), or two flags are on
  and the slot shows the first.
- Hide the `_equipped` flags with `reveal: 999999`. The page shows what is worn; the inventory shows what you own.
- A slot's `icon` and `name` show only while it is empty; once something is worn, that item's own icon and name
  show.
- Two ring slots can list the same rings; give each ring its own `_equipped` flag per slot if both can be worn.
- With `paired: true`, list `worn` and `choices` in the same order (one flag per item, same position). Validation
  rejects a paired slot whose lists differ in length. The list of choices then marks the worn item with a gold
  border, and its tooltip says "Equipped". Without `paired`, nothing in the list is marked.
- Resting the mouse on a slot or on an item in its list shows that item's name and description.
