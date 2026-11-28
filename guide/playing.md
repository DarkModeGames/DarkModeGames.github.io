# Playing Isometry

## Controls

These are the default keybinds. Every binding can be remapped (keyboard/mouse and
gamepad) from the options menu.

### Movement

- **Right Click** — set a destination; your character paths to it around
  obstacles.
- **W / A / S / D** — move directly in a direction (hold to keep moving).
- **Arrow Keys** — change your facing direction (bearing).
- **N** — switch between running and walking. Walking is half pace.
- **Flag button** (left-hand HUD buttons, above the eye) — pick up a flag, then left-click to place it. On open
  ground your character walks to the flag. On a character, yours follows them, and talks to them or opens their
  menu on arrival. Right-click or press the button again to put the flag away.

### Camera

- **L** — toggle camera lock on your character.
- **Space** — recenter the camera on your character (hold to keep it centered).
- **`=`** — zoom in.
- **`-`** — zoom out.

### Character

- **U** — open the Character page: what you wear in each slot, and your stats. Pick a slot to see the items you
  carry that fit it, and pick one of those to equip or unequip it.

### Quests

- **J** — open the Quests page: your quests, what to do now, and the steps already done. Tick **Show completed**
  to see finished quests too.

### Targeting

- **Alt (hold)** — outline everything you can target nearby, like Diablo's Alt. On a gamepad: the Share / capture
  button (rebind it in the gamepad menu).
- **Tab** — cycle to the next target (nearest first, then each one in view in turn).
- **`` ` `` (Grave)** — move to the current target.
- **V** — follow the current target. Your character keeps walking after it and waits when close. Press
  **V** again, move, or change target to stop.
- **E** — select the nearest target.
- **Q** — cycle to the previous target.
- **C** — cycle to the next target group.
- **Z** — cycle to the previous target group.
- **X** — target yourself.
- **Esc** — clear the current target.
- **Click the icon on a plate** — interact with that character, as **Enter** does with a target: you walk up to
  talk, trade, loot or fight. Works on the target plate and on focus plates.
- **Show Outlines button** (bottom of the left-hand HUD buttons, an eye) — hold it down with the mouse to outline
  everything you can target, as holding **Alt** does.

### Focus

A focus plate keeps a character's plate in a corner of the screen, so you can watch a friend's health or a
boss's while you target something else. Each corner holds one.

- **R / T / F / G** — the four corners (top left, top right, bottom left, bottom right). Tap with a target to pin
  it there. Tap again later to target the pinned character. Hold to clear the corner.
- **Corner icons** — each corner shows a small figure in crosshairs. Hover it for a reminder. Click it to pin your target there
  (replacing whoever was there); with no target, a click targets the pinned character. Right-click clears it.
  The icons hide with the other HUD buttons (**Show Menu Buttons** in the options menu).
- **H** — clear every focus corner and start fresh.
- A pinned character who leaves the world (dies and despawns, or goes where you can't see them) leaves its
  corner by itself after a couple of seconds.

### Skills

- **1 – 9** — activate the skills in hotbar slots 1 through 9.
- Skills may require a target to be selected.
- Some skills are instant; others have casting animations.
- Hold a skill key to charge it (if the skill supports charging).
- **Mouse-over casting.** With the mouse over a character, a skill key casts on that character instead of your
  target, and your target stays selected: heal a friend under the cursor while keeping the enemy targeted. With
  the mouse over nothing, the skill goes to your target as usual. Clicking a hotbar button always uses your
  target (ctrl+click casts on yourself). Turn it off with **Mouse-over Casting** in the options menu (the
  default is **ON**).

### Menus & Views

- **Enter** — open the selection menu for the current target.
- **Backspace** — open the global menu.
- **M** — toggle the map view.
- **O** — toggle the resources view.
  - **Q / E** (left / right shoulder on a gamepad) — step through the category chips over the list.
  - **/** (**Y** on a gamepad) — search by name. Esc clears the search; a second Esc closes the view.
  - **Page Up / Page Down** (the triggers) — turn the page.
- **K** — toggle the skills view. Pick a skill up (click or accept) and put it on another key to swap them.
- **P** — toggle the talents view: your talent trees, read from the bottom up. Click a talent to put a point in.
- **Minions** (only while you have some): a strip under the target plate lists them with their orders. Click chips to
  pick some (none = all), then Attack, Follow or Stay. **F1 / F2 / F3** give the orders, **F4** picks the next minion,
  **I** hides or shows the strip. On a gamepad, **Back** enters command mode: **D-pad left/right** pick, **X / A / Y**
  give Attack / Follow / Stay; **Back** or **B** leaves it.
- **Esc** — close the current menu.

### Typing with a gamepad

Chat and the resources search can be typed into with a gamepad alone. When one of them takes the cursor and
your last press was on a gamepad, an on-screen keyboard opens at the bottom of the screen (at the top if the
text box is low). It never opens for keyboard or mouse players.

- **D-pad / left stick** — move between keys; rows wrap round at the edges.
- **A** — press the selected key.
- **B** — delete the letter before the cursor; on an empty box, cancel.
- **X** — space.
- **Y** — shift, for one capital letter. The **Caps** key keeps capitals on.
- **Start** — done: chat sends the line, the search returns to the list.
- **Left / right shoulder** — move the cursor.
- **Cancel** puts the text back the way it was. In chat it also closes the chat.

The keys can also be clicked. To turn the keyboard off, set **On-Screen Keyboard** to **OFF** in the options
menu (the default is **AUTO**).

## Gameplay Mechanics

### Resources

Resources are numeric values tracked on your character (health, mana, gold, and
so on). They display on your data plate and can be viewed by right-clicking your
resource bar.

- Resources have minimum and maximum values.
- Resources can regenerate via timers.
- Resources can trigger actions when they change (for example, death at 0
  health).

### Skills & Actions

Skills are bound to the number keys `1`–`9` by default. Pressing a skill key
executes its associated action, which may:

- Deal damage to a target
- Heal yourself
- Move to a location
- Display information
- Trigger conditional logic

### Perception & Vision

Characters have a perception radius. You can only see actors within your
perception range. Actors also have a visibility setting (salience) that affects
how easily you can spot them.

### Groups & Factions

Actors belong to groups (players, enemies, neutral). Group membership affects
targeting and determines the colored outline on characters.

## Multiplayer

### Hosting a Game

1. Launch with `--network=host` (or use your campaign's own launcher, if it has one).
2. Share your IP address and port with players.
3. Players connect with `--network=client` pointing at your address.

For details, see [Hosting & Networking](/guide/hosting.html).

### Joining a Game

1. Get the server's IP and port from the host.
2. Launch with `--network=client --uri=SERVER_IP --port=PORT`.
3. Enter the same campaign name along with your credentials.

### Authentication

{{ product_name }} keeps your password secure and encrypted. Players sign in with
their username and password to join and spawn.

## Troubleshooting

### Campaign won't load

- Verify the `.zip` file exists in your campaign directory.
- Check the campaign name matches (case-sensitive, no `.zip` extension).
- Run with `--log-level=trace` for detailed error output.
- Validate the campaign directly with `--validate=/path/to/campaign.zip`.

### Can't connect to server

- Verify the server is running and the port is open.
- Check firewall settings (the UDP port must be reachable).
- Ensure both client and server use the same campaign version.

### Performance issues

- Reduce the zoom level.
- Play on maps with fewer simultaneously visible actors.
- Check `--log-level=trace` for bottleneck information.
