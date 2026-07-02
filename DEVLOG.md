# DEVLOG

One entry per session: what was built/learned, current blockers, next task.
(See `BW_PLAYGROUND_SPEC.md` — the project's source of truth.)

---

## 2026-07-02 — Session 1: Fork setup

**Built:**
- Imported openblack's full git history (`upstream/master` @ `aa8c2b42`) as the base of this repo.
- Added the `upstream` remote (`https://github.com/openblack/openblack.git`) so periodic `git merge upstream/master` stays possible per the mergeability ground rule.
- Added project scaffolding: `BW_PLAYGROUND_SPEC.md` (source of truth), this devlog, `docs/notes/` for Phase 1 codebase-mapping notes, and defensive `.gitignore` entries for original game asset directories.

**Learned:**
- openblack vendors nothing heavy: dependencies come via the vcpkg manifest (`vcpkg.json`) plus a `vcpkg` git submodule — clone with `--recursive`.
- Upstream `.gitignore` already ignores `GameData`; we extend it defensively for other common asset-path names.

**Blockers:** none.

**Next task:** Phase 0 — build vanilla openblack (CMake + vcpkg preset), point it at the original game data, run it, and write `docs/notes/build.md` + `docs/notes/baseline.md`.
