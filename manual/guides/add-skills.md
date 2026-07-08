# Add Skills

Skills are the abilities a player triggers from the hotbar — a punch, a heal, a charged shot. This page shows how to define a Skill, wire it to the [actions](/manual/guides/define-actions.html) it runs, list it on an actor, and build a skill that charges while you hold the button.

## Introduction

A **Skill** is an [entity](/manual/getting-started/glossary.html#terms) that connects a hotbar button to an [Action](/manual/getting-started/glossary.html#terms). When the player presses the button, the skill runs one action; when they release it, it can run another. Each skill carries the display details the interface needs — a name and an icon — and optional fields for charged or animated abilities.

Skills are how players act on the world. An actor with no skills can move but cannot attack, heal, or use any other ability. The hotbar holds at most nine skills, mapped to the keys `Q W E R T Y U I O` by default.

## Prerequisites

This page assumes you have already:

- Created an actor — see [Create an Actor](/manual/guides/create-an-actor.html).
- Defined the actions a skill will run — see [Define Actions](/manual/guides/define-actions.html).

## What a Skill points at

A Skill does not contain game logic. It points at one or two [KeyRefs](/manual/getting-started/glossary.html#terms) — references to Action entities by their key:

- `start` runs when the player presses the skill button.
- `end` runs when the player releases the button.

Either may be omitted. The example skills set both to the same action, so the ability fires that one action on press and again on release.

The remaining fields describe the skill to the player and the engine:

| Field | Type | Required | Default | Purpose |
|-------|------|----------|---------|---------|
| `name` | String | No | — | Name shown in the interface. |
| `start` | KeyRef → Action | No | — | Action run when the button is pressed. |
| `end` | KeyRef → Action | No | — | Action run when the button is released. |
| `icon` | String | No | — | Path to the icon shown on the hotbar. |
| `description` | String | No | `""` | Tooltip text. |
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
    end: attackAction
    icon: /assets/icons/sword.png
```

`attackSkill` shows as "Punch" with the sword icon. Pressing or releasing its hotbar key runs `attackAction`, an Action defined elsewhere in the campaign. The icon path starts with a leading slash and points at a file inside the campaign's assets.

> Note: The `start` and `end` values must be the keys of real Action entities. Validation rejects a Skill whose `start` or `end` points at a missing action. See [Define Actions](/manual/guides/define-actions.html) for how those actions are written.

The other skill, `healSkill`, has the same shape — it shows as "Heal" with a health icon and runs `conditionalHealAction` on both press and release.

```yaml
Skill:
  healSkill:
    name: Heal
    start: conditionalHealAction
    end: conditionalHealAction
    icon: /assets/icons/health.png
```

A skill can run different actions on press and release — set `start` and `end` to different action keys. The example keeps them identical because a punch or a heal is a single instant action; there is no separate release behavior to model.

## Add Skills to an actor

A skill does nothing until an actor lists it. An actor references its skills through the `skills` field: an array of Skill keys, in hotbar order. The hero declares both skills.

```yaml
Actor:
  heroActor:
    name: Hero
    base: 8
    speed: 75
    skills:
    - attackSkill
    - healSkill
```

The order matters: `attackSkill` takes the first hotbar slot (`Q` by default) and `healSkill` the second (`W`). See [Create an Actor](/manual/guides/create-an-actor.html) for the actor's other fields, such as `base` and `speed`.

> Warning: An actor may list at most nine skills. The tenth and beyond are ignored — both the campaign validator and the runtime cap the hotbar at nine slots.

The default hotbar keys, in order, are `Q W E R T Y U I O`. They map to the inputs `action_1` through `action_9`. You can rebind them; see the [CLI reference](/reference/cli.html) for how keybinds are configured.

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

## Show a skill's values in a dialog

A [dialog](/manual/getting-started/glossary.html#terms) can display live actor values next to a skill's effect using templating tokens. Use these only inside a Dialog entity's `text` field.

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
