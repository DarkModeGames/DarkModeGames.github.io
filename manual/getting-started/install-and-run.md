# Install and Run

This page shows you how to download the Isometry engine, get it running, and load an example [campaign](/manual/getting-started/glossary.html#terms). It ends with the exact commands you use to confirm a campaign loads.

## Introduction

Isometry is a single program that loads game [campaigns](/manual/getting-started/glossary.html#terms) defined entirely in YAML. There is no separate server to install and nothing else to set up; the program is the whole product. To build campaigns you first need it on your machine, and you need a reliable way to check that a campaign you wrote actually loads.

This page covers downloading the engine, running it, and the headless verification boot you return to throughout the manual.

## Prerequisites

- You have read [What is Isometry](/manual/getting-started/introduction.html) and understand that a campaign is YAML data the engine loads.

## Download Isometry

Isometry ships as a ready-to-run binary from itch.io:

<https://dark-mode-games.itch.io/isometry>

Download the build for your platform and unzip it. The download contains the engine executable and nothing else to install: no separate installer, no runtime, and no extra dependencies. The binary is the whole product.

> Note: Isometry is in open beta, version {{ version }}. itch.io is the only official download source.

## Run the engine

Run the engine executable with command-line flags that name the campaign, the network mode (host or join), and your username and password; see the [CLI reference](/reference/cli.html) and the host boot command below. There is no built-in launcher screen, though a campaign may ship its own launcher that passes these flags for you.

<a id="where-files-go"></a>
### Where files go

Two locations matter, and they are different:

- **Campaigns** go in a folder named `campaigns` next to the engine executable (on macOS, next to `Isometry.app`). To use another folder, pass `--dir=/absolute/path/to/folder`.
- **The log, saves, and keys** go in the engine's data folder:
  - macOS: `~/Library/Application Support/isometry/`
  - Linux: `~/.local/share/isometry/`
  - Windows: `%APPDATA%\isometry\`

Not sure where the engine is looking? Run `isometry --print-paths`. It prints the data folder and every campaign folder it checks, then exits.

Throughout this manual, command examples call the engine as `isometry`; use the actual file name of the executable you downloaded for your platform.

## Load an example campaign

A campaign's [Main](/manual/getting-started/glossary.html#terms) entity is the single required entry point that names the starting map, and optionally the player actor. A minimal `Main.yaml` defines one Main entity:

```yaml
Main:
  demoMain:
    actor: heroActor
    map: demoMap
```

Here `actor` is a [KeyRef](/manual/getting-started/glossary.html#terms) to the `heroActor` entity and `map` is a KeyRef to `demoMap`. The hero actor it points to uses real values you will reuse later:

```yaml
Actor:
  heroActor:
    name: Hero
    speed: 1.0
    base: 8
```

`speed` is the actor's movement rate and `base` is the footprint size in pixels.

To play a campaign, package its folder into a `.zip` (see [Your first campaign](/manual/getting-started/your-first-campaign.html) for packaging), put the zip in the `campaigns` folder (see [Where files go](#where-files-go)), and launch.

## Verify a campaign loads (headless)

When you write your own campaign, you need a fast way to confirm it loads and validates without opening a window. The engine offers two headless checks.

### Quick validate

The fastest check runs the campaign validator and exits with status `0` on success or `1` on failure, without building the world. Point it at a campaign directory or a `.zip`:

```bash
isometry --validate=mycampaign.zip
```

Use this in scripts and quick iteration: a zero exit code means every entity, KeyRef, and dice expression passed validation.

### Full host boot

To exercise the full load-and-validate pipeline, boot in host mode. The `--network` flag selects the multiplayer mode: `none`, `host`, `server`, or `client`. Host mode runs the pipeline you want to exercise.

> Warning: `--network=none` currently loads no campaign. The engine boots but does nothing. To play single-player or boot a campaign, use `--network=host`.

Put the campaign zip in the `campaigns` folder, then run the boot:

```bash
isometry --campaign=demo --network=host --port=5000 --username=p --secret=p --log-level=INFO
```

The `--campaign` flag names the zip without its extension, so `--campaign=demo` loads `campaigns/demo.zip`.

> Note: The log level defaults to `NONE`, which produces an essentially empty log. You must pass `--log-level=INFO` (or higher) to see the verification lines. Valid levels, from quietest to loudest, are `NONE`, `FATAL`, `ERROR`, `WARN`, `INFO`, `DEBUG`, and `TRACE`.

The engine writes to `log.txt` in its data folder (see [Where files go](#where-files-go)). Open that file and look for two lines that confirm success:

- `Campaign validation passed`
- `Successfully created Entity`

If both appear, the campaign loaded and every entity was created. If validation fails instead, the log names the entity and field at fault.

## See also

- [What is Isometry](/manual/getting-started/introduction.html) — the big picture before you set up.
- [Your first campaign](/manual/getting-started/your-first-campaign.html) — build the smallest campaign that loads.
- [Glossary](/manual/getting-started/glossary.html) — definitions of every term used here.
- [Campaign structure](/manual/core-concepts/campaign-structure.html) — file layout and the merge model.
- [Validation](/manual/core-concepts/validation.html) — what the load check verifies and how errors read.
