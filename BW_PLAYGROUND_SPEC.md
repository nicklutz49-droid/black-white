# PROJECT: Black & White Playground
## Personal fork of openblack — implementation spec for Claude Code

**End goal:** a functioning Black & White (2001) running on modern systems via openblack, extended with added features, a sandbox mode, and an architecture I can keep experimenting with. Personal project: no distribution, GPL obligations don't trigger, original game assets are legally owned by the user.

This document is the source of truth. Work through phases in order. Each phase ends with acceptance criteria that must pass before moving on.

---

## 0. Ground rules

1. **This is a fork, not a rewrite.** Follow openblack's existing architecture, style, and conventions. Do not restructure upstream code unless a task requires it.
2. **Stay mergeable early on.** Until Phase 3, keep changes in focused commits on feature branches so upstream openblack updates can be pulled in (`git remote add upstream https://github.com/openblack/openblack.git`, periodic `git merge upstream/master`). After we diverge heavily (Phase 4+), syncing becomes optional.
3. **Never commit game assets.** The B&W data directory stays outside the repo, referenced by path/config. Add asset paths to `.gitignore` defensively.
4. **Contribute-back candidates:** when a fix or system completion is generally useful (not a personal feature), tag the commit `[upstreamable]` and keep it free of personal-feature code so it could be PR'd upstream later.
5. Maintain `DEVLOG.md`: one entry per session — what was built/learned, current blockers, next task. The codebase-mapping notes from Phase 1 live in `docs/notes/`.
6. **Match original behavior first, improve second.** When completing an unfinished system, the first target is "behaves like the 2001 game" (use the retail game running side-by-side, community documentation, and the bw1-decomp project as behavior references). Tweaks come after parity.

---

## 1. PHASE 0 — Build, run, and baseline

**Goal:** vanilla openblack building locally and running with the original game files.

### Tasks
1. Clone recursively (`git clone --recursive`), set up the toolchain: CMake + vcpkg manifest deps + the platform preset (Visual Studio / VS Code / CLion presets exist upstream).
2. Extract original game assets per openblack's documented guide: install the game from the user's original media/installer, apply official patches 1.10 then 1.2, point openblack at the install directory.
3. Build debug and release. Fix any local build breaks (toolchain drift is common in experimental projects) — these fixes are `[upstreamable]`.
4. Run the game. Document in `docs/notes/baseline.md`: what works today (rendering, camera, audio, villager pathfinding, LHVM script VM…), what is visibly missing or broken, with screenshots.
5. Set up a debugger workflow (breakpoints hitting, symbols loading) — nothing else proceeds well without this.

