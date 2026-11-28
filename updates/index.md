# Updates

Release notes for Isometry, newest first.

> Adding a release: copy the most recent `## vX.Y — date` block to the top and fill in
> grouped bullets in plain language.

## v0.407 — Character page and HUD fixes (still beta)

A small update on top of the 0.405 release candidate.

### For players
- **One target plate, always.** An old target's plate could stay at the top of the screen beside the new one
  after you changed form or map, or talked to someone. Now there is only ever one.
- **Character page items are icons.** Pick a slot and what fits it shows as a grid of icons. Rest the mouse on an
  item (or on a slot) to see its name and what it does. The item you're wearing has a gold border.
- **New focus corner icon.** The corners show a small figure in crosshairs instead of a pin.

### For campaign makers
- **`paired` slots.** A Slot with `paired: true` says that `worn[i]` goes with `choices[i]`, so the Character page
  can mark the worn item. Both lists must be the same length. See the [Slot reference](/reference/entities.html#slot)
  and the [Character page recipe](/recipes/character-page.html). Campaigns that use it need 0.407 or newer.

## v0.405 — Release candidate (still beta)

**Isometry 0.405 is a release candidate.** It is still a beta, but everything below is finished, tested (1,990
automated tests, all passing) and played in a full campaign, **Aurelia**. This is
everything since 0.401.

### For players
- **Quests page (J).** Every quest you've started, what to do now, and the steps you've done. Tick
  *Show completed* to see finished ones.
- **Character page (U).** What you wear in each slot and your key stats. Pick a slot to swap gear.
- **Talent trees (P).** Spend points from the bottom row up; each talent shows what it does.
- **Auto-attack.** Right-click an enemy and your character keeps swinging until it dies, you change target or you
  walk away. Casting a skill in between doesn't reset your swing.
- **Tab cycles targets**, **E** picks the nearest, and **hold Alt** (or the new eye button) to outline everything
  you can target, as in Diablo.
- **Focus corners (R T F G)** keep a character's plate in a corner of the screen. Each corner now shows a pin:
  click it to pin your target, right-click to clear it, and press **H** to clear them all. A pinned character who
  leaves the world now leaves its corner too.
- **Click the icon on any plate** to interact with that character.
- **Skill tooltips show real numbers**, worked out from your own stats ("12–17 damage"), and skills you can't use
  right now (no resource, on cooldown, out of range) are darkened on the hotbar.
- **Conversations are a list of spoken replies**, with stat checks such as `[CHA 12]` shown up front.
- Minions: a strip under your target shows them with their orders (F1–F4).
- Smaller things: a walk/run toggle (N), a follow key (V), dimmer HUD icons, five HUD buttons a side, an Interact
  flag, an on-screen keyboard for gamepads, long names that fit their plates, and campaign credits in the System
  menu.

### For campaign creators
- **New entities:** `Quest` and `Step` (the Quests page), `Sheet`, `Slot` and `Stat` (the Character page),
  `Tree` and `Talent` (talent trees) and `Nav` (extra pathfinding points).
- **New fields:** `Actor.auto_attack` / `auto_attack_interval` / `auto_attack_range`, `Actor.on_interact`,
  `Actor.icon` and `Group.icon` (shown on plates), `Deployment.respawn` (monsters come back), `Skill.ready` and
  `Skill.in_range` (hotbar readiness), `Condition.all`, `Menu.hide_unavailable`, `Resource.combat_text` and
  `tags`, `Sound.volume` and `range`, and `Main.credits`.
- **New actions:** `start_auto_attack`, `stop_auto_attack_self`, `play_sound_self` / `play_sound_target`,
  `move_home_self`, minion commands (`follow_master` with formations, `each_minion`, `target_master`), and
  `echo_*` gained a public/private audience, a colour and a duration.
- Skill descriptions accept {% raw %}`{{range: <dice>}}` and `{{dice: <dice>}}`{% endraw %}.
- **Test your campaign with scripts:** `--ui-script` plays keys and clicks and checks the screen, with PASS/FAIL
  lines. See [Test your campaign with UI scripts](/manual/guides/test-with-ui-scripts.html).
- The validator now rejects a map without a tilemap, a tilemap without a tileset, and an incomplete tileset.
- **Seven new recipes:** [quest log](/recipes/quest-log.html), [talent trees](/recipes/talent-trees.html),
  [character page](/recipes/character-page.html), [auto-attack](/recipes/auto-attack.html),
  [monsters that come back](/recipes/respawning-monsters.html),
  [skill tooltips and readiness](/recipes/skill-tooltips-and-readiness.html) and
  [credits](/recipes/campaign-credits.html). Every recipe's example is validated against this release.

### Multiplayer
- Tested end to end with real processes: host alone, host with two clients, a dedicated server with two clients,
  a client quitting and rejoining, and a client crashing mid-fight (the others carry on and the lost player is
  removed).
- Fixed: late joiners now see current values; appearance changes reach every player; a change of form no longer
  leaves ghost bodies or hides the world; discovered waypoints survive a respawn; any-peer messages check who sent
  them.

### Performance
Measured on an Apple M5 with the full Aurelia campaign (about 1,000 actors over three maps), per frame at 60 fps
(the budget is 16.7 ms): idle 0.4 ms, walking the moor 2.5 ms, a four-player fight 4.5 ms. On average a host stays
under budget with 8 players in one fight (8.7 ms) and with 500 monsters awake at once (14 ms).

### Isometry Editor
The editor now matches this release: every new entity and field appears in the inspector, options with a fixed
set of words (such as a message's audience) are drop-downs, and `Nav` points can be placed on the map painter.

## v0.401 — Open beta

**Isometry is now in open beta.** This release covers roughly three months of work bringing
the framework to a public, playable state — gameplay, multiplayer, authoring, and a brand-new
documentation site.

### Gameplay &amp; controls
- Click to select an actor, click the ground to move, and click again to act — mouse-driven
  play throughout.
- Movement keys and the facing keys now update an actor's bearing and heading, so you face
  the way you move or aim.
- The cursor reacts to what's under it (move, target, interact).
- Actor-to-actor interactions now resolve edge-to-edge ("base to base") instead of from
  centers, so reach feels right.
- You can move while a dialog or menu is open; talking to someone and then walking away (or
  clicking empty ground) closes the conversation cleanly.
- **Flying actors**: actors can be set to fly over walls and obstacles, stopped only by a
  new impassable **boundary** terrain — with fly/land actions and a flying state you can
  check in your own logic.
- **Null actor**: leave a campaign's starting actor unset and players begin as an invisible,
  immovable spectator — ideal for letting players pick their hero. See
  [Campaign structure](/manual/core-concepts/campaign-structure.html#the-main-entity).

### Camera &amp; map view
- A pop-out **map view** that opens from the bottom of the screen with its own zoom, pan
  (movement/look keys), and recenter-on-player.
- The world camera remembers your zoom level between sessions.
- Left-click on the map view centers the camera there.

### Skills &amp; actions
- Click to use skills from the hotbar.
- A skill's **charge** time can now affect action dice (e.g. scale damage by how long you held
  it), and the cast animation holds on its final frame while charging.
- Action parameters accept **dice expressions**, and resource minimums/maximums can be dice
  too — so values can roll at runtime.
- New **particle-effect** actions to attach and remove cosmetic effects on actors.
- Movement tracks accept ordered, numbered waypoints; teleport resumes travel to your
  original destination after the jump.

### Multiplayer
- Teleports and facing now reach and replicate to the owning client correctly, so what you
  do shows up the same for everyone.
- A first-login hook and per-player login statistics.
- Stability fixes to actors updating correctly on everyone's screen, and joining a game while
  it's still loading.

### Interface
- Menus (resources, skills, system, and more) now have outlined, keyboard-navigable slots, a
  dedicated description panel, and a theme-aware selection outline; keybind and gamepad
  screens are fully keyboard-navigable.
- Focus plates show an actor's group name in the group's color.
- The system info screen scrolls with the arrow keys.
- Reworked the validation-error screen and added syntax highlighting to the error log.
- Resource icons fit their slots, a placeholder shows for missing icons, and various spacing
  and theme-opacity polish.
- Timed-hold keybinds with a configurable hold time.

### Campaign authoring
- Stronger validation: full dice-notation checking and per-action parameter checking, so
  mistakes surface before the game runs.
- Actions may omit their function to be animation-only or chain-only.
- Bad or missing campaign names now fail gracefully instead of crashing.
- Tiles can carry isolated navigation, and unused optional fields dissolve away on load.

### Platform
- Engine upgraded for performance and stability.
- Adopted a `major.commit-count` version scheme.
- All documentation moved to this website; the player log is written to `log.txt`.
