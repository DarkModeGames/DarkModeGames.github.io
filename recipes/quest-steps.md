# Quest steps and an NPC that follows them

**Problem.** A villager asks you to kill three wolves, notices when you have, and rewards you once.

**Pattern.** Keep the quest's progress in a **step resource** (0 not taken, 1 hunting, 2 rewarded) and the count
in another. The villager's `menu` lists every option. Each option checks the step with `if` and refuses
politely otherwise. Kills are counted from the wolf's death action with an area effect over nearby players who are
on step 1.

{% raw %}
```yaml
Resource:
  wolfHuntStepResource:
    name: The Wolf Hunt
    default: 0
    min: '0'
    max: '2'
    reveal: 999999
  wolfKillsResource:
    name: Wolves Killed
    default: 0
    min: '0'
    max: '3'
    reveal: 1
  goldResource:
    name: Gold
    default: 0
    min: '0'
    max: '99999'

Condition:
  huntNotTakenCondition:
    left: '$wolfHuntStepResource'
    operator: '=='
    right: '0'
  huntDoneCondition:
    left: '$wolfKillsResource'
    operator: '>='
    right: '3'
  huntActiveCondition:
    left: '$wolfHuntStepResource'
    operator: '=='
    right: '1'
  targetHuntingCondition:
    left: '@wolfHuntStepResource'
    operator: '=='
    right: '1'

Parameter:
  wolfHuntStepParameter:
    key: resource
    value: wolfHuntStepResource
  wolfKillsParameter:
    key: resource
    value: wolfKillsResource
  goldParameter:
    key: resource
    value: goldResource
  stepOneValueParameter:
    key: value
    value: '1'
  stepTwoValueParameter:
    key: value
    value: '2'
  zeroValueParameter:
    key: value
    value: '0'
  oneExpressionParameter:
    key: expression
    value: '1'
  rewardParameter:
    key: expression
    value: '25'
  huntDialogParameter:
    key: dialog
    value: huntDialog
  notYetMessageParameter:
    key: message
    value: 'Not yet'
  questDoneMessageParameter:
    key: message
    value: 'Quest complete'
  privateAudienceParameter:
    key: audience
    value: private
  creditRadiusParameter:
    key: radius
    value: '300'
  creditEachParameter:
    key: action
    value: huntCreditAction

Dialog:
  huntDialog:
    title: The Wolf Hunt
    text: 'Three wolves have been at the sheep. Bring me word when they are dead. So far: {{$wolfKillsResource}} of 3.'

Action:
  takeHuntAction:
    name: Take the wolf hunt
    if: huntNotTakenCondition
    do: set_resource_self
    parameters: [wolfHuntStepParameter, stepOneValueParameter]
    then: showHuntAction
    else: notYetAction
  showHuntAction:
    do: open_dialog
    parameters: [huntDialogParameter]
  turnInHuntAction:
    name: Report the wolves dead
    if: huntActiveCondition
    then: turnInCheckAction
    else: notYetAction
  turnInCheckAction:
    if: huntDoneCondition
    do: set_resource_self
    parameters: [wolfHuntStepParameter, stepTwoValueParameter]
    then: turnInPayAction
    else: notYetAction
  turnInPayAction:
    do: plus_resource_self
    parameters: [goldParameter, rewardParameter]
    then: turnInClearAction
  turnInClearAction:
    do: set_resource_self
    parameters: [wolfKillsParameter, zeroValueParameter]
    then: questDoneAction
  questDoneAction:
    do: echo_self
    parameters: [questDoneMessageParameter]
  notYetAction:
    do: echo_self
    parameters: [notYetMessageParameter, privateAudienceParameter]
  # In the wolf's death chain: credit every hunting player within 300 px.
  wolfCreditAction:
    do: area_of_effect_at_self
    parameters: [creditRadiusParameter, creditEachParameter]
  huntCreditAction:
    if: targetHuntingCondition
    do: plus_resource_target
    parameters: [wolfKillsParameter, oneExpressionParameter]

Menu:
  shepherdMenu:
    name: Shepherd
    actions: [takeHuntAction, turnInHuntAction]

Actor:
  shepherdActor:
    name: Shepherd
    sprite: questGiverSprite
    menu: shepherdMenu
    dialog: huntDialog
```
{% endraw %}

**Entities used.** [`Action`](/reference/entities.html#action), [`Actor`](/reference/entities.html#actor), [`Condition`](/reference/entities.html#condition), [`Dialog`](/reference/entities.html#dialog), [`Menu`](/reference/entities.html#menu), [`Parameter`](/reference/entities.html#parameter), [`Resource`](/reference/entities.html#resource)

**Gotchas.**

- A menu shows every option every time; the `if` decides what happens. Name options so a refusal makes sense.
- The step resource is the one source of truth. Use one per quest, and never move it backwards.
- Kill credit runs from the **wolf**, so `@` is each player hit by the area. Filtering on `@wolfHuntStepResource`
  also keeps the wolf's friends and the scenery from counting.
- `{% raw %}{{$key}}{% endraw %}` and `{% raw %}{{@key}}{% endraw %}` in dialog text show live values: `$` is the player, `@` the NPC.
- Show `wolfKillsResource` with `reveal: 1` so the tally appears on the player's inventory once the first wolf
  falls.
