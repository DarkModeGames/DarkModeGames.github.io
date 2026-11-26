# Add Skills

Skills are the abilities a player triggers from the hotbar — a punch, a heal, a charged shot. This page shows how to define a [Skill](/reference/entities.html#skill), wire it to the [actions](/manual/guides/define-actions.html) it runs, list it on an actor, and build a skill that charges while you hold the button.

## Introduction

A **Skill** is an [entity](/manual/getting-started/glossary.html#terms) that connects a hotbar button to an [Action](/manual/getting-started/glossary.html#terms). When the player presses the button, the skill runs one action; when they release it, it can run another. Each skill carries the display details the interface needs — a name and an icon — and optional fields for charged or animated abilities.

Skills are how players act on the world. An actor with no skills can move but cannot attack, heal, or use any other ability. The hotbar holds at most nine skills, mapped to the number keys `1`–`9` by default.

## Prerequisites

This page assumes you have already:

- Created an actor — see [Create an Actor](/manual/guides/create-an-actor.html).
- Defined the actions a skill will run — see [Define Actions](/manual/guides/define-actions.html).

## What a Skill points at

A Skill does not contain game logic. It points at one or two [KeyRefs](/manual/getting-started/glossary.html#terms) — references to Action entities by their key:

- `start` runs when the player presses the skill button.
- `end` runs when the player releases the button.

Either may be omitted. For an ordinary ability, set only `start`. If you set `start` and `end` to the same action, it runs **twice**: once on press and again on release.

The remaining fields describe the skill to the player and the engine:

| Field | Type | Required | Default | Purpose |
|-------|------|----------|---------|---------|
| `name` | String | No | — | Name shown in the interface. |
| `start` | KeyRef → Action | No | — | Action run when the button is pressed. |
| `end` | KeyRef → Action | No | — | Action run when the button is released. |
| `icon` | String | No | — | Path to the icon shown on the hotbar. |
| `description` | String | No | `""` | Tooltip text. BBCode formatting (`[b]`, `[color=#...]`, `[br]`) shows as formatting. |
| `charge` | Int | No | `0` | Maximum charge for held abilities; `0` means no charging. |

No field is required. In practice you give a skill at least a `name`, an `icon`, and a `start` action so the player can see it and use it.

## Define a Skill

Skill entities live in their own file. The example defines both of its skills together.

The attack skill is the simplest useful shape: a name, an icon, and a single action wired to both press and release.

```yaml
Skill:
  attackSkill:
    name: Punch
    start: attackAction
    icon: /assets/icons/sword.png
    description: A quick jab. Shown as the hotbar tooltip.
```

`attackSkill` shows as "Punch" with the sword icon. Pressing its hotbar key runs `attackAction`, an Action defined elsewhere in the campaign. The icon path starts with a leading slash and points at a file inside the campaign's assets.

> Note: The `start` and `end` values must be the keys of real Action entities. Validation rejects a Skill whose `start` or `end` points at a missing action. See [Define Actions](/manual/guides/define-actions.html) for how those actions are written.

The other skill, `healSkill`, has the same shape — it shows as "Heal" with a health icon and runs `conditionalHealAction` on press.

```yaml
Skill:
  healSkill:
    name: Heal
    start: conditionalHealAction
    icon: /assets/icons/health.png
```

Use `end` only when releasing the button should do something different, as in the charged skill below.

> Tip: To give a skill a cooldown, a mana cost, or a range limit, put an `if` [Condition](/reference/entities.html#condition) on its action and check a [Resource](/reference/entities.html#resource). If the check fails, the action is refused and the player can act again right away. See the cooldown recipe in [Recipes](/recipes/).

## Add Skills to an actor

A skill does nothing until an actor lists it. An actor references its skills through the `skills` field: an array of Skill keys, in hotbar order. The hero declares both skills.

```yaml
Actor:
  heroActor:
    name: Hero
    base: 8
    speed: 1.0
    skills:
    - attackSkill
    - healSkill
```

The order matters: `attackSkill` takes the first hotbar slot (key `1`) and `healSkill` the second (key `2`). See [Create an Actor](/manual/guides/create-an-actor.html) for the actor's other fields, such as `base` and `speed`.

> Warning: An actor may list at most nine skills. The tenth and beyond are ignored — both the campaign validator and the runtime cap the hotbar at nine slots.

The default hotbar keys are `1` through `9`, mapped to the inputs `action_1` through `action_9`. Players can rebind them in the in-game Options menu.

**Who a skill is cast on.** A skill's actions run with the player as caller and, as target, the player's current target, or the player when they ctrl+click the hotbar button. With **mouse-over casting** (an Options setting, on by default), a skill key pressed while the mouse is over a targetable actor casts on that actor instead, and the player's target is left as it was. `@` in the skill's conditions and dice is always the actor it is cast on. For a mouse-over cast, `$distance_to_target`, `$has_target` and `$line_of_sight` also read that actor while the skill's actions run, so a range check like "Out of range" works for it too. A charged or two-part skill's release goes to the same actor as its press.

## Build a charged skill

A **charged skill** builds up power while the player holds its button, then releases it. Set the `charge` field to a number greater than zero to turn charging on.

While the button is held, the charge builds up, capped at the `charge` maximum. Switching to a different skill mid-hold resets the charge to zero. When `charge` is `0` — the default — the skill fires immediately with no build-up.

The following skill reuses the `attackAction` but charges up to a maximum of `10` before releasing.

```yaml
Skill:
  chargedPunch:
    name: Charged Punch
    start: attackAction
    end: attackAction
    icon: /assets/icons/sword.png
    charge: 10
```

Here `start` runs `attackAction` on press to begin the ability, `charge` accumulates while the button is held, and `end` runs `attackAction` again on release. A skill's cast visual comes from its `start`/`end` action's own `animation` field — see [Visuals: Sprites and Animations](/manual/guides/visuals-sprites-animations.html).

> Note: Neither example skill uses `charge`, so the values above are illustrative. The `start` and `end` actions (`attackAction`) are real keys.

> Tip: A charged skill reads best when `start` and `end` run different actions — `start` begins the cast and `end` releases the stored charge. Wire `end` to an action that consumes the built-up charge for a payoff that scales with hold time.

## How long a skill takes

An action's `time` (in seconds) is how long the caster is busy. While a skill's action is running, the player can't start another skill. Triggers and Timers keep running regardless. Leave `time` at `0` for instant abilities.

## Show a skill's values in a dialog

A [dialog](/manual/getting-started/glossary.html#terms) can display live actor values next to a skill's effect using templating tokens. Use these only inside a [Dialog](/reference/entities.html#dialog) entity's `text` field.

{% raw %}
```
HP: {{@health}} / Your luck: {{$luck}}
```
{% endraw %}

`{% raw %}{{@health}}{% endraw %}` reads the target actor and `{% raw %}{{$luck}}{% endraw %}` reads the caller. This is useful for a heal skill whose dialog should show the target's current health.

## See also

- [Create an Actor](/manual/guides/create-an-actor.html)
- [Define Actions](/manual/guides/define-actions.html)
- [Conditions and Logic](/manual/guides/conditions-and-logic.html)
- [Visuals: Sprites and Animations](/manual/guides/visuals-sprites-animations.html)
- [CLI Reference](/reference/cli.html)