### Acceptance criteria
- Clean build from scratch on this machine, documented step-by-step in `docs/notes/build.md` (so it's reproducible after an OS reinstall).
- Game launches, loads a land, camera and rendering work.
- Baseline capability report written.

---

## 2. PHASE 1 — Map the territory

**Goal:** understand the codebase well enough to change it confidently. No feature work yet.

### Tasks
1. Produce `docs/notes/architecture.md`: module map (rendering, ECS/entities, game systems, LHVM script VM, resource loaders, audio), how the main loop ticks, how entities are composed, where villager AI lives, how the influence/belief systems are (or aren't) implemented.
2. Trace three vertical slices end-to-end and document each: (a) a villager deciding and pathing to a task, (b) a hand interaction (grab/throw) from input to physics, (c) how a land/level file becomes loaded state.
3. Inventory the gameplay gap: for each core B&W system — hand, villager AI, belief/worship, influence, miracles, creature, missions/LHVM scripting, save/load — record its completion status: `working / partial / stub / absent`.
4. Build the **dev console/tooling foundation** if not already present: an in-game debug overlay (ImGui is likely already integrated upstream — verify) with entity inspection, and a command console (spawn villager, add resource, teleport camera, toggle AI). This tooling pays for itself for the rest of the project.

### Acceptance criteria
- Architecture doc + three slice traces complete.
- Gap inventory table exists and drives the Phase 2 backlog.
- Debug console can spawn a villager and inspect an entity's components at runtime.

---

## 3. PHASE 2 — Complete the core simulation (parity work)

**Goal:** the game plays like Black & White: villages live, belief flows, miracles work. Order within this phase follows the Phase 1 gap inventory; the expected work, in priority order:

### 2.1 Villager AI completion
- Finish the state machine so villagers autonomously satisfy needs and work roles (forester, farmer, builder, breeder, worshipper), matching retail behavior observed side-by-side.
- Buildings: construction sites, resource hauling, completion.

### 2.2 Belief, worship & influence
- Belief income from worship and impressive acts; miracle costs; the influence ring bounding hand interaction, growing with village development.

### 2.3 Miracles & hand
- Gesture recognition parity for the retail miracle set; miracle effects (food, wood, water/rain, fire with spread, heal…); throwing/pouring feel matched against retail.

### 2.4 Creature (minimum viable)
- Whatever creature functionality upstream has, keep working. Full creature learning AI is explicitly NOT required for our goals — a creature that exists, wanders, and responds to basic leash/interaction is enough. Depth here is optional later; it must simply not crash.

### 2.5 Save/load
- Reliable save/load of sim state. If upstream's format is incomplete, extend it (our fork's saves needn't be retail-compatible).

### Acceptance criteria
- A skirmish-style land can be played for 30+ minutes: village grows autonomously, player earns and spends belief, all core miracles function, no crashes.
- Behavior parity checklist (vs. retail running side-by-side) passes for villager needs, worship, influence growth.
- Save → quit → load resumes correctly.

---

## 4. PHASE 3 — Sandbox mode (first original feature)

**Goal:** play any land freely, on my terms.

### Tasks
1. **Sandbox launcher:** menu/config to load any land file with story scripting disabled (bypass or stub the LHVM mission scripts) — pure god-sim on that terrain.
2. **Sandbox parameters:** starting belief, starting population, resource multipliers, AI opponents on/off, creature on/off. Config file + in-menu UI.
3. **God console (cheat layer, sandbox-only):** unlimited belief toggle, spawn/remove villagers and buildings, force weather/time if supported, instant-build.
4. **Land freedom:** enumerate and load all lands shipped with the game data, plus support for community custom land files if format-compatible.

### Acceptance criteria
- Any shipped land loads in sandbox with scripting off and remains stable for a long session.
- All sandbox parameters demonstrably work.
- The creature can be disabled cleanly (no crashes from systems expecting it) — this is the "B&W without the creature" experience realized as a toggle rather than surgery.

---

## 5. PHASE 4 — Playground features (the fun part)

Now the project becomes whatever I want. Backlog, roughly prioritized — each item should be a feature branch:

1. **Skirmish mode:** AI god opponent on sandbox lands using the belief/influence systems; neutral village conversion; win condition. (This mirrors retail's later lands but player-configurable.)
2. **QoL modernization:** camera improvements, UI scaling for modern resolutions, speed controls (2x/4x sim speed), autosave.
3. **New miracles:** the data/scripting pipeline for adding a miracle end-to-end (gesture, cost, effect), proven by shipping one original miracle.
4. **Villager tweaks:** adjustable AI parameters exposed to config (need decay rates, work speeds) for experimentation.
5. **Map/mod tooling:** whatever makes creating or importing custom lands easier.

Rule for this phase: one feature branch at a time, merged only when stable, so the playable build never rots.

---

## 6. Reference materials

- **openblack repo, issues, and Discord** — architecture decisions and current status live here.
- **bw1-decomp** (openblack org) — the decompilation project; the authoritative reference for exact retail behavior/structures. **Behavior reference only: never copy or translate recovered code** — it is EA's copyrighted code even though this project is private; keep our code clean so options stay open.
- **Retail game running side-by-side** (via the fan patch or openblack itself) — the ground truth for parity checks.
- **B&W modding community documentation** — file formats, land formats, LHVM scripting references.

---

## 7. Working rules for Claude Code

1. One task per session where possible: finish, test in-game, commit.
2. Follow upstream C++ style and existing patterns; when uncertain, find a similar existing implementation in the codebase and mirror it.
3. Every gameplay change gets tested by actually launching and playing — no "compiles, ship it."
4. Keep `[upstreamable]` fixes isolated from personal features.
5. If a task reveals the gap inventory was wrong (a system is more/less complete than recorded), update `docs/notes/` first, then re-plan.
6. Never touch, commit, or transform original game assets beyond what openblack's loaders already do.
