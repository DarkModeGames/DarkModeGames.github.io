# Choices that change dialog and shops

**Problem.** The player sides with the smith or the ferryman. Afterwards the smith should greet them differently,
and sell a blade only to friends.

**Pattern.** Record the choice in one resource (`sideResource`: 0 undecided, 1 smith, 2 ferryman) and the
relationship in another (`smithFavourResource`). An actor's own `dialog` is fixed, so give the NPC a "Talk" menu
option that branches on the choice with `if` and opens one of several [`Dialog`](/reference/entities.html#dialog)s
with `open_dialog`. Shop options check favour the same way.

```yaml
Resource:
  sideResource:
    name: Sided With
    default: 0
    min: '0'
    max: '2'
    reveal: 999999
  smithFavourResource:
    name: Smith's Favour
    default: 0
    min: '0'
    max: '100'
    reveal: 999999
  goldResource:
    name: Gold
    default: 100
    min: '0'
    max: '99999'
  fineBladeResource:
    name: Fine Blade
    default: 0
    min: '0'
    max: '1'
    reveal: 1

Condition:
  undecidedCondition:
    left: '$sideResource'
    operator: '=='
    right: '0'
  sidedWithSmithCondition:
    left: '$sideResource'
    operator: '=='
    right: '1'
  smithFriendCondition:
    left: '$smithFavourResource'
    operator: '>='
    right: '10'
  affordBladeCondition:
    left: '$goldResource'
    operator: '>='
    right: '60'

Parameter:
  sideParameter:
    key: resource
    value: sideResource
  smithFavourParameter:
    key: resource
    value: smithFavourResource
  goldParameter:
    key: resource
    value: goldResource
  fineBladeParameter:
    key: resource
    value: fineBladeResource
  smithSideValueParameter:
    key: value
    value: '1'
  ferrymanSideValueParameter:
    key: value
    value: '2'
  favourGainParameter:
    key: expression
    value: '10'
  bladePriceParameter:
    key: expression
    value: '60'
  oneExpressionParameter:
    key: expression
    value: '1'
  smithAskDialogParameter:
    key: dialog
    value: smithAskDialog
  smithWarmDialogParameter:
    key: dialog
    value: smithWarmDialog
  smithColdDialogParameter:
    key: dialog
    value: smithColdDialog
  smithRefuseDialogParameter:
    key: dialog
    value: smithRefuseDialog

Dialog:
  smithAskDialog:
    title: The Smith
    text: "The ferryman says my iron is bad. Will you tell the village otherwise?"
  smithWarmDialog:
    title: The Smith
    text: "Ah, my friend. Come in out of the cold."
  smithColdDialog:
    title: The Smith
    text: "The ferryman's friend. Buy what you need and go."
  smithRefuseDialog:
    title: The Smith
    text: "That blade is for friends of this forge."

Action:
  smithTalkAction:
    name: Talk
    if: undecidedCondition
    do: open_dialog
    parameters: [smithAskDialogParameter]
    else: smithTalkSidedAction
  smithTalkSidedAction:
    if: sidedWithSmithCondition
    do: open_dialog
    parameters: [smithWarmDialogParameter]
    else: smithTalkColdAction
  smithTalkColdAction:
    do: open_dialog
    parameters: [smithColdDialogParameter]
  sideWithSmithAction:
    name: Speak up for the smith
    if: undecidedCondition
    do: set_resource_self
    parameters: [sideParameter, smithSideValueParameter]
    then: sideWithSmithFavourAction
  sideWithSmithFavourAction:
    do: plus_resource_self
    parameters: [smithFavourParameter, favourGainParameter]
    then: smithTalkSidedAction
  sideWithFerrymanAction:
    name: Side with the ferryman
    if: undecidedCondition
    do: set_resource_self
    parameters: [sideParameter, ferrymanSideValueParameter]
    then: smithTalkColdAction
  buyBladeAction:
    name: Buy the Fine Blade (60g)
    if: smithFriendCondition
    then: buyBladePayAction
    else: buyBladeRefusedAction
  buyBladePayAction:
    if: affordBladeCondition
    do: minus_resource_self
    parameters: [goldParameter, bladePriceParameter]
    then: buyBladeTakeAction
  buyBladeTakeAction:
    do: plus_resource_self
    parameters: [fineBladeParameter, oneExpressionParameter]
  buyBladeRefusedAction:
    do: open_dialog
    parameters: [smithRefuseDialogParameter]

Menu:
  smithMenu:
    name: The Smith
    actions: [smithTalkAction, sideWithSmithAction, sideWithFerrymanAction, buyBladeAction]

Actor:
  smithActor:
    name: The Smith
    sprite: villagerSprite
    menu: smithMenu
```

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Dialog`](/reference/entities.html#dialog), [`Menu`](/reference/entities.html#menu), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource)

**Gotchas.**

- `open_dialog` swaps the text in the open panel, so the menu stays usable underneath.
- Make choices one-way: every choice option checks "undecided" first.
- Favour is a number, not a flag, so later deeds can add to it and different shops can ask for different amounts.
- Dialog text can show live numbers with `{% raw %}{{$smithFavourResource}}{% endraw %}` (`$` is the player).
- The ferryman reads the same `sideResource` for his side of the story.
