# Updates

Release notes for Isometry, newest first.

> Adding a release: copy the most recent `## vX.Y — date` block to the top and fill in
> grouped bullets in plain language.

## v{{ version }} — Open beta

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
