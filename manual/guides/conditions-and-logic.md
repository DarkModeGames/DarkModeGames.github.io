# Conditions and Logic

A [Condition](/manual/getting-started/glossary.html) is a single yes-or-no comparison that the engine uses to decide whether an [action](/manual/getting-started/glossary.html) runs and which branch it takes, and to decide what an AI actor does next. This page explains the three fields of a Condition, the operator symbols you can use, how operands resolve to live actor values, and how the engine evaluates the result.

## Introduction

Most gameplay logic comes down to a question: is this actor hurt? do I have a target? am I close enough to attack? A Condition answers one such question by comparing a left value to a right value with an operator. The answer, `true` or `false`, gates two things:

- An action's branching. An action runs its `do` only when its `if` Condition passes, then chains to `then` or `else`.
- An AI actor's behavior. A [behavior](/manual/getting-started/glossary.html) lists Conditions as goals, and runs its action only when all goals pass.

Because both operands flow through the same engine that rolls [dice expressions](/manual/core-concepts/dice-expressions.html), a Condition can compare plain numbers, dice rolls, or live values pulled from an actor's resources and measures.

## Prerequisites

Read these pages first. This page builds directly on them:

- [Define actions](/manual/guides/define-actions.html) — how `if`, `do`, `then`, and `else` fit together in an action.
- [Add resources and measures](/manual/guides/add-resources-and-measures.html) — the actor values a Condition reads.
- [Dice expressions](/manual/core-concepts/dice-expressions.html) — the evaluator that turns an operand string into a number.

## The three fields of a Condition

A Condition entity lives in your campaign YAML under a `Condition` block. It has three fields:

- `left` — the left operand of the comparison. Optional. If you omit it or leave it empty, it evaluates to `0`.
- `operator` — the comparison symbol. **Required.** This is the only required field.
- `right` — the right operand. Optional. Same rules as `left`; an empty value evaluates to `0`.

A Condition holds data only; it contains no logic of its own. The engine evaluates it when an action or behavior asks.

Here is the `isHurt` Condition, which asks whether the target's health is below 20:

```yaml
Condition:
  isHurt:
    left: '@health'
    operator: <
    right: '20'
```

At evaluation time the engine resolves `@health` to the target actor's `health` resource, treats `20` as the literal number twenty, and returns `true` when the resource is less than twenty.

> Note: An action that has no `if` Condition always runs. A missing or empty `if` auto-passes, so you only add a Condition when you want to gate something.

## Operator symbols

The `operator` field takes one of these symbols:

| Meaning | Symbols |
| --- | --- |
| Equal | `=`, `==` |
| Not equal | `!=`, `<>` |
| Greater than | `>` |
| Less than | `<` |
| Greater than or equal | `>=` |
| Less than or equal | `<=` |

Both spellings of equal and not-equal are valid. The `hasTarget` Condition uses the single `=` form:

```yaml
Condition:
  hasTarget:
    left: '@has_target'
    operator: '='
    right: '1'
```

> Warning: If you write an operator the engine does not recognize, it logs a warning and the Condition counts as `true`. A typo in this field will silently let gated logic run, so check the symbol against the table above.

## How operands resolve

Each operand string is run through the dice and expression engine. The engine handles three kinds of value in one operand:

1. A literal number, such as `"20"` or `"0"`.
2. A [dice expression](/manual/core-concepts/dice-expressions.html), such as `1d6`, which the engine rolls.
3. A reference to a live actor value, marked with `@` or `$`.

### Target and caller markers

A reference operand starts with one of two markers:

- `@key` reads from the **target** actor — the actor the action is acting on.
- `$key` reads from the **caller** actor — the actor running the action.

For each reference, the engine looks up `key` first in that actor's [resources](/manual/getting-started/glossary.html), then in its [measures](/manual/getting-started/glossary.html). If the key is found in neither, or the actor does not exist, the operand evaluates to `0`.

> Note: This is the same `@`/`$` convention used by [dialog](/manual/getting-started/glossary.html) templating. There the tokens read values for display; in a Condition they read values for comparison.

For example, `@health` reads the target's `health` resource, while `$health` would read the caller's own `health`. The example Conditions all use `@`, because they gate actions and AI that act on a target.

