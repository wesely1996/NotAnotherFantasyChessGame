# A Pawn's Tale — Game Design Document

**Status:** Pre-production / Planning
**Engine:** Custom, in C++ using [raylib](https://www.raylib.com/) (windowing, 3D rendering, input, audio)
**Genre:** Single-player tactics roguelite, chess-based
**Visual style:** 2.5D (3D board and pieces, fixed/semi-fixed camera angle)

> Name check: no existing released game is titled "A Pawn's Tale" (closest
> hits are an unrelated fan-fiction mention and similarly-themed but
> differently-named chess roguelites such as *Once a Pawn a King* and
> *Pawnbarian*), so the name is currently free to use.

---

## 1. High-Concept

A single-player roguelite where you start a run with one pawn and fight a sequence
of chess-like battles against AI-controlled armies. Between battles you spend gold
to transform your pieces into other chess pieces (changing how they move) and to
buy randomized power-ups (changing how strong they are). Pieces you lose are gone
forever — every battle is a resource-management puzzle as much as a tactics fight.

Think: *chess* + *auto-battler stat growth* + *roguelite shop runs* (Slay the
Spire / Auto Chess lineage), themed entirely around a chess board and chess
pieces.

### Story Premise

A lone pawn leaves its home village in search of adventure, chasing one
dream: to one day earn the right to become a Knight. Each level is a step
further from home — a battle fought, a piece transformed, a fragment of
power earned. Every other piece on the board, ally or enemy, is walking the
same road by the same rules: rank is earned through battle, not given.
Whether the pawn's journey ends in coronation, in the throne room as a King,
or shattered on some forgotten tile, is for the player to decide.

This premise frames the Basic Shop transformations (§6.1) not just as a
mechanical choice but as the pawn's personal arc — each transformation is a
step up in rank along the journey, and the King transformation (§8) is the
literal endpoint of that dream, with all the risk that implies.

---

## 2. Terminology

To avoid ambiguity for the rest of this document:

| Term | Meaning |
|---|---|
| **Run** | One full playthrough, from level 1 until the player wins or loses. |
| **Level / Encounter** | One full chess battle against an enemy army. Equivalent to "a round" in the original pitch when talking about rewards, the shop, and win/lose conditions. |
| **Turn-cycle** | One player move followed by one enemy move, inside a single encounter. This is the unit used for the board-decay timer (see §7). |
| **Piece** | Any unit on the board, player or enemy. All pieces start as pawns unless stated otherwise. |

---

## 3. Core Gameplay Loop

```
 ┌──────────────────────────────────────────────────────────────────┐
 │                                                                    │
 │   Start Run (1 pawn, $0)                                          │
 │        │                                                          │
 │        ▼                                                          │
 │   Enter Level N ──► Fight Encounter (turn-cycles, see §6/§7) ──┐   │
 │        ▲                                                       │   │
 │        │                                                       ▼   │
 │   Shop Phase  ◄────────────── Win Encounter ─────── Lose Encounter │
 │   - Basic Shop (transform)                               │        │
 │   - Special Upgrades (3 random)                          ▼        │
 │   - Gold reward + interest                          GAME OVER      │
 │        │                                            (Run ends)     │
 │        ▼                                                          │
 │   N = N + 1, repeat. Every 5th level is a Boss Level.             │
 │                                                                    │
 └──────────────────────────────────────────────────────────────────┘
```

1. Player starts a run with exactly one pawn and $0.
2. Player fights an encounter on a chess board against an AI army.
3. Win condition: eliminate all enemy pieces, or (on boss levels) satisfy a
   special objective.
4. Loss condition: all player pieces destroyed (see also King rule, §8).
5. On win: receive a gold reward, then resolve interest, then enter the Shop
   Phase.
6. In the Shop Phase: buy transformations (Basic Shop) and/or pick one of three
   randomized Special Upgrades, then proceed to the next level.
7. Repeat with increasing difficulty. Every 5th level is a Boss Level.
8. A loss ends the run. There is no mid-run continue — pieces and gold do not
   persist between runs (see §13 for optional meta-progression scope).

---

## 4. The Board & Movement

- Standard 8×8 grid (subject to per-encounter layout variation — boss levels
  may use irregular boards, see §9).
- Each piece moves according to **standard chess movement rules** for its
  current type (pawn, knight, bishop, rook, queen, king). Transformations
  change a piece's movement type but not its identity/upgrades (see §6).
- Unlike classical chess, moving onto an enemy-occupied square does not
  guarantee a kill — it triggers **combat resolution** (§5). Whether the
  defender dies depends on HP, Shield, and incoming Attack damage.
- Turn order: Player moves one piece → Enemy AI moves one piece → repeat.
  This single player-move + single enemy-move pair is one **turn-cycle**.

---

## 5. Combat Resolution System

Classical chess capture is a binary, instant kill. To support the upgrade
system (HP, Shield, multi-attack, abilities), combat is resolved with stats
instead:

| Stat | Default (unupgraded pawn) | Effect |
|---|---|---|
| **HP** | 1 | Piece is destroyed when HP reaches 0. |
| **Attack** | 1 | Damage dealt to a defender on a successful capture-move. |
| **Shield** | 0 | Flat damage reduction applied before HP loss; shield itself can have its own HP-like pool that depletes before regular HP does. |
| **Multi-attack** | Off | Lets a piece resolve combat against multiple valid targets in range during a single move (e.g., a queen hitting two pieces along one line), instead of just the first one encountered. |
| **Special abilities** | None | Unique, piece-specific effects layered on top of the base resolution (e.g., lifesteal, splash damage to adjacent tiles, counter-attack, poison-on-hit). See §6.3 for examples. |

Because base stats are HP 1 / Attack 1, an un-upgraded piece still behaves
exactly like classical chess (any hit kills). All of the depth comes from
upgrades pushing pieces away from that baseline.

**Permanent loss rule:** there is no resurrection, healing back from 0, or
"return to shop" for a destroyed piece. Once a piece's HP hits 0, it is removed
from the board and from the player's roster for the rest of the run.

---

## 6. Upgrades

All upgrades are **piece-specific** — they are bound to the individual piece
they were purchased for, not to the player globally. If that piece dies, the
upgrades invested in it are lost with it.

### 6.1 Basic Shop (Transformations)

- Always available, every shop phase, no randomization.
- Lets the player spend gold to transform a chosen piece into one of the other
  chess piece types: Knight, Bishop, Rook, Queen, King (King has special
  rules, see §8).
- Transforming changes movement rules only. Existing HP/Attack/Shield/ability
  upgrades on that piece carry over.
- Cost scales with piece type power (e.g., Knight/Bishop cheap, Rook
  mid-cost, Queen expensive, King very expensive and capped at one).
- A piece can be re-transformed later (e.g., Knight → Queen) at the
  appropriate cost difference, at the designer's discretion during balancing.

### 6.2 Special Upgrades

- Offered at the end of every level: **3 randomized options**, pick at most
  one (or none, if the player wants to save gold).
- The pool of 3 resets every level — declining or being unable to afford an
  option means it's gone, not saved for later.
- Always tied to a specific piece the player selects before/while shopping.
- Categories:
  - **Survivability:** +HP, +Shield, damage reduction.
  - **Offense:** +Attack, multi-attack, armor-piercing (ignore Shield).
  - **Utility / Special abilities:** examples below.

### 6.3 Example Special Abilities (initial design pass — to be balanced later)

| Ability | Effect |
|---|---|
| Lifesteal | Heal HP equal to a portion of damage dealt. |
| Splash | Deal reduced damage to tiles adjacent to the primary target. |
| Counter-attack | When this piece survives an attack, deal damage back to the attacker. |
| Guardian | Redirect a portion of damage aimed at adjacent allied pieces to this piece instead. |
| Reach | Extend movement/attack range by one extra tile in the piece's normal pattern. |
| Last Stand | When this piece would die, survive at 1 HP once per encounter. |

These are starting examples for prototyping/balancing, not a final list.

---

## 7. Board Decay (Demolition) Mechanic

A pressure mechanic to discourage turtling/stalling.

- Tracked per-encounter, reset at the start of every new level.
- If **3 consecutive turn-cycles** pass within the current encounter without
  any capture (player or enemy piece destroyed), the board starts to
  deteriorate:
  - At the start of the decay state, **1 tile** is marked for demolition.
  - Each additional turn-cycle that passes *without a capture* while already
    in the decay state increases the number of newly marked tiles by 1 (so:
    1 new tile, then 2 more, then 3 more, etc. — cumulative).
  - At the **end of each turn-cycle**, any piece (player or enemy) standing on
    a tile marked for demolition is destroyed, then that tile becomes
    permanently unusable (impassable, no piece can occupy or move through
    it) for the rest of the encounter.
- **Reset condition:** the moment any capture occurs, the decay timer and all
  pending "marked for demolition" warnings reset to zero, and the destroyed
  tiles already removed stay removed (they do not return).
- Decay state and removed tiles do **not** persist between encounters; every
  new level starts with a full, stable board.

This mechanic only matters in passive/stalemate-prone fights; it should not
trigger in normal aggressive play.

---

## 8. The King Rule

- Transforming a piece into a **King** is allowed via the Basic Shop, but only
  **one King may exist on the player's side at a time**.
- The King follows standard King movement (one tile in any direction) and can
  receive the same upgrades as any other piece.
- **If the King dies, the run ends immediately in a loss**, regardless of how
  many other player pieces remain alive — this overrides the normal "all
  pieces destroyed" loss condition.
- **Consolation reward:** on a King death, the player receives **$1 for every
  other player piece still alive on the board** at the moment the King dies.
  This money has no further use since the run is over, but it is recorded in
  the run summary/score (useful for meta-progression or leaderboards later,
  see §13).
- Design intent: the King is a high-risk, high-reward transformation — likely
  the strongest possible single-piece power spike, but it converts the
  player's whole run into a single point of failure.

---

## 9. Boss Levels

- Every **5th level** (5, 10, 15, …) is a Boss Level.
- Boss Levels are harder than the normal difficulty curve at that point (e.g.,
  larger/stronger enemy army, unique board layout, or modified rules for that
  encounter only).
- Win condition on a Boss Level can be the normal "eliminate all enemy
  pieces," **or** a special objective unique to that boss (examples: survive
  N turn-cycles, escort a piece to a specific tile, defeat a single named
  elite piece with very high HP).
