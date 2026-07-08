# Menus, Dialogs, and Waypoints

This page covers the data entities that drive interface panels and world markers: Menu, Dialog, Waypoint, Group, and Deployment. You use them to give actors interaction options, show the player text, mark travel destinations, color factions, and place actors on a map when the campaign starts.

## Introduction

The entities on this page are pure-data records, like every other [entity](/manual/getting-started/glossary.html#terms) in Isometry. They hold no game logic. Instead, they describe what the player sees and how a campaign is laid out:

- A **Menu** lists the [actions](/manual/guides/define-actions.html) an actor offers when the player interacts with it.
- A **Dialog** is a titled text panel that can show live actor values.
- A **Waypoint** marks a named, discoverable location on a map.
- A **Group** is a faction with a name and an outline color.
- A **Deployment** places an actor on the map at the start of play.

All examples come from an example campaign.

## Prerequisites

Before you read this page, you should be comfortable with:

- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html) — how entities reference each other by key.
- [Create an actor](/manual/guides/create-an-actor.html) — the actors that menus, deployments, and dialogs act on.
- [Build a map](/manual/guides/build-a-map.html) — the maps that waypoints and deployments sit on.

## Menus: actions on interaction

A Menu is a named list of [Action](/manual/getting-started/glossary.html#terms) keys. When the player interacts with a target (the Enter key by default), the engine opens an **interaction panel** in the bottom-center of the screen, replacing the skill and resource blocks until the interaction ends. The panel shows the actor's `dialog` text (if it has one) above its menu options laid out as a 2×2 grid (paginated beyond four). The player moves the selection with the arrow keys and confirms with Enter; Esc closes the interaction.

The `heroMenu` offers four actions: two interactions and two dialogs.

```yaml
Menu:
  heroMenu:
    name: Actions
    actions:
    - inspectAction
    - greetAction
    - showWelcomeDialog
    - showStatusDialog
```

The `name` field is the human-readable title shown above the list. The `actions` field is a [KeyRefArray](/manual/getting-started/glossary.html#terms) of Action keys, resolved at load time. Both fields are optional.

> Note: Each entry in `actions` must match an Action entity key. See [Define actions](/manual/guides/define-actions.html) for how those actions are written.

The example defines four menus: `heroMenu`, `guardMenu`, `merchantMenu`, and `healthMenu`. Each one tailors the action list to the actor that uses it.

### Branching: dialog trees and combat menus

Because an option just runs an Action, and an Action can `open_dialog` (swap the panel's text), `open_menu` (swap the options), or `close_interaction` (leave), you can build decision trees and Pokémon-style combat menus without code. Each node is the text + options currently shown; an option's action moves to the next node in place. The guard shows this: it has a `dialog` (`guardGreeting`) and a `guardMenu` whose options ask about town, threaten (each swaps the text via `open_dialog`), inspect, or leave (`close_interaction`).

## Dialogs: text panels

A [dialog](/manual/getting-started/glossary.html#terms) is a titled text panel shown to the player. You use dialogs for dialogue, lore, instructions, or status readouts. A Dialog has two fields, `title` and `text`, and both are required.

The `welcomeDialog` is plain text with no templating.

```yaml
Dialog:
  welcomeDialog:
    title: Welcome to Isometry
    text: This is the example campaign. It showcases all entity types available in the Isometry engine.
      Explore the map, interact with NPCs, and try your skills.
```

The `title` shows prominently at the top of the panel; the `text` is the body. Long body text is split across pages automatically, so you do not need to manage page breaks yourself.

### Showing live actor values with templating

Dialog body text supports templating tokens that read live values from actors at the moment the dialog is shown. There are two markers:

- `@` reads the **target** actor — the actor being viewed.
- `$` reads the **caller** actor — the actor whose dialog is open, that is, the one acting.

The two markers are not interchangeable. Each token is evaluated through the [dice expression](/manual/core-concepts/dice-expressions.html) engine, so it can reference any of the actor's [resources](/manual/guides/add-resources-and-measures.html) or measures.

The `statusDialog` combines both markers to report the target's health and mana alongside the caller's gold.

{% raw %}
```yaml
Dialog:
  statusDialog:
    title: Status Report
    text: 'Health: {{@health}} / 20

      Mana: {{@mana}} / 10

      Gold: {{$gold}}'
```
{% endraw %}

Here `{% raw %}{{@health}}{% endraw %}` and `{% raw %}{{@mana}}{% endraw %}` show the target actor's `health` and `mana` resources, while `{% raw %}{{$gold}}{% endraw %}` shows the caller's `gold` resource. The literal text around each token, such as `/ 20`, is shown unchanged. The caller and target are bound when the dialog is opened, typically by an action such as `showStatusDialog` listed in `heroMenu`.

> Tip: Use templating tokens only inside a Dialog's `text` field. They work for both resources and measures.

## Waypoints: map markers

A Waypoint is a named, discoverable location on a [map](/manual/guides/build-a-map.html). It gives the player a marker to recognize and travel toward. Every Waypoint field is optional.

The example defines one waypoint, `townCenter`.

```yaml
Waypoint:
  townCenter:
    name: Town Center
    location: townCenterPos
    icon: /assets/icons/sun.png
    map: demoMap
    description: The center of the example town.
Vector:
  townCenterPos:
    x: 300
    y: 200
```

The fields are:

- `name` — the human-readable label shown in the interface.
- `location` — a [KeyRef](/manual/getting-started/glossary.html#terms) to a Vector entity that holds the position (`x`, `y`).
- `icon` — the image path used for the marker on the map and in the interface.
- `map` — a KeyRef to the Map entity the waypoint belongs to.
- `menu` — an optional KeyRef to a Menu entity for interaction options.
- `description` — text shown in the interface.

The example co-locates the backing Vector, `townCenterPos`, in the same file. This is the common pattern: keep a waypoint and its position together so they are easy to read and edit. The `townCenter` omits the optional `menu` field, so it offers no interaction options.

## Groups: factions and color

A Group is a faction or team. The engine uses a group's color to outline its actors and to identify them in the interface, so players can tell allies from enemies at a glance. Both fields are optional.

The example defines three groups.

```yaml
Group:
  playerGroup:
    name: Players
    color: '#00FF00'
  enemyGroup:
    name: Enemies
    color: '#FF0000'
  neutralGroup:
    name: Neutral
    color: '#FFFF00'
```

The `name` field is the faction's label. The `color` field is a hex color string, such as `#00FF00` for green. You assign a group to an actor through the actor's group field; see [Create an actor](/manual/guides/create-an-actor.html).

## Deployments: initial placement

A Deployment defines an actor's initial placement when a campaign starts. It pairs an actor with a spawn position. Both fields are optional.

The example defines three deployments, one per non-player actor.

```yaml
Deployment:
  guardDeploy:
    location: guardPos
    actor: guardActor
Vector:
  guardPos:
    x: 128
    y: -48
```

The `location` field is a KeyRef to a Vector entity holding the spawn position (`x`, `y`). The `actor` field is a KeyRef to the Actor entity to place there. As with waypoints, the example co-locates the spawn Vector, `guardPos`, in the same file.

> Note: A Deployment references only a position and an actor. The player actor's own starting placement comes from the [Main](/manual/getting-started/glossary.html#terms) entity, not from a Deployment.

## See also

- [Entities and KeyRefs](/manual/core-concepts/entities-and-keyrefs.html)
- [Dice expressions](/manual/core-concepts/dice-expressions.html)
- [Create an actor](/manual/guides/create-an-actor.html)
- [Add resources and measures](/manual/guides/add-resources-and-measures.html)
- [Define actions](/manual/guides/define-actions.html)
- [Build a map](/manual/guides/build-a-map.html)
- [Action function reference](/reference/actions.html)