### Built-in measures

Some operands reference built-in measures rather than resources you defined. These are values the engine computes about an actor at runtime:

- `has_target` — `1` if the actor has a target set, otherwise `0`.
- `distance_to_target` — the isometric distance to the actor's current target, or `0` if there is no target.
- `distance_to_destination` — the isometric distance to the actor's destination.
- `flying` — `1` if the actor is currently flying, otherwise `0`. For example, gate a melee action on `@flying = 0` so it misses airborne targets.

So the Condition `@has_target = 1` reads as "the target actor has a target."

> Note: When an action runs with no target, the target actor is empty and any `@key` operand resolves to `0`. Design target-dependent Conditions with that fallback in mind.

## How the engine evaluates a Condition

When an action or behavior needs the answer, the engine does the following:

1. Looks up the Condition entity by its key.
2. Evaluates `left` through the dice engine, supplying the caller and target actors so `$` and `@` references resolve.
3. Evaluates `right` the same way.
4. Compares the two results with `operator` and returns `true` or `false`.

Both operands are reduced to whole numbers before the comparison, and any result below `0` is treated as `0`. The comparison itself is always whole number to whole number.

## Gating an action

An action names a Condition in its `if` field. When the action runs:

- If the Condition is `true`, the engine runs the action's `do`, then chains to `then` if one is set.
- If the Condition is `false`, the engine chains to `else` if one is set; otherwise nothing runs.

The `conditionalHealAction` (Smart Heal) shows the full pattern. It heals only a hurt actor and reports which branch ran:

```yaml
Action:
  conditionalHealAction:
    name: Smart Heal
    if: isHurt
    do: plus_resource_self
    then: healConfirmAction
    else: alreadyFullAction
    parameters:
    - healResourceParam
    - healAmountParam
```

When `isHurt` passes, the action adds health and chains to `healConfirmAction`. When it fails, it skips the heal and chains to `alreadyFullAction` instead.

> Note: `parameters` is a flat array of KeyRefs to `Parameter` entities defined in a sibling top-level `Parameter` block. See [Define actions](/manual/guides/define-actions.html) for how parameters are wired.

A simpler case is `deathAction`, where one Condition gates one effect:

```yaml
Action:
  deathAction:
    name: Death Check
    if: isDead
    do: echo
    parameters:
    - deathMsg
```

The `isDead` Condition (`@health <= 0`) must pass before the message runs.

## Gating AI behavior

A behavior's `goals` field is an array of Condition keys. The engine evaluates every goal Condition, using the behaving actor as the caller and its current target as the target. The behavior's action runs only if **all** goals pass — the goals are combined with logical AND.

A strategy wires Conditions as AI goals:

```yaml
Behavior:
  patrolBehavior:
    goals:
    - hasTarget
    action: patrolAction
  followBehavior:
    goals:
    - isNearTarget
    action: pursueAction
```

`patrolBehavior` runs its patrol only while the actor has a target, and `followBehavior` pursues only while the target is more than one tile away (`@distance_to_target > 1`). See [Add NPC AI](/manual/guides/add-npc-ai.html) for how behaviors combine into a [strategy](/manual/getting-started/glossary.html).

## The example Conditions at a glance

The example campaign defines five Conditions, all reading the target actor:

| Key | Comparison | Reads as |
| --- | --- | --- |
| `isDead` | `@health <= 0` | health at or below zero |
| `isHurt` | `@health < 20` | health below twenty |
| `hasTarget` | `@has_target = 1` | the actor has a target |
| `isNearTarget` | `@distance_to_target > 1` | the target is more than one tile away |
| `isAtDestination` | `@distance_to_destination < 1` | the actor has reached its destination |

Use these as templates: change the operand key, operator, and right-hand value to ask your own question.

## See also

- [Define actions](/manual/guides/define-actions.html)
- [Dice expressions](/manual/core-concepts/dice-expressions.html)
- [Add resources and measures](/manual/guides/add-resources-and-measures.html)
- [Add NPC AI](/manual/guides/add-npc-ai.html)
- [Add skills](/manual/guides/add-skills.html)
- [Validation](/manual/core-concepts/validation.html)
- [Action function reference](/reference/actions.html)