- Boss Levels grant better rewards than normal levels: larger gold payout,
  and/or a guaranteed Special Upgrade slot, and/or a guaranteed rarer
  upgrade in the post-fight pool.

---

## 10. Economy

### 10.1 Gold Rewards

- Completing any level (normal or boss) grants a gold reward. Reward size
  scales with level number and is higher for Boss Levels (see §9).

### 10.2 Interest

- At the point gold is awarded for completing a level, if the player's
  **total saved gold exceeds $10**, they gain an **interest bonus equal to
  10% of their total saved gold** (rounded down), added on top of the level
  reward.
  - Example: player has $47 saved when the reward is granted → interest =
    floor($47 × 0.10) = **$4** bonus, before the new level's reward is even
    added.
  - This rewards saving gold across multiple levels instead of always
    spending down to $0, while still being skippable for players who prefer
    to power up immediately.

### 10.3 Spending

- Basic Shop transformations and Special Upgrades are the only gold sinks.
- No mid-encounter spending — gold can only be spent during the Shop Phase
  between levels.

---

## 11. Visual & Camera Style (2.5D)

- Board and pieces are full 3D models; presentation is "2.5D" in the sense of
  a fixed or semi-fixed camera angle (similar in spirit to *Hearthstone*,
  *Inscryption*, or *Auto Chess* games) rather than free 3D camera control.
