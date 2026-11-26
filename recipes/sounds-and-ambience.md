# Sounds on animations and map ambience

**Problem.** A swing should make a sound, and the moor should have wind blowing in the background.

**Pattern.** A [`Sound`](/reference/entities.html#sound) is an audio file with `loop` and a pitch `scale`.

- `scale` is the **pitch**, as a dice expression divided by 100: `'100'` is normal, `'90+1d20'` varies it a little
  on each play. It does not change loudness.
- Attach a one-shot Sound to an [`Animation`](/reference/entities.html#animation) with its `sound` field. It plays
  each time that clip starts.
- List looping Sounds in a [`Map`](/reference/entities.html#map)'s `audio`. They play while a player is on that
  map.

```yaml
Sound:
  swingSound:
    source: assets/sound.mp3
    loop: false
    scale: '90+1d20'
  moorWindSound:
    source: assets/sound.mp3
    loop: true
    scale: '100'

Animation:
  heroSwingAnimation:
    N: [0, 1, 2]
    NE: [0, 1, 2]
    E: [0, 1, 2]
    SE: [0, 1, 2]
    S: [0, 1, 2]
    SW: [0, 1, 2]
    W: [0, 1, 2]
    NW: [0, 1, 2]
    loop: false
    sound: swingSound

Vector:
  moorSpawnVector:
    x: 384
    y: 8

Map:
  moorMap:
    name: The Moor
    tilemap: groundTilemap
    spawn: moorSpawnVector
    audio: [moorWindSound]
```

**Entities used.** [`Animation`](/reference/entities.html#animation), [`Map`](/reference/entities.html#map), [`Sound`](/reference/entities.html#sound), [`Vector`](/reference/entities.html#vector)

**Gotchas.**

- Use **MP3** files. WAV files don't load reliably.
- `scale` is re-rolled each time the sound finishes, so a looping sound with a random `scale` changes pitch on each
  repeat. Use a plain `'100'` for ambience.
- Sounds are easy to overdo. Put them on key moments (big attacks, deaths, a door) rather than every footstep.
- A clip with a sound plays to the end even when started as a keyframe.
- Asset paths are relative to the campaign root, as for textures.
