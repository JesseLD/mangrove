# Mangrove

A mod framework for single-player GTA VI on PC compatibility layers, starting with its manifest format.

Projects such as [Leonida](https://github.com/Vyzerythe/Leonida), [XCL](https://github.com/ivall/xcl) and
[VIctory](https://github.com/Marvisak/VIctory) run the game's Xbox Series files on Windows. Because the layer
loads the game and answers every file read, mod support can live in the layer itself instead of in injected
DLLs. Mangrove describes what a mod is, so the same mod folder works on any layer that implements it.

## Status

| Part | State |
|---|---|
| [Manifest format v1](SPEC.md) | written; reference implementation in progress in a Leonida fork |
| Load order, dependencies, conflicts | planned ([roadmap](ROADMAP.md), phase 1) |
| Version lock and per-version hook addresses | planned (phase 1) |
| Files a mod brings (layers over the game folder) | planned (phase 1) |
| Replacing resources inside game archives | phase 2 |
| Scripts and game functions | phase 3 |

No mod changes the game yet: the hooks a mod needs depend on addresses in each version of the game's
executable, and those are still being found.

## Rules

- **No game data.** This repository holds a format, documentation and code written from scratch. No archives,
  executables, shaders, textures, audio, models or dumps, and no files derived from them.
- **Nothing for piracy or protection.** No keys, no decryption, no way around DRM. Mangrove works on files
  the player already has, through a layer that loads them.
- **Single-player only.** Nothing that reaches online services or gives an advantage over other players.

## Disclaimer

Mangrove is an independent project. It is not affiliated with, endorsed by or connected to Rockstar Games or
Take-Two Interactive. Grand Theft Auto and related names are trademarks of their owners and are used here only
to say which game the format is for.

Mods are made and distributed by their own authors, who are responsible for what they publish. Mangrove does not
host, review, endorse or distribute mods, and is not responsible for any mod, for what a mod does, or for how it
is used. The software and the specification are provided "as is", without warranty of any kind; see
[LICENSE](LICENSE).

## License

[MIT](LICENSE).