- Recommended camera: orthographic or narrow-FOV perspective camera angled
  down over the board (~35–50° from horizontal), allowing slight rotation but
  no free-fly. Implementable directly with raylib's `Camera3D` (set
  `projection` to `CAMERA_ORTHOGRAPHIC` or a narrow `fovy` for perspective).
- Pieces should have clear, readable silhouettes per type (pawn/knight/
  bishop/rook/queen/king) so movement options are instantly recognizable,
  plus a secondary visual layer (color, glow, attached props) communicating
  upgrade tier and ability set at a glance.
- Tile state communication is critical: highlight legal moves/attacks,
  telegraph tiles marked for demolition before the turn-cycle resolves
  (§7), and visually distinguish the King piece.

---

## 12. Technical Architecture (Custom C++ Engine / raylib)

### 12.1 Suggested Project Structure

```
src/
  main.cpp               Entry point, window/init, top-level screen loop
  core/                  GameState, RunManager, ScreenFlow (MainMenu/Battle/Shop)
  board/                 GridManager, TileController, BoardDecayManager
  pieces/                PieceController, MovementRules, CombatResolver
  ai/                    EnemyAIController, EnemyEncounterComposer
  economy/               EconomyManager (gold, interest)
  shop/                  ShopManager, BasicShopUI, SpecialUpgradeUI
  upgrades/              UpgradeDefinition, UpgradeEffectHandlers
  ui/                    HUD, GameOverScreen, RunSummaryScreen
  persistence/           SaveManager (run-state only, see §13)
data/
  pieces/                PieceDefinition data files (JSON): movement pattern, base stats
  upgrades/              UpgradeDefinition data files (JSON): effect, rarity, cost, piece-type filter
  encounters/            EncounterDefinition data files (JSON): enemy composition, board layout, boss flag/objective
assets/
  models/                3D models for board/pieces (e.g. .glb/.obj, loaded via raylib)
  textures/
  animations/
```

