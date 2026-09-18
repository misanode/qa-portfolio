# Squidd — Test Plan

## 1. Scope

**In scope**
- Spotify connection: OAuth sign-in, token storage, refresh, sign-out
- Playback control: play/pause, skip, progress bar, now-playing state sync
- Apple Music playback support
- Global hotkeys
- Settings: persistence of preferences (`AppStore.swift`), Settings modal UI
- Widget window geometry: sizing, clamping to screen, corner-drag resize (`WidgetGeometry.swift`)
- Now-playing pill: layout, transitions, customization (colors, GIFs, music notes, glow ring)

**Out of scope**
- Spotify / Apple Music service behavior itself (API uptime, catalog)
- Performance and load testing
- Localization
- macOS versions below the deployment target (26.0)

## 2. Risk areas (ranked)

<!-- REVIEW: your call — ranking is a draft; adjust likelihood/impact to your judgment. -->

| Rank | Area | Why it is risky |
|------|------|-----------------|
| 1 | Spotify auth / token lifecycle | Expired or revoked tokens break every feature; hard to observe from the UI |
| 2 | Playback control / desync | App state can drift from the real player state (open #3: pause button and progress bar) |
| 3 | Apple Music | Second integration path, newer code, less exercised |
| 4 | Global hotkeys | System-level registration; conflicts with other apps |
| 5 | Settings persistence (`AppStore.swift`) | Lost or corrupted preferences on relaunch |
| 6 | Window geometry (`WidgetGeometry.swift`) | Clamping/resizing math; pure logic, easy to unit test |
| 7 | Pill / modal rendering | Visual defects, low user impact (#2, #6, #8 all landed here) |

## 3. Approach

- **Automated (XCTest):** pure logic with no UI or network dependency — `WidgetGeometry`, `CardCorner`, settings encode/decode.
- **Manual scripted:** test cases in [test-cases/](./test-cases), executed and recorded in [test-runs/](./test-runs).
- **Exploratory:** time-boxed sessions on auth, playback sync, and visual transitions (animations can't be reproduced by unit tests — e.g. #6).
- **Confirmation testing:** every fixed defect gets its linked test case re-run; result recorded in a test run.
- **Regression:** re-run all test cases in the affected feature area after each fix.

## 4. Test design techniques

<!-- REVIEW: your call — candidate values below; decide which to turn into test cases. -->

**Boundary value analysis**
- `WidgetGeometry.minimum` = 282 × 170: widths 281 / 282 / 283, heights 169 / 170 / 171; frame larger than the screen.
- `WidgetGeometry.launcherAllowance` = 72: window height at `screen.height - 72` ± 1; top edge dragged to `bounds.maxY - 72`.
- Screen smaller than the minimum size (clamp falls back to screen bounds).

**Equivalence partitioning**
- `CardCorner`: topLeft, topRight, bottomLeft, bottomRight — one resize case per corner.
- Auth states: never signed in / valid token / expired token (refreshable) / revoked token / no network.

## 5. Environment

| Item | Value |
|------|-------|
| OS | macOS 27.0 (deployment target 26.0) |
| Xcode | 27.0 (27A266a) |
| Accounts | Spotify (Premium), Apple Music |
| Build | Record the commit hash in each test run |

## 6. Entry / exit criteria

**Entry**
- Build compiles and launches from Xcode
- Test cases for the area under test are written and reviewed

**Exit**
- All planned test cases executed and recorded in a test run
- No open High-severity defects
- Every fixed defect has a passing confirmation test
- Automated tests pass in CI
