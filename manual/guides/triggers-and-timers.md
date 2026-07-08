# Triggers and Timers

This page shows you how to make an actor react automatically. A *trigger* runs an [action](/manual/guides/define-actions.html) when one of the actor's resources changes, and a *timer* runs an action over and over on an interval. You attach both to an actor and let the engine fire them.

## Introduction

Most actions in a campaign run because something asks for them: a player clicks an ability, or an AI [strategy](/manual/getting-started/glossary.html) decides to act. Triggers and timers are different. They are passive watchers attached to an actor that run an action on their own.

You use a **Trigger** when you want a reaction to a value: heal a wound when health drops, or check for death whenever health changes. You use a **Timer** when you want something to happen on a clock: regenerate health every few seconds.

Both are [entities](/manual/getting-started/glossary.html). You define them in YAML, then point an actor at them. The engine builds the watchers when the actor spawns.

> Note: Triggers and timers only run while the game is running on the host or server. They do not run on a plain client.

## Prerequisites

This page assumes you have already done the following:

- [Create an actor](/manual/guides/create-an-actor.html), so you have an actor to attach watchers to.
- [Add resources and measures](/manual/guides/add-resources-and-measures.html), so the trigger has a resource to watch and the timer has a resource to change.
- [Define actions](/manual/guides/define-actions.html), so you have an action for each watcher to run.

You should also know how to write a [dice expression](/manual/getting-started/glossary.html) (see [Dice expressions](/manual/core-concepts/dice-expressions.html)), because a timer's timing is written in dice notation.

## Triggers

A **Trigger** watches one [resource](/manual/getting-started/glossary.html) on an actor and runs an action whenever that resource's value changes to a new value.

This matters because some reactions should happen the instant a value moves, not on a fixed clock. Watching health is the classic case: the moment health changes, you want to check whether the actor just died.

### Define a trigger

A Trigger entity has two fields, both optional:

- `resource` is a [KeyRef](/manual/getting-started/glossary.html) to the [Resource](/manual/getting-started/glossary.html) to watch.
- `action` is a KeyRef to the [Action](/manual/guides/define-actions.html) to run when that resource changes.

Here is the `deathTrigger`, which watches the actor's `health` resource and runs `deathAction`:

```yaml
Trigger:
  deathTrigger:
    resource: health
    action: deathAction
```

When `health` changes to any new value, the trigger fires `deathAction`. Both the caller and the target of that action are the actor itself.

> Warning: A trigger fires on *any* change to the resource, an increase as well as a decrease. It does not detect death on its own. The death logic lives in the action, not the trigger.

### Gate the effect inside the action

Because the trigger fires on every change, the action it runs decides whether anything actually happens. The example does this with an `if` [condition](/manual/guides/conditions-and-logic.html) on `deathAction`:

```yaml
Action:
  deathAction:
    name: Death Check
    if: isDead
    do: echo
    parameters:
    - deathMsg
```

The `if: isDead` condition checks whether health has reached zero. Only then does the `echo` run. So the trigger fires constantly as health moves, but the message appears only on death. This split, a trigger that watches and an action that decides, keeps each piece simple.

> Note: `parameters` is a flat array of KeyRefs to `Parameter` entities, defined in a sibling top-level `Parameter` block. See [Define actions](/manual/guides/define-actions.html) for how parameters resolve.

## Timers

A **Timer** runs an action over and over on an interval. Unlike a trigger, it does not watch anything; it acts on a clock.

You use a timer for effects that should repeat on their own: health regeneration, a recurring scan, ongoing damage over time.

### Define a timer

A Timer entity has three fields, all optional:

- `interval` is a [dice expression](/manual/core-concepts/dice-expressions.html) giving the number of seconds between runs. It is rolled again before every run, so the gap can vary.
- `total` is a dice expression giving the number of *times* to run before the timer stops. It is rolled once when the timer starts.
- `action` is a KeyRef to the [Action](/manual/guides/define-actions.html) to run each time.

Here is the `regenTimer`:

```yaml
Timer:
  regenTimer:
    total: ''
    interval: 1d12
    action: regenAction
```

Every `1d12` seconds the timer runs `regenAction`. The interval is re-rolled each time, so the wait is one to twelve seconds, rarely the same twice in a row.

> Warning: `total` counts intervals, not seconds. If `total` evaluates to `0`, the timer never expires and runs for the actor's whole life. The example leaves `total` empty, which evaluates to `0`, so `regenTimer` regenerates health forever.

### The action a timer runs

The `regenTimer` runs `regenAction`, which heals the actor:

```yaml
Action:
  regenAction:
    name: Regen Health
    do: plus_resource_self
    parameters:
    - healResourceParam
    - regenAmountParam
```

The parameters resolve to `healResourceParam` (`{"key": "resource", "value": "health"}`) and `regenAmountParam` (`{"key": "expression", "value": "1d4"}`). So each interval adds `1d4` to `health`. Combined with the `1d12` interval, `regenTimer` heals one to four health every one to twelve seconds.

> Note: A timer's action targets the actor's current target if it has one, falling back to the actor itself; the actor is always the caller. This differs from a trigger, whose action always targets the actor itself.

## Attach watchers to an actor

A Trigger or Timer does nothing until an actor references it. The [Actor](/manual/guides/create-an-actor.html) entity has two [KeyRefArray](/manual/getting-started/glossary.html) fields for this:

- `triggers` lists Trigger keys.
- `timers` lists Timer keys.

The hero actor uses both:

```yaml
Actor:
  heroActor:
    name: Hero
    base: 8
    speed: 60
    triggers:
    - deathTrigger
    timers:
    - regenTimer
```

When the hero spawns on the host or server, the engine arms `deathTrigger` to watch the hero's `health` and starts `regenTimer` on its `1d12` interval.

> Tip: You can reuse one watcher across many actors. The `guardActor` also lists `"triggers": ["deathTrigger"]`, so both the hero and the guard run the same death check.

## Steps: add regeneration to an actor

To give an actor self-healing, follow these steps:

1. Confirm the actor has a `health` [resource](/manual/guides/add-resources-and-measures.html) with a `max` it can heal toward.
2. Write a healing [action](/manual/guides/define-actions.html) that uses `plus_resource_self`, with parameters naming the resource and a dice expression for the amount, as `regenAction` does.
3. Define a `Timer` entity with an `interval` dice expression, a `total` of `""` to run forever, and `action` pointing at your healing action.
4. Add the timer's key to the actor's `timers` array.
5. Load the campaign on the host or server and watch the resource climb.

Every key you reference, the trigger's `resource` and `action`, the timer's `action`, and the actor's `triggers` and `timers`, is checked when the campaign loads. The `interval` and `total` expressions are validated as dice expressions. A typo stops the campaign before it runs, so fix any reported reference or expression error before testing.

## See also

- [Create an actor](/manual/guides/create-an-actor.html)
- [Add resources and measures](/manual/guides/add-resources-and-measures.html)
- [Define actions](/manual/guides/define-actions.html)
- [Conditions and logic](/manual/guides/conditions-and-logic.html)
- [Dice expressions](/manual/core-concepts/dice-expressions.html)
- [Add NPC AI](/manual/guides/add-npc-ai.html)
- [Entity reference](/reference/entities.html)
- [Action function reference](/reference/actions.html)
