# Mangrove manifest, version 1

This document says what a mod is and how a runtime (a compatibility layer that loads the game) reads it. The
words **must**, **must not** and **should** are requirements on runtimes.

## 1. Mods on disk

- Mods live in one folder, the *mods folder*. Where it is belongs to the runtime (Leonida: `mods` next to its
  executable, or `--mods DIR`).
- Each mod is a sub-folder that holds a file named `mod.json`. A sub-folder without one is not a mod and **must**
  be ignored without an error (it may hold notes, or files of another tool).
- A mod **must not** contain game data. It refers to what the game already has, by the names or hashes the game
  uses.

## 2. Encoding

- `mod.json` is UTF-8 JSON whose top level is an object.
- A UTF-8 byte-order mark (`EF BB BF`) in front **must** be accepted and skipped.
- A UTF-16 file (with or without a byte-order mark) **must** be refused with a message that names the encoding,
  rather than a JSON error.

## 3. Format

- `"format": 1` marks this version. The value is a whole number of 1 or more.
- A manifest without `format` is **format 0** (section 6).
- A runtime **must** refuse a manifest whose `format` is higher than the highest it reads, with a message saying
  so. It **must not** read such a manifest in part.

## 4. Keys (format 1)

| Key | Type | Required | Meaning |
|---|---|---|---|
| `format` | number | yes | `1` |
| `id` | text | yes | 1-64 characters of `a-z`, `0-9` and `_`. Unique among the installed mods (section 5). |
| `name` | text | no | Shown to the player. Defaults to `id`. |
| `version` | text | no | The mod's own version, free text. |
| `authors` | list of non-empty text | no | Shown, never interpreted. |
| `description` | text | no | Shown, never interpreted. |
| `enabled` | true / false | no | Default `true`. `false` keeps the mod installed but off. |
| `game_versions` | list of non-empty text | no | The game package versions the mod was made for, e.g. `["1.0.2.1"]`. Version 1 does not require a runtime to enforce it. |
| `archetype_swaps` | list | no | Feature, section 7.1. |
| `camera` | object | no | Feature, section 7.2. |

Rules:

- A key listed above with a value of the wrong type **must** make the manifest an error. A runtime **must not**
  fall back to a default.
- `game_version` (format 0's key) in a format 1 manifest **must** be an error that names `game_versions`.
- A key not listed above **must** be skipped with a warning the player can see, and the rest of the manifest
  **must** still load. This lets a mod written for a newer runtime load the keys an older runtime knows.
  A skipped key is never applied in part.
- A manifest with an error is skipped as a whole, with a message naming its folder. Other mods still load.

## 5. Ids and order

- The `id` of a format 1 mod is its `id` key. The `id` of a format 0 mod is its folder name.
- Mods are read in the order of their folder names, sorted as text. Name folders so the order does not depend
  on case or accents (for example `10_base`, `20_overrides`).
- When two mods have the same `id`, the first in that order keeps it. The other **must** be skipped with a
  message naming both folders. This holds for disabled mods and across the two formats.
- When two enabled mods set the same thing (the same `from` in `archetype_swaps`, or both a `camera`), the later
  one wins.

## 6. Format 0

The format the first mods were written in. A runtime that reads format 1 **must** also read format 0, and read it
as follows:

- `id` is the folder name.
- `name` is used when it is text, else the folder name. A `name` of another type is ignored, not an error.
- `game_version`, when it is text, is the one entry of `game_versions`. Another type is ignored.
- `enabled`, `archetype_swaps` and `camera` are as in format 1.
- Other keys are skipped with a warning, as in format 1.

## 7. Features

A feature changes something in the game. Each one names the hook it needs in the game's executable. When a
runtime does not have that hook for the running version, it **must** load and show the feature but **must not**
apply it, and **should** say that it is not applied and why.

### 7.1 `archetype_swaps`

When the game looks up the model archetype `from`, it gets `to` instead.

```json
"archetype_swaps": [ { "from": "player_story000_00", "to": "cs_mandy", "when": "always" } ]
```

- `from` and `to` are model names or 32-bit hashes written `0x` followed by up to eight hex digits. A name is
  hashed with Jenkins one-at-a-time (`joaat`) over the name in lower case, as the game does.
- `when` is `always` (the default). Any other value **must** be refused until the specification defines it, so a
  mod never does more than it says.
- `from` equal to `to` is an error.
- A swap is applied once. A model swapped to another that is itself swapped is not followed again, so two mods
  cannot loop.
- Hook: the game's archetype lookup by name hash.

### 7.2 `camera`

```json
"camera": { "view_mode": "first_person", "first_person_value": 4, "contexts": [0] }
```

- `view_mode`: `first_person` or `default` (leave the game's choice). Required in a `camera` object.
- `first_person_value`: whole number 0-255, default 4. The game's enum value for first person.
- `contexts`: list of whole numbers 0-4294967295. The camera contexts (on foot, in a vehicle, ...) to override.
  Absent or empty: every context.
- When several enabled mods have a `camera`, the last one wins.
- Hook: the game's getter of the view mode for a camera context.

## 8. Example

```json
{
  "format": 1,
  "id": "mandy_prison_intro",
  "name": "Mandy in the prison intro",
  "version": "1.0.0",
  "authors": ["..."],
  "description": "Mandy replaces Lucia's player model.",
  "enabled": true,
  "game_versions": ["1.0.2.1"],
  "archetype_swaps": [
    { "from": "player_story000_00", "to": "cs_mandy", "when": "always" }
  ]
}
```

## 9. What version 1 does not define yet

Dependencies (`requires`, `conflicts`, `load_after`), a load-order file, checking `game_versions` against the
running package, files a mod brings, and conditions narrower than `always`. They are planned in
[ROADMAP.md](ROADMAP.md). They will be added so that a valid format 1 manifest stays valid; until then a runtime
treats those keys as unknown (section 4).
