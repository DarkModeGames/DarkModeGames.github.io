# Help pathfinding round corners with Nav points

**Problem.** Walkers take odd routes through a narrow doorway or a custom-shaped yard, or get stuck at a gap.

**Pattern.** Walkers plan paths across the map's walkable tiles and bend their path at corners. The corners of walls
and obstacles are found automatically. Where that isn't enough (a doorway, a gap between props, a twisty path), add
a [`Nav`](/reference/entities.html#nav) point: a [`Vector`](/reference/entities.html#vector) on a given map where
walkers may also bend their path.

```yaml
Vector:
  townSpawnVector:
    x: 384
    y: 8
  smithyDoorVector:
    x: 416
    y: 40
  yardGapVector:
    x: 352
    y: 72

Nav:
  smithyDoorNav:
    map: townMap
    location: smithyDoorVector
  yardGapNav:
    map: townMap
    location: yardGapVector
```

**Entities used.** [`Nav`](/reference/entities.html#nav), [`Vector`](/reference/entities.html#vector)

**Gotchas.**

- Place a Nav point in the **middle** of the opening, on walkable ground. A point off the walkable area or inside
  an obstacle is skipped with a warning when the map is built.
- A point too tight for a large body is left out of that body's paths, so big monsters may still not fit.
- Add Nav points only where you see a problem. Most maps need none.
- Test with a walker that crosses the spot (a test actor with `set_destination_self` on a [Timer](/reference/entities.html#timer)) and watch it.
- Big open fields next to roads can also cause long detours. Breaking them up with props helps.
