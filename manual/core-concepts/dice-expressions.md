# Dice Expressions

A few numeric fields in a campaign accept a [dice expression](/manual/getting-started/glossary.html#terms) instead of a fixed number: a short string in tabletop dice notation that the engine rolls to produce an integer.

## Introduction

A dice expression lets a value vary at runtime. Instead of writing a flat `10`, you write `1d20` and the engine rolls a twenty-sided die each time the value is read. This page covers where dice expressions are allowed, the exact set of characters they may contain, the operators the engine understands, and the integer-only rules that govern the result.

Only four fields across all {{ entity_count }} entity types accept dice expressions. Everything else that looks numeric is a plain number. Knowing which fields are dice fields, and which characters they accept, keeps your campaign from failing [validation](/manual/core-concepts/validation.html).

## Prerequisites

Read these pages first:

- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html)
- [Campaign structure](/manual/core-concepts/campaign-structure.html)

## What a dice expression is

A dice expression is a string written in dice notation. The core piece is the **dice pool**, written `NdM`: roll `N` dice, each with `M` sides, and sum the results. So `3d6` rolls three six-sided dice and adds them together.

The `N` is optional and defaults to `1`. Both lowercase `d` and uppercase `D` work, so `d20` and `1D20` mean the same thing. Each die rolls a whole number from `1` to `M`.

A `luckMeasure` is the simplest case: a single twenty-sided die.

```yaml
Measure:
  luckMeasure:
    expression: 1d20
    icon: /assets/icons/sun.png
    public: false
    private: true
    reveal: 0
```

Each time an actor reads `luckMeasure`, the engine rolls `1d20` and returns a value from `1` to `20`.

## The valid character set

A dice expression may contain only these characters:

```
0123456789()*%/+-dD<>
```

That is: the digits, parentheses, the four arithmetic operators (`*`, `/`, `+`, `-`), the modulus sign (`%`), the dice letters `d` and `D`, and the filter signs (`>`, `<`).

Any other character is rejected. At load time, validation reports an `INVALID_DICE` error naming the bad character and its position. At runtime, the roll logs an error and returns `0`.

> Warning: The character set excludes the decimal point. Dice expressions are integer-only, so `0.25` or `1d4*0.25` will not validate. To express a fraction, scale up to integers (see [Sound scale as a percentage](#sound-scale-as-a-percentage)).

## Operators and evaluation order

You can combine dice pools and numbers with arithmetic. The engine evaluates an expression in this order:

1. Parentheses, innermost first.
2. Filters (`>` and `<`).
3. Dice pools (`NdM`).
4. Multiplication (`*`).
5. Division (`/`).
6. Modulus (`%`).
7. Addition (`+`).
8. Subtraction (`-`).

Use parentheses to group a sub-expression and force it to resolve first. An `attackSound` that rolls a hundred-sided die inside parentheses, then adds `50`:

```yaml
Sound:
  attackSound:
    source: /assets/Audio/fx.mp3
    scale: 50+(1d100)
```

This produces a value from `51` to `150`.

### Keep-highest and keep-lowest filters

A filter trims a dice pool down to its best or worst rolls before summing. Write `NdM>X` to keep the highest `X` rolls, or `NdM<X` to keep the lowest `X` rolls. For example, `4d6>3` rolls four six-sided dice and sums the highest three. The kept rolls are added together.

## Integer rules and capping

Every dice expression evaluates to a whole number. Arithmetic follows these rules:

- **Division rounds toward zero.** `10/3` is `3`.
- **Division or modulus by zero yields `0`.** `10/0` is `0`, and `7%0` is `0`.
- **Subtraction caps at zero.** `3-5` is `0`, not `-5`.
- **The final result is capped at a minimum of `0`.** A dice expression never returns a negative number.
- **An empty or absent expression evaluates to `0`.**

That last rule matters for measures: a `Measure` with no `expression` field is valid and simply reads as `0`.

> Note: All `Measure` fields, including `expression`, are optional.

## Which fields accept dice expressions

Exactly four fields are dice fields. They are the only fields the engine validates as dice notation:

| Entity | Field | Meaning |
| --- | --- | --- |
| `Measure` | `expression` | The value calculated fresh each time the [measure](/manual/guides/add-resources-and-measures.html) is read. |
| `Sound` | `scale` | The audio pitch, as a percentage of normal (see below). |
| `Timer` | `total` | The total duration, in seconds. |
| `Timer` | `interval` | The seconds between repeated action runs. |

Any other numeric-looking field, such as an actor's `speed` or a resource's `max`, is a plain number, not a dice expression.

<a id="sound-scale-as-a-percentage"></a>
### Sound scale as a percentage

A `Sound`'s `scale` is rolled to an integer, then divided by `100.0` to set the audio's pitch. Treat the value as a percentage of normal pitch: `100` is normal speed, `50` is half, `200` is double.

A `bgMusic` that uses `1d200` spans a pitch from near-silent crawl up to double speed:

```yaml
Sound:
  bgMusic:
    source: /assets/Audio/Crystal caverns Chimes 1.mp3
    scale: 1d200
    loop: true
```

This is why `scale` uses dice expressions rather than a decimal pitch: the integer-only engine has no `0.5`, so you write `50` and the engine divides for you. See [Audio and backgrounds](/manual/guides/audio-and-backgrounds.html) for more on sounds.

### Timer total and interval

A `Timer` runs an [action](/manual/guides/define-actions.html) on a schedule. Its `interval` is the seconds between runs and its `total` is the overall duration; both accept dice expressions, so a timer's timing can vary each time it starts. A `regenTimer` that fires its action every `1d12` seconds:

```yaml
Timer:
  regenTimer:
    total: ''
    interval: 1d12
    action: regenAction
```

The empty `total` reads as `0`. The `action` field is a [KeyRef](/manual/getting-started/glossary.html#terms) to an Action entity, not a dice field. See [Triggers and timers](/manual/guides/triggers-and-timers.html) for the full timer lifecycle.

## What does not belong in a dice field

Some parts of a campaign use the `$` and `@` markers to pull in live values at runtime. `$` reads a value from the **caller** and `@` reads a value from the **target**. These markers look like they might fit a dice expression, but they do not.

> Warning: `$` and `@` are not in the dice character set (`0123456789()*%/+-dD<>`). A marker inside a `Measure.expression`, `Sound.scale`, or `Timer` field fails validation with `INVALID_DICE`. Keep markers out of dice fields.

The markers live in two other places:

- In [Condition](/manual/guides/conditions-and-logic.html) operands, such as `@health`, which compares against the target's `health`.
- In [dialog](/manual/guides/menus-dialogs-waypoints.html) text, using the templating tokens below.

When a dialog shows a live value, it wraps the marker in double braces:

{% raw %}
```
HP: {{@health}} / Your luck: {{$luck}}
```
{% endraw %}

These templating paths do not go through dice validation. They are covered on the conditions and dialogs pages, not here. No dice field uses a marker.

> Note: Markers behave defensively. A key that names no resource or measure resolves to `0`, and a missing actor resolves its marker to `0`.

## See also

- [Validation](/manual/core-concepts/validation.html)
- [Add resources and measures](/manual/guides/add-resources-and-measures.html)
- [Triggers and timers](/manual/guides/triggers-and-timers.html)
- [Audio and backgrounds](/manual/guides/audio-and-backgrounds.html)
- [Conditions and logic](/manual/guides/conditions-and-logic.html)
- [Menus, dialogs, and waypoints](/manual/guides/menus-dialogs-waypoints.html)
- [Action function reference](/reference/actions.html)
