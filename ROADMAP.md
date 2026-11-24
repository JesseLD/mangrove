# Roadmap

## Phase 1: the foundation

No new kind of content. Everything here is testable without the game, with synthetic data.

1. **Manifest v1** ([SPEC.md](SPEC.md)): format number, id, metadata, `game_versions`, format 0 read as before.
   *Done in the reference implementation (Leonida fork, not published yet).*
2. **Load order, dependencies, conflicts:** an optional `load_order.txt`; `requires`, `conflicts` and
   `load_after` between ids; the log says which mod won each conflict. A `mods list` / `mods check` command that
   validates everything without starting the game.
3. **Version lock:** the runtime identifies the game package by file hashes; a mod whose `game_versions` does not
   include it is turned off, with the reason. An override flag turns it on anyway.
4. **Per-version hooks:** the addresses of the hooks features need are kept in the runtime's per-version file,
   not on the command line. A hook with fewer callers than expected is refused, so a wrong address does not crash
   the game. A feature whose hook is unknown is loaded but not applied, and the log says so.
5. **Pluggable features:** each feature is one unit in the runtime (parse, merge, conflicts, apply), so adding a
   feature does not touch the loader.
6. **Files a mod brings:** a `files/` folder in the mod, laid over the game folder. Reads look in the player's
   overlay, then the mods (last to first), then the game; writes never go into a mod. On its own this can only
   replace whole files; it is the base for phase 2.

## Phase 2: resources inside archives

Replace a model, texture or data file inside the game's archives, with the mod bringing only that resource.
Depends on reading and rebuilding the archive and texture-pool formats well enough, without decryption or any
way around protection.

## Phase 3: scripts and game functions

A plugin interface over the game's own script functions, and conditions such as "only in this mission".
Depends on finding those functions and the mission state in each version of the executable.

## Never

Anything online, anything for piracy or protection, and any game data in this repository.
