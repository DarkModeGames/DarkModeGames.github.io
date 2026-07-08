# Add Resources and Measures

This page explains the two ways an actor carries a number: a **Resource**, a stored value the game tracks and changes over time, and a **Measure**, a value the engine computes fresh from a [dice expression](/manual/getting-started/glossary.html#terms) each time it is read. You will learn the fields of each entity, how an actor wires them up, and how to choose between them.

## Prerequisites

This page assumes you have read:

- [Create an Actor](/manual/guides/create-an-actor.html), since resources and measures live on actors.
- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html), since actors point at resources and measures by key.
- [Dice Expressions](/manual/core-concepts/dice-expressions.html), since a measure's value comes from one.

## Resources versus measures

A [Resource](/manual/getting-started/glossary.html#terms) is a trackable numeric value that the engine stores on each actor and changes as the game runs. Health, mana, and gold are resources: they have a current value, they go up and down, and that value persists between reads.

A [Measure](/manual/getting-started/glossary.html#terms) is a value computed on demand from a dice expression. It is not stored. Every time the engine reads a measure it rolls the expression again, so a measure can be different each time and can be random.

Choose a Resource when the value is owned and mutated: a pool that drains, fills, or transfers. Choose a Measure when the value is derived or rolled in the moment and you do not need to remember it, such as a luck roll.

> Note: Every field on both entities is optional. Provide only the ones your campaign needs.

## Define resources

Resources are declared under a top-level `Resource` block, keyed by [entity key](/manual/getting-started/glossary.html#terms). The example defines three. The following is the example `health` resource, the fullest of the three.

```yaml
Resource:
  health:
    name: Health
    default: 20
    min: 0
    max: 20
    icon: /assets/icons/health.png
    description: Hit points. Reach 0 and you're done.
    menu: healthMenu
```

Each field, in plain terms:

- `name` — the display name shown in the UI, here `"Health"`.
- `default` — the value the actor starts with. The hero starts at `20` health.
- `min` — the lowest the value may reach. All three example resources use `0`.
- `max` — the highest the value may reach. Health caps at `20`.
- `icon` — a path to an icon image for the UI.
- `description` — tooltip text describing the resource.
- `menu` — a [KeyRef](/manual/getting-started/glossary.html#terms) to a Menu entity for interaction options. See [Menus, dialogs, and waypoints](/manual/guides/menus-dialogs-waypoints.html).
- `reveal` — a visibility threshold. The resource is shown only when its value is at least `reveal`; `0` (the default) means always visible.

> Note: Write the clean YAML keys `name`, `min`, and `max`. The trailing-underscore forms (`name_`, `min_`, `max_`) are internal to the engine and never appear in a campaign file.

The other two example resources are leaner. `mana` defaults to `10` with a `max` of `10`; `gold` defaults to `5` with a `max` of `999`. Neither sets a `menu`.

```yaml
Resource:
  mana:
    name: Mana
    default: 10
    min: 0
    max: 10
    icon: /assets/icons/sun.png
    description: Magical energy for casting skills.
  gold:
    name: Gold
    default: 5
    min: 0
    max: 999
    icon: /assets/icons/sun.png
    description: Currency for trading with merchants.
```

## Define measures

Measures are declared under a top-level `Measure` block. The example defines one, `luckMeasure`.

```yaml
Measure:
  luckMeasure:
    expression: 1d20
    icon: /assets/icons/sun.png
    public: false
    private: true
    reveal: 0
```

Each field:

- `expression` — the [dice expression](/manual/getting-started/glossary.html#terms) the engine rolls to produce the value. The validator checks it as dice notation, so `"1d20"` is valid but an arbitrary string is not.
- `icon` — a path to an icon image for the UI.
- `public` — when `true`, the measure appears on the target focus [plate](/manual/getting-started/glossary.html#terms) shown when another actor views this one. The example leaves it `false`.
- `private` — when `true`, the measure appears on the actor's own data plate. The example sets it `true`.
- `reveal` — a visibility threshold, the same as on a resource. `0` means always visible.
- `menu` — a KeyRef to a Menu entity. Omitted in the example.

> Note: Measure visibility lives on the measure itself, through its own `public` and `private` booleans. Resource visibility works differently and lives on the actor, as the next section shows.

## Wire them to an actor

An actor lists the resources and measures it carries through two [KeyRefArray](/manual/getting-started/glossary.html#terms) fields, `resources` and `measures`. Resource visibility is set by two more KeyRefArrays on the actor, `public` and `private`, both pointing at resources. The hero ties all of this together.

```yaml
Actor:
  heroActor:
    name: Hero
    resources:
    - health
    - mana
    - gold
    measures:
    - luckMeasure
    public:
    - health
    private:
    - health
    - mana
    - gold
```

This actor has three resources and one measure. Its `public` array means only `health` is shown on the target focus plate when another actor views the hero. Its `private` array means all three resources are shown on the hero's own data plate.

Note the split: a resource's visibility comes from the actor's `public` and `private` arrays, while a measure's visibility comes from the measure's own `public` and `private` booleans. For the full actor field list, see [Create an Actor](/manual/guides/create-an-actor.html).

> Tip: Put a resource in `private` but not `public` to let an actor see its own value while hiding it from others, as the example does for `mana` and `gold`.

## Change a resource with actions

You do not edit a resource value directly; you change it by running an [action](/manual/getting-started/glossary.html#terms). The action functions that move resources are `plus_resource_self`, `plus_resource_target`, `minus_resource_self`, `minus_resource_target`, `set_resource_self`, `set_resource_target`, and `transfer_resource`.

Each takes two [parameters](/manual/getting-started/glossary.html#terms): `resource`, a KeyRef to the Resource to change, and `expression`, a dice expression for the amount. The new value is capped at the resource's own `min` and `max`, so a `plus` that would exceed `max` stops at the cap.

An action lists its parameters as a flat array of KeyRefs, with the Parameter entities defined in a sibling top-level `Parameter` block. The shape looks like this.

```yaml
Action:
  healHealth:
    do: plus_resource_self
    parameters:
    - pResource
    - pAmount
Parameter:
  pResource:
    key: resource
    value: health
  pAmount:
    key: expression
    value: 1d6
```

Measures take no such functions: there is nothing to change, because the engine rolls the expression each time it reads the value. For the full mechanics of actions and parameters, see [Define Actions](/manual/guides/define-actions.html).

## Show a value on a dialog

Dialog text can print a live resource or measure value with templating tokens. The `@` token reads the target actor; the `$` token reads the caller. The names after the token are entity keys.

{% raw %}
```
HP: {{@health}} / Luck: {{$luckMeasure}}
```
{% endraw %}

Here `{% raw %}{{@health}}{% endraw %}` prints the target's stored `health` resource, and `{% raw %}{{$luckMeasure}}{% endraw %}` rolls and prints the caller's `luckMeasure`. Use these tokens only inside a Dialog entity's `text` field; see [Menus, dialogs, and waypoints](/manual/guides/menus-dialogs-waypoints.html).

## See also

- [Create an Actor](/manual/guides/create-an-actor.html)
- [Dice Expressions](/manual/core-concepts/dice-expressions.html)
- [Define Actions](/manual/guides/define-actions.html)
- [Triggers and Timers](/manual/guides/triggers-and-timers.html)
- [Menus, dialogs, and waypoints](/manual/guides/menus-dialogs-waypoints.html)
