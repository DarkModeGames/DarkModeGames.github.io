# Define Actions

An [Action](/manual/getting-started/glossary.html#terms) entity is how a campaign changes the game: it deals damage, moves an actor, opens a dialog, or chains into more actions. This page shows how to write an action, pass values to it with [parameters](/manual/getting-started/glossary.html#terms), and branch on a [condition](/manual/getting-started/glossary.html#terms).

## Introduction

Every action names one built-in *action function* and hands it the values it needs. The function does the work; your YAML decides which function runs, with what inputs, under what condition, and what happens next. There are exactly {{ action_count }} action functions, listed in the [action function reference](/reference/actions.html).

This page covers the `do` field that picks the function, the `parameters` array that feeds it, the `if`/`then`/`else` fields that chain actions conditionally, and the `time` and `animation` fields that make an action take real seconds on screen.

## Prerequisites

Read these pages first:

- [Create an actor](/manual/guides/create-an-actor.html) — actions are run by and against actors.
- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html) — actions and parameters reference each other by key.
- [Dice expressions](/manual/core-concepts/dice-expressions.html) — parameter values can be dice rolls such as `(1d6)-1`.

## What an action does

An action is a thin wrapper around a function. The `do` field holds the function's name, and that function performs the effect. For example, `minus_resource_target` subtracts a value from a resource on the action's target.

The `do` field is the only required field. Here is the smallest useful action from an example campaign:

```yaml
Action:
  attackAction:
    name: Attack
    time: 1.1
    animation: tool
    do: minus_resource_target
    parameters:
    - attackResourceParam
    - attackDamageParam
```

This action is named "Attack", takes `1.1` seconds, plays the `tool` animation, and runs `minus_resource_target` with two parameters. The `do` value must be one of the {{ action_count }} valid function names; an unknown name fails campaign validation with an `INVALID_ACTION_NAME` error.

> Note: Write clean YAML keys only — `name`, `if`, `then`, `else`, `do`, `time`, `animation`. The engine uses internal aliases such as `name_` and `if_` in GDScript, but your entity files never use them.

## Pass values with parameters

Most functions need inputs. `minus_resource_target` needs to know *which* resource to subtract from and *how much*. You supply those through [Parameter](/manual/getting-started/glossary.html#terms) entities.

The `parameters` field is a **flat array of [KeyRefs](/manual/getting-started/glossary.html#terms)** pointing at [Parameter](/reference/entities.html#parameter) entities. Each Parameter is defined separately in a sibling top-level `Parameter` block. You never nest a parameter's definition inside the action.

A Parameter has two required string fields: `key` and `value`. The `key` must match the argument the action function reads. The `value` is the input, written as a string — a literal, a dice expression, or a KeyRef to another entity.

The example defines the two parameters that `attackAction` references:

```yaml
Parameter:
  attackResourceParam:
    key: resource
    value: health
  attackDamageParam:
    key: expression
    value: (1d6)-1
```

The `resource` parameter tells the function to act on `health`. The `expression` parameter is a [dice expression](/manual/core-concepts/dice-expressions.html): the engine rolls `(1d6)-1` each time to decide the damage. [Resource](/reference/entities.html#resource) changes stay within the resource's `min` and `max` bounds (see [Add resources and measures](/manual/guides/add-resources-and-measures.html)).

### Why the names must match

When an action runs, the engine hands each Parameter's `value` to the function under its `key`. So the Parameter's `key` is what the function looks up. If a function expects a `resource` and your Parameter uses `target` instead, the function gets nothing.

The [action function reference](/reference/actions.html) lists the parameter names each function expects. A few common ones:

- `minus_resource_target`, `plus_resource_self`, and the other resource functions read `resource` and `expression`.
- `echo` reads `message`.
- `open_dialog` reads `dialog` (a KeyRef to a [Dialog](/reference/entities.html#dialog) entity).
- `use_track` reads index-numbered parameters (names `0`, `1`, `2`, …; values are [Vector](/reference/entities.html#vector) keys), visited in ascending index order.
- `change_strategy` reads `strategy`.
- `set_destination_self` reads `destination`, a KeyRef to a Vector entity.

Some functions take no parameters at all — for example `wait`, `target_nearest`, `move_to_target`, `swap_positions`, and `despawn_self`. For those you can omit `parameters` entirely.

> Note: The validator checks that every required parameter is present, that there are no unexpected parameter keys, and that each value type-checks (keyrefs resolve, dice parse, numbers/colors are valid). A wrong or missing `key` now fails campaign validation rather than misbehaving at runtime.

## Branch with if, then, and else

An action can run conditionally. The `if` field is a KeyRef to a [Condition](/manual/getting-started/glossary.html#terms) entity. When `if` is present, the engine evaluates the condition first and only runs `do` when the condition passes. When `if` is absent, the action always runs.

Two more fields chain actions:

- `then` is a KeyRef to another Action that runs **after** `do` finishes, when the condition passed.
- `else` is a KeyRef to another Action that runs **instead** of `do`, when the condition failed.

The `conditionalHealAction` shows all three together:

```yaml
Action:
  conditionalHealAction:
    if: isHurt
    do: plus_resource_self
    then: healConfirmAction
    else: alreadyFullAction
    parameters:
    - healResourceParam
    - healAmountParam
```

If the `isHurt` condition passes, the actor heals (`plus_resource_self`) and then runs `healConfirmAction`. If `isHurt` fails, the actor is already at full health, so the engine runs `alreadyFullAction` instead. See [Conditions and logic](/manual/guides/conditions-and-logic.html) for how to write the `isHurt` condition.

## Add timing and animation

Two fields control how an action plays out over time.

`time` is how many seconds the action takes, defaulting to `0.0` for an instant effect. `animation` is a KeyRef to an [Animation](/manual/getting-started/glossary.html#terms) entity that plays while the action runs. The `attackAction` uses `"time": 1.1` and `"animation": "tool"`, so the attack plays the `tool` animation across just over a second.

> Note: If you set `animation`, set `time` to a value greater than `0.0`. An animation needs a non-zero duration to play. See [Visuals: sprites and animations](/manual/guides/visuals-sprites-animations.html) for defining [Animation](/reference/entities.html#animation) entities.

## Dialogs and live values

Functions like `open_dialog` show a [dialog](/manual/getting-started/glossary.html#terms) — a titled text panel. Dialog text can display live actor values with templating tokens: `@` reads the target actor and `$` reads the caller.

{% raw %}
```
HP: {{@health}} / Your luck: {{$luck}}
```
{% endraw %}

Here `{% raw %}{{@health}}{% endraw %}` shows the target's `health` resource and `{% raw %}{{$luck}}{% endraw %}` shows the caller's `luck` measure. Put these tokens only in a Dialog entity's `text` field, then point an action's `dialog` parameter at that Dialog.

## See also

- [Conditions and logic](/manual/guides/conditions-and-logic.html)
- [Dice expressions](/manual/core-concepts/dice-expressions.html)
- [Add skills](/manual/guides/add-skills.html)
- [Action function reference](/reference/actions.html)
- [Add resources and measures](/manual/guides/add-resources-and-measures.html)
- [Visuals: sprites and animations](/manual/guides/visuals-sprites-animations.html)
- [Entity reference](/reference/entities.html)
