# plan
kind: let
---

## Beads Graph Summary

**Total issues:** 13 (8 ready, 5 blocked)
**Epic:** bd-19o "Daylight Mirror 10x Improvement" (parent of all 12 others)

## Tracks

### Track A: Distribution (can run in parallel with B and C)
| Order | ID | Title | P | Type | Blocked? |
|-------|----|-------|---|------|----------|
| A1 | bd-fma | Fix Homebrew cask APK download URL | 1 | bug | ready |
| A2 | bd-1hs | Fix Mac app code signing | 1 | bug | ready |
| A3 | bd-2do | Ship binaries for all tagged releases | 2 | task | ready |

**Note:** bd-1hs unblocks 2 downstream items (bd-2z7 Bundle APK, bd-3aj Zero-config). Fixing signing first maximizes unblocks.

### Track B: Bugs (can run in parallel with A and C)
| Order | ID | Title | P | Type | Blocked? |
|-------|----|-------|---|------|----------|
| B1 | bd-39g | Fix Android package name mismatch | 1 | bug | ready |
| B2 | bd-1v0 | Achieve advertised 30 FPS | 1 | bug | ready |

**Note:** bd-1v0 unblocks bd-36n (demo video) -- no point recording a demo at 13 FPS.

### Track C: Docs (can run in parallel with A and B)
| Order | ID | Title | P | Type | Blocked? |
|-------|----|-------|---|------|----------|
| C1 | bd-3h2 | Update README with honest alpha status | 2 | task | ready |

### Track D: UX (blocked -- skip for now)
| ID | Title | P | Blocked by |
|----|-------|---|------------|
| bd-3aj | Zero-config USB detection | 2 | bd-1hs (signing), bd-19o (epic) |
| bd-2z7 | Bundle APK inside Mac app | 2 | bd-1hs (signing), bd-19o (epic) |
| bd-1dy | Embed minimal ADB binary | 2 | bd-3aj, bd-19o |
| bd-2d8 | Auto-reconnect on USB disconnect | 3 | blocked |
| bd-cb6 | Add menu bar connection status | 3 | blocked |

### Standalone
| ID | Title | P | Notes |
|----|-------|---|-------|
| bd-36n | Record demo video | 1 | ready, but depends on bd-1v0 logically (don't demo at 13 FPS) |

## Proposed Execution Order

**Phase 1 -- P1 bugs (do first, all parallel-safe):**
1. **bd-39g** Fix Android package name mismatch -- small, fast, fixes scripted launch
2. **bd-fma** Fix Homebrew cask APK download URL -- small, fixes broken install path
3. **bd-1hs** Fix Mac app code signing -- unblocks Track D downstream

**Phase 2 -- P1 performance + P2 distribution:**
4. **bd-1v0** Achieve advertised 30 FPS -- harder, investigate SCStream dirty-pixel workaround
5. **bd-2do** Ship binaries for all tagged releases -- mechanical, needs signing fix landed first ideally

**Phase 3 -- Docs + demo:**
6. **bd-3h2** Update README with honest alpha status -- informed by what we learned fixing bugs
7. **bd-36n** Record demo video -- do after FPS fix lands

**Phase 4 -- UX (unblocked after Phase 1):**
8. Track D items in dependency order: bd-3aj -> bd-2z7 -> bd-1dy

## Rationale

- P1 bugs first: three of them (bd-39g, bd-fma, bd-1hs) are small fixes that remove user-facing breakage.
- bd-1hs is the critical-path item: it unblocks both UX features and is required for trustworthy distribution.
- FPS (bd-1v0) is P1 but harder; start investigation in Phase 1 but expect it to land in Phase 2.
- README update deferred to Phase 3 so it reflects actual fixed state rather than guessing.
- Demo video last in Phase 3 since it should show the product working well.
