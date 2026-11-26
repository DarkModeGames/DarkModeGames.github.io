# A vendor with a gold check

**Problem.** A shopkeeper should sell potions only to players who can pay, and buy them back for less.

**Pattern.** Give the vendor actor a `menu`. Each option is an [`Action`](/reference/entities.html#action) with a `name` (the label) and a chain of
checks: is there room in the bag, can the player afford it, take the gold, hand over the item. The buyer is the
caller (`$`), so every step uses `*_self`. A refusal says why, privately.

```yaml
Resource:
  goldResource:
    name: Gold
    default: 50
    min: '0'
    max: '99999'
  healingDraughtResource:
    name: Healing Draught
    default: 0
    min: '0'
    max: '5'
    reveal: 1

Condition:
  draughtRoomCondition:
    left: '$healingDraughtResource'
    operator: '<'
    right: '5'
  affordDraughtCondition:
    left: '$goldResource'
    operator: '>='
    right: '10'
  ownsDraughtCondition:
    left: '$healingDraughtResource'
    operator: '>='
    right: '1'

Parameter:
  goldParameter:
    key: resource
    value: goldResource
  healingDraughtParameter:
    key: resource
    value: healingDraughtResource
  draughtPriceParameter:
    key: expression
    value: '10'
  draughtSellParameter:
    key: expression
    value: '4'
  oneExpressionParameter:
    key: expression
    value: '1'
  tooPoorMessageParameter:
    key: message
    value: 'Not enough gold'
  bagFullMessageParameter:
    key: message
    value: 'You cannot carry more'
  privateAudienceParameter:
    key: audience
    value: private

Action:
  buyDraughtAction:
    name: Buy Healing Draught (10g)
    if: draughtRoomCondition
    then: buyDraughtPayAction
    else: bagFullAction
  buyDraughtPayAction:
    if: affordDraughtCondition
    do: minus_resource_self
    parameters: [goldParameter, draughtPriceParameter]
    then: buyDraughtTakeAction
    else: tooPoorAction
  buyDraughtTakeAction:
    do: plus_resource_self
    parameters: [healingDraughtParameter, oneExpressionParameter]
  sellDraughtAction:
    name: Sell Healing Draught (4g)
    if: ownsDraughtCondition
    do: minus_resource_self
    parameters: [healingDraughtParameter, oneExpressionParameter]
    then: sellDraughtPaidAction
  sellDraughtPaidAction:
    do: plus_resource_self
    parameters: [goldParameter, draughtSellParameter]
  tooPoorAction:
    do: echo_self
    parameters: [tooPoorMessageParameter, privateAudienceParameter]
  bagFullAction:
    do: echo_self
    parameters: [bagFullMessageParameter, privateAudienceParameter]

Menu:
  apothecaryMenu:
    name: Apothecary
    actions: [buyDraughtAction, sellDraughtAction]

Actor:
  apothecaryActor:
    name: Apothecary
    sprite: vendorSprite
    targetable: true
    menu: apothecaryMenu
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Menu`](/reference/entities.html#menu), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource)

**Gotchas.**

- From an NPC's menu the player is the caller and the NPC is the target. A player walks up to the NPC
  (right-click) and the menu opens when they are close.
- Check room before taking gold. Taking gold and then failing to fit the item loses the player's money.
- Item counts stop at their `max`, so the room check is what stops a 6th potion, not the bounds.
- The engine doesn't refresh a menu after an option runs. That's fine for a shop where the player buys again and
  again; close it first when an option should disappear.
- To gate a shop on reputation, add one more `if` step on a reputation resource at the front of the chain.