There is no built-in scene/prefab system in raylib, so "scenes" are just
states in the top-level screen loop (`MainMenu`, `Battle`, `Shop`, or `Shop`
as an overlay drawn on top of `Battle`), and "prefabs" are plain data structs
plus a loader function that instantiates a piece/tile from a `PieceDefinition`.

### 12.2 Data-Driven Design

- **PieceDefinition** (data file, e.g. JSON parsed with a small library like
  [nlohmann/json](https://github.com/nlohmann/json)): base piece type,
  movement pattern, base HP/Attack/Shield, model/texture reference.
- **UpgradeDefinition** (data file): effect type (stat delta or special
  ability), eligible piece types/tiers, gold cost, rarity weight for random
  selection.
- **EncounterDefinition** (data file): board size/layout, enemy composition
  and placement, boss flag, special win condition (if any), reward table.
- Definitions are loaded into plain C++ structs at startup/level-load time.
  This lets designers add new pieces, upgrades, and encounters by editing
  data files without touching code — important for balancing iteration.

### 12.3 Core Systems / State Machine

`RunManager` drives the high-level flow: `RunStart → Level(N) → ShopPhase →
Level(N+1) → … → RunEnd`.

Within a level, a `TurnManager` drives: `PlayerTurn → CombatResolution →
EnemyTurn → CombatResolution → DecayCheck → WinLossCheck → repeat`.

`BoardDecayManager` listens for capture events (to reset its counter) and for
turn-cycle-end events (to mark/resolve demolition tiles per §7).

`EconomyManager` owns gold balance and applies the interest formula (§10.2)
at level-completion time, before handing control to `ShopManager`.

### 12.4 Persistence Scope (MVP)

- Within a run: track current gold, player roster (pieces + their
  upgrades), current level number — enough to support pause/resume of a
  single run if desired.
- Between runs: no persistence required for MVP. See §13 for future scope.

---

## 13. Out of Scope for MVP (Future Considerations)

These are explicitly **not** part of the initial implementation, but are
worth tracking since they came up naturally during design:

- Meta-progression between runs (unlockable pieces/upgrades, currency that
  persists across runs, run history/leaderboard using the King-death gold
  total from §8).
- Multiplayer or asynchronous PvP.
- Alternate board shapes/sizes beyond what individual boss encounters need.
- Full ability/upgrade balancing pass (the lists in §6.3 are a starting
  point, not final numbers).
- Localization, accessibility pass (colorblind-safe tile/ability indicators
  are recommended early, but full audit is post-MVP).

---

## 14. Implementation Plan / Milestones

| # | Milestone | Goal |
|---|---|---|
| M0 | Project setup | C++ project with raylib dependency wired up (via vcpkg/CMake or git submodule), folder structure (§12.1), basic window + 3D rendering baseline, version control hygiene. |
| M1 | Grid & movement prototype | 8×8 grid, camera (§11), one pawn, click-to-select/move, legal-move highlighting for all 6 piece movement patterns (no combat yet — instant capture like classical chess). |
| M2 | Turn-cycle & basic enemy AI | Player/enemy alternating turns, a minimal enemy AI (even "random legal move" is enough for this milestone), win/loss detection on full elimination. |
| M3 | Combat resolution system | Layer in HP/Attack/Shield (§5), replace instant-capture with damage resolution, verify base stats reproduce classical chess behavior. |
| M4 | Economy core | Gold reward on level win, interest formula (§10.2), minimal Shop scene/UI shell. |
| M5 | Basic Shop (transformations) | Implement piece-type transformation purchase flow (§6.1), cost table, movement pattern swap on a live piece. |
| M6 | Special upgrades | Randomized 3-option offer system (§6.2), UpgradeDefinition application to a specific piece, at least the example abilities in §6.3. |
| M7 | King rule | Single-King constraint, override loss condition, consolation gold payout (§8). |
| M8 | Board decay mechanic | Turn-cycle-without-capture counter, tile marking/telegraphing, end-of-turn-cycle destruction and tile removal, reset-on-capture (§7). |
| M9 | Boss levels | Every-5th-level detection, at least one bespoke boss encounter with a special win condition, boosted rewards (§9). |
| M10 | Encounter content pass | Author a meaningful sequence of normal-level EncounterDefinitions with an increasing difficulty curve. |
| M11 | 2.5D art & feel pass | Final piece/board models or placeholders upgraded, camera tuning, move/attack/death animations, VFX for abilities, SFX. |
| M12 | UI/UX polish | HUD, piece inspector panel, shop UI polish, game-over/run-summary screen, settings (audio, etc.). |
| M13 | Balancing & playtesting | Tune costs, stat numbers, AI difficulty curve, boss encounters; iterate based on playtests. |
| M14 | Release prep | Bug bash, platform/build settings, credits, store page assets if shipping publicly. |

Milestones M1–M3 form the playable core loop without economy; M4–M9 add every
mechanic described by the user's pitch; M10+ is content, polish, and
balancing. This ordering lets the riskiest/most novel mechanics (combat
resolution, board decay, king rule) get prototyped and validated early,
before investing in art or content volume.

---

## 15. Open Design Questions (to revisit during prototyping)

- Exact cost curve for Basic Shop transformations and how it scales across
  levels (flat, level-indexed, or based on number of pieces already owned?).
- Exact rarity weights and gold costs for Special Upgrades.
- Whether the player can decline all 3 Special Upgrade options without
  penalty (current assumption: yes).
- Whether transformed pieces can be transformed again, and whether that
  costs the full new price or a difference (current assumption in §6.1:
  designer's discretion, likely a discount).
- AI difficulty scaling approach (scripted per-encounter vs. a general
  difficulty parameter feeding a shared AI).
- Whether Boss Level special objectives are all bespoke/hand-authored or
  follow a smaller set of reusable objective "templates."
