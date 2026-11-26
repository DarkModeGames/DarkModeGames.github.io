# Test your campaign with UI scripts

Clicking through your campaign after every change gets old fast, and it misses things. A **UI script** plays the
game for you: it presses keys, clicks buttons and characters, and checks what the screen shows. It prints a PASS
or FAIL line for every check and exits with 0 (all passed) or 1, so you can run it after each change, or from a
script, and know at once if something broke.

## A first script

Save this as `tests/smoke.ui` beside your campaign:

```text
# Open the Quests page and check our first quest is listed.
ready 90                      # wait (up to 90 s) until the player is in the world
wait 2
press toggle_quests_view      # the Quests key, as the player has it bound
wait 1
expect_state MENU_QUESTS
expect_text "The Wolf Hunt"
press toggle_quests_view
expect_state GAMEPLAY
```

Run it with your campaign hosted, headless:

```bash
Isometry --headless --campaign=MyCampaign --network=host --port=5000 \
  --username=tester --secret=tester --dir=/path/to/folder/with/the/zip \
  --ui-script=/path/to/tests/smoke.ui
```

The output ends with `UI-SCRIPT DONE: 3 passed, 0 failed`.

## Seeing what happened

Swap `--headless` for Movie Maker to record every frame while the script runs:

```bash
Isometry --campaign=MyCampaign --network=host ... --ui-script=tests/smoke.ui \
  --write-movie /tmp/run/frame.png --fixed-fps 10 --resolution=1280x720
```

`mark "<text>"` lines print the frame number they happened on, so you can find the right image.

## What a script can do

- **Input:** `press` and `hold` a bound action (`press action_1`, `hold show_outlines 2`), `key` a raw key,
  `click "<text>"` a button or dialog reply by its words, `click_actor` a character, `click_near_player` a spot,
  `type "<text>"` into a text box.
- **Checks:** `expect_state`, `expect_text` / `expect_no_text`, `expect_visible` / `expect_hidden`,
  `expect_target`, `expect_map`, `expect_distance`, and `expect <dice> <op> <dice>` over the player's own resources
  (`expect $goldResource >= 10`).

The full list is in the [CLI reference](/reference/cli.html#--ui-scriptfile).

## Tips

- Use a test username (and its own saves) so a script never touches a real character. On macOS and Linux you can
  also run with `HOME=/tmp/testhome` to keep the test's logs and saves apart.
- Scripts run only when hosting (`--network=host`): they drive your own player.
- Start each script with `ready`: loading takes a few seconds, and input sent during the loading screen is lost.
- A new character may see a welcome dialog first; `key escape` closes it.
- Keep scripts short and focused (one feature each). They are quick to run, so run them often.
