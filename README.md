# A Pawn's Tale

A single-player, chess-based tactics roguelite built with a custom C++ engine
using [raylib](https://www.raylib.com/). Start with one
pawn, fight a sequence of battles, and spend gold between fights to transform
your pieces into other chess pieces and buy stat/ability upgrades. Pieces you
lose are gone for good — every fight is a resource-management decision as much
as a tactics puzzle.

> **Status:** Pre-production. No code has been written yet — see
> [`GDD.md`](docs/GDD.md) for the full game design document and milestone plan.

## Story

A lone pawn leaves its home village in search of adventure, hoping that one
day it will earn the right to become a Knight. Every battle fought and every
upgrade earned is a step further along that road — and the King
transformation (see [`GDD.md` §8](docs/GDD.md#8-the-king-rule)) is the literal
endpoint of that dream, with all the risk that implies.

## Concept

- Chess movement rules + HP/Attack/Shield combat instead of instant capture.
- Buy piece transformations (Basic Shop, always available) and randomized
  Special Upgrades (3 options, reset every level) between fights.
- Upgrades are piece-specific and permanently lost if that piece dies.
- Save gold across levels to earn 10% interest once savings exceed $10.
- Transform a pawn into a King for a major power spike — but only one King
  is allowed, and losing it ends the run immediately (with a small gold
  consolation per surviving piece).
- Stalling is punished: after 3 turn-cycles without a capture, tiles start
  crumbling and destroy anything standing on them at the end of each
  turn-cycle, escalating until a capture happens.
- Every 5th level is a harder Boss Level with a special win condition and
  better rewards.
- Presented in 2.5D: full 3D board and pieces, fixed/semi-fixed camera angle.

Full mechanics, combat math, and the system-by-system design are documented
in [`GDD.md`](docs/GDD.md).

## Tech Stack

- **Language:** C++
- **Engine:** Custom, built on [raylib](https://www.raylib.com/) (pinned to
  [`5.5`](https://github.com/raysan5/raylib/releases/tag/5.5), via CMake
  `FetchContent`) for windowing, 3D rendering, input, and audio
- **Data format:** JSON for piece/upgrade/encounter definitions (see
  [`GDD.md` §12.2](docs/GDD.md#122-data-driven-design))
- **Build system:** CMake (>= 3.20)

## Project Status / Structure

The project itself has not been created yet. The planned folder layout
(`src/`, `data/`, `assets/`) is documented in
[`GDD.md` §12.1](docs/GDD.md#121-suggested-project-structure) and will be created
starting at milestone **M0** of the implementation plan
([`GDD.md` §14](docs/GDD.md#14-implementation-plan--milestones)).

## Getting Started

**Toolchain verified against:**
- CMake >= 3.20 (tested with 4.4.0-rc2)
- GCC/g++ 14.2.0 (MSYS2 UCRT64), via the `MinGW Makefiles` CMake generator
- raylib `5.5`, fetched automatically via CMake `FetchContent` (no manual
  install needed — first configure will clone and build it, so expect the
  first `cmake -B build` to take longer than subsequent ones)
- nlohmann/json `v3.11.3`, fetched the same way (header-only, no separate
  build step)

```sh
cmake -G "MinGW Makefiles" -B build
cmake --build build
./build/apawnstale.exe
```

raylib and nlohmann/json are both wired in; the executable currently just
runs a throwaway JSON parse round-trip and returns, no window yet. The
folder structure and the render smoke test land in later M0 tickets — this
section will be expanded as each one completes.

**Note (Windows/MinGW):** this toolchain's `ld` (GNU ld 2.43.1, MSYS2
UCRT64) fails to link any object file containing C++ exception-handling
unwind tables against the dynamic `libgcc_s_seh-1.dll`/`libstdc++-6.dll`
(`collect2.exe: error: ld returned 116 exit status`, no other diagnostic —
reproduces with a bare `throw`/`catch`, unrelated to json or raylib).
`CMakeLists.txt` already links `apawnstale` with `-static-libgcc
-static-libstdc++` to work around it, so this should be transparent to
contributors on the same toolchain — flagging it here in case it resurfaces
elsewhere (e.g. a future `tests/` target).

**Note (Windows):** if the raylib fetch fails with a git SSL error
(`unable to get local issuer certificate`), your global git config may be
set to the `openssl` SSL backend without a usable CA bundle. Switch to the
Windows certificate store instead:
```sh
git config --global http.sslBackend schannel
```

## Documentation

- [`GDD.md`](docs/GDD.md) — full Game Design Document: mechanics, economy,
  combat resolution, board decay rules, boss levels, technical architecture,
  and the phased implementation/milestone plan.
- [`DEVELOPMENT_PLAN.md`](docs/DEVELOPMENT_PLAN.md) — checkbox-style task tracker
  for implementation, broken down by milestone.
- [`M0_PROJECT_SETUP.md`](docs/M0_PROJECT_SETUP.md) — Jira-style ticket
  breakdown for milestone M0.

## Contributing

This is currently a solo/early-stage project. No contribution process is set
up yet.

## License

TBD.
