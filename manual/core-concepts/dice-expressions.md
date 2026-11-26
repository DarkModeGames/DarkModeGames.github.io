# Dice Expressions

A few numeric fields in a campaign accept a [dice expression](/manual/getting-started/glossary.html#terms) instead of a fixed number: a short string in tabletop dice notation that the engine rolls to produce an integer.

## Introduction

A dice expression lets a value vary at runtime. Instead of writing a flat `10`, you write `1d20` and the engine rolls a twenty-sided die each time the value is read. This page covers where dice expressions are allowed, the exact set of characters they may contain, the operators the engine understands, and the integer-only rules that govern the result.

Dice expressions appear in a handful of fields (listed [below](#which-fields-accept-dice-expressions)) and in many action parameters, such as damage amounts and radii. They can also read live values from actors with `$key` and `@key`. Knowing which fields are dice fields, and which characters they accept, keeps your campaign from failing [validation](/manual/core-concepts/validation.html).

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

That is: the digits, parentheses, the four arithmetic operators (`*`, `/`, `+`, `-`), the modulus sign (`%`), the dice letters `d` and `D`, and the filter signs (`>`, `<`). On top of that, it may contain **value references**: `$key` and `@key` (see [Reading live values](#reading-live-values)).

Validation checks the whole expression, not just the characters: it must be well-formed, and every `$`/`@` reference must name a real [Resource](/reference/entities.html#resource), [Measure](/reference/entities.html#measure), or built-in value. A problem is reported as `INVALID_DICE`. At runtime, a broken roll logs an error and returns `0`.

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

> Warning: Because addition happens before subtraction, `10-2+3` means `10-(2+3)` = `5`, not `11`. Put subtractions last (`10+3-2`) or use parentheses (`(10-2)+3`).

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

These entity fields are dice fields:

| Entity | Field | Meaning |
| --- | --- | --- |
| `Measure` | `expression` | The value calculated fresh each time the [measure](/manual/guides/add-resources-and-measures.html) is read. |
| `Resource` | `min`, `max` | The bounds, re-read whenever the value changes (so `max` can be a measure, like `$maxHealthMeasure`). |
| [`Condition`](/reference/entities.html#condition) | `left`, `right` | The two sides of a comparison. |
| [`Timer`](/reference/entities.html#timer) | `total` | How many times the timer runs (`0` = forever). |
| `Timer` | `interval` | The seconds between runs. |
| [`Sound`](/reference/entities.html#sound) | `scale` | The audio pitch, as a percentage of normal (see below). |

Many action parameters are dice too, such as `expression` on the resource actions and `radius` and `distance` on area and movement actions. The [action reference](/reference/actions.html) marks each one. Other numeric-looking fields, such as an actor's `speed`, are plain numbers.

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
    total: '0'
    interval: 1d12
    action: regenAction
```

A `total` of `0` means "repeat forever". The `action` field is a [KeyRef](/manual/getting-started/glossary.html#terms) to an Action entity, not a dice field. See [Triggers and timers](/manual/guides/triggers-and-timers.html) for the full timer lifecycle.

<a id="reading-live-values"></a>
## Reading live values

An expression can read an actor's current values:

- `$key` reads the **caller** (the actor running the action, or the owner of the measure).
- `@key` reads the **target** (the caller's current target).

`key` can be any Resource or Measure key, or a built-in value such as `distance_to_target` or `has_target`. For example, a damage parameter of `$attackPowerMeasure+1d6` adds the attacker's attack power to a die roll, and a measure of `$level*5+20` grows with level.

Measures may hold fractions internally (`$speed*100` works); the final result is rounded to a whole number.

> Warning: In a Resource's `min` or `max`, `$` means the actor that owns the resource. Never use `@` there: the bound would change whenever the owner switches targets.

[Dialog](/reference/entities.html#dialog) text uses the same markers, wrapped in double braces:

{% raw %}
```
HP: {{@health}} / Your luck: {{$luck}}
```
{% endraw %}

> Note: Markers behave defensively. A key that names no resource or measure resolves to `0`, and a missing actor resolves its marker to `0`.

## See also

- [Validation](/manual/core-concepts/validation.html)
- [Add resources and measures](/manual/guides/add-resources-and-measures.html)
- [Triggers and timers](/manual/guides/triggers-and-timers.html)
- [Audio and backgrounds](/manual/guides/audio-and-backgrounds.html)
- [Conditions and logic](/manual/guides/conditions-and-logic.html)
- [Menus, dialogs, and waypoints](/manual/guides/menus-dialogs-waypoints.html)
- [Action function reference](/reference/actions.html)
