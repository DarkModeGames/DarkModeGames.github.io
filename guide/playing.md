# Playing Isometry

## Controls

These are the default keybinds. Every binding can be remapped (keyboard/mouse and
gamepad) from the options menu.

### Movement

- **Right Click** — set a destination; your character paths to it around
  obstacles.
- **W / A / S / D** — move directly in a direction (hold to keep moving).
- **Arrow Keys** — change your facing direction (bearing).

### Camera

- **L** — toggle camera lock on your character.
- **Space** — recenter the camera on your character (hold to keep it centered).
- **`=`** — zoom in.
- **`-`** — zoom out.

### Targeting

- **Tab** — select the nearest target.
- **`` ` `` (Grave)** — move to the current target.
- **E** — cycle to the next target.
- **Q** — cycle to the previous target.
- **C** — cycle to the next target group.
- **Z** — cycle to the previous target group.
- **X** — target yourself.
- **Esc** — clear the current target.

### Skills

- **1 – 9** — activate the skills in hotbar slots 1 through 9.
- Skills may require a target to be selected.
- Some skills are instant; others have casting animations.
- Hold a skill key to charge it (if the skill supports charging).

### Menus & Views

- **Enter** — open the selection menu for the current target.
- **Backspace** — open the global menu.
- **M** — toggle the map view.
- **O** — toggle the resources view.
- **K** — toggle the skills view.
- **Esc** — close the current menu.

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

1. Launch with `--network=host`, or click **HOST** in the launcher.
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
