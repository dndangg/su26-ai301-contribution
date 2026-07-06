# Contribution #1: 	Allow users to specify the drag-and-drop button in the configuration

**Contribution Number:** 1 

**Student:** Dylan Dang

**Issue:** https://github.com/miracle-wm-org/miracle-wm/issues/348

**Status:** Phase III - In Progress

---

## Why I Chose This Issue

I chose this issue because it is a focused feature in an area that affects real user interaction. It is a good opportunity for me to learn how Miracle WM routes mouse events, how configuration is parsed through and applied, and how changes in the configuration model can affect behavior. The scope seems manageable, and it's in a language that I want to gain more experience in for my future work.

---

## Understanding the Issue

### Problem Description

The drag and drop behavior does not allow users to choose which mouse button should start or end a drag. Instead, it reslies on the existing button handling. There is no dedicated configuration for this feature.

### Expected Behavior

Users should be able to configure one or more mouse buttons for drag and drop. Only the configured buttons should trigger dragging and stop the drag action.

### Current Behavior

Drag and drop is handled without a dedicated button setting, so the feature cannot be restricted to a user selected mouse button.

### Affected Components

src/drag_and_drop_service.cpp for the drag start and stop logic

src/policy.cpp for pointer routing into the service

src/config.h and src/config.cpp for the configuration interface and filesystem config values

miracle-wm-c/src/config-cpp.cpp for the YAML parsing of drag and drop settings

---

## Reproduction Process

### Environment Setup

I had to set up a Linux virtual machine on my Mac because this project would not build natively on macOS. That was the first problem I ran into, since I originally started on my laptop and realized the build instructions depend on Linux only packages and tools. I ended up using an Ubuntu VM through Parallels, then followed the project’s build guide to install the required dependencies, clone the repository, and build the project in Debug mode with CMake.

The biggest challenge was getting the environment ready before I could even test the issue. I ran into many missing package errors at first, so I went back to the documentation, installed the missing Linux and Mir dependencies, and rebuilt until everything worked. After that, I was able to run the compositor in the VM and start checking the drag and drop behavior.

### Steps to Reproduce

1. Start MiracleWM in the Linux virtual machine with drag and drop enabled in the config.
2. Try to begin dragging a container using the mouse button behavior described in the issue.

You will note that the drag and drop behavior is tied to the primary button path, and there is no separate user configurable button option for it.

### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/dndangg/miracle-wm/tree/feature/348-configurable-dnd-button
- **Screenshots/logs:** [If applicable]
- **My findings:** The issue comes from the fact that drag and drop is currently handled through the primary mouse button logic, while the configuration only exposes drag and drop enabling and modifiers. I confirmed that there is no separate setting yet for choosing which mouse button should start or stop dragging.

---

## Solution Approach

### Analysis

The drag-and-drop button is hardcoded and configurable. In drag_and_drop_service.cpp, both the start and stop of a drag key off config->get_primary_button():

Start (on mir_pointer_action_button_down): if ((config->get_primary_button() & buttons) == 0) return false;
Stop (on mir_pointer_action_button_up while dragging): if (action == mir_pointer_action_button_up && (config->get_primary_button() & buttons) == 0)
get_primary_button() returns the global options.primary_button (default mir_pointer_button_primary), which isn't parsed from YAML. So the root cause is that no feature button setting is placed into the drag_and_drop config section. The service has nowhere to read a user preference from.

### Proposed Solution

Add a single button to the existing drag_and_drop configuration section (press-to-start / release-to-drop, configurable). Default it to mir_pointer_button_primary so existing behavior is unchanged. Thread it through the established config pipeline that enabled/modifiers already use, then have the drag service read config->drag_and_drop().button instead of get_primary_button().

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Users currently cannot choose which mouse button initiates/terminates a window drag. It's always the primary button. We want a config option, like
drag_and_drop: button: secondary   # left/primary (default), right/secondary, middle/tertiary, back, forward, side, extra, task

**Match:** existing patterns this mirrors exactly:

DragAndDropConfiguration already carries enabled + modifiers end-to-end (config-cpp.h:87) — button is a third field on the same struct.
try_parse_modifiers is the template for a string→bitmask parser; I'll add an analogous try_parse_button mapping names → mir_pointer_button_* (enums.h:200-207: primary, secondary, tertiary, back, forward, side, extra, task).
The C ABI struct miracle_drag_and_drop_config_t (config.h:974) and its get/set (config-c.cpp:1008) already mirror the C++ struct field-for-field.
Whole-struct merge (config-cpp.cpp:~1986) and plugin-config copy (~1401) pass drag_and_drop by value, so the new field propagates with no change.

**Plan:**

config-cpp.h — add uint button = mir_pointer_button_primary; to DragAndDropConfiguration.
config-cpp.cpp — add try_parse_button(string) -> std::optional<uint> (aliases: left=primary, right=secondary, middle=tertiary, plus the rest); in read_drag_and_drop (line 950) parse the optional button key; in the serializer (~1673) emit button when non-default.
config.h / config-c.cpp — add unsigned int button; to miracle_drag_and_drop_config_t and wire it into get/set.
src/drag_and_drop_service.cpp — replace both config->get_primary_button() calls with config->drag_and_drop().button. (get_primary_button() stays for its other callers.)
Tests — extend DragAndDropAllValues in test_filesystem_configuration.cpp:487; add default / valid-parse / invalid-name / C-API round-trip cases in miracle-wm-c/tests/test_config_cpp.cpp and test_config_c.cpp.
Docs — add drag_and_drop.button (with allowed values) to the config reference under docs/.

**Implement:** 

**Review:** self-checklist:

 Default value preserves current behavior (no breaking change for existing configs).
 Invalid button name produces a config error via the existing create_error path, not a crash.
 C ABI struct + getters/setters kept in sync with the C++ struct.
 Config round-trips (parse → serialize → re-parse) without drift.
 New button key is documented.

**Evaluate:** sudo apt install -y libgtest-dev libgmock-dev, reconfigure with -DENABLE_TESTS=ON, run the config suite (parse, default, invalid, round-trip, C-API).
Manual (nested): set button: secondary in ~/.config/miracle-wm/config.yaml, launch WAYLAND_DISPLAY=wayland-99 miracle-wm, and confirm a right-click drag (with the configured modifier) moves a window while left-click does not — then flip back to default and confirm left-click drag still works

---

## Testing Strategy

### Unit Tests

- [x] **Test case 1 — `DragAndDropAllValues` (extended):** Parses `button: secondary` from YAML and asserts `config.drag_and_drop().button == mir_pointer_button_secondary` (alongside the existing `enabled`/`modifiers` checks). Covers the happy path through `FilesystemConfiguration`.
- [x] **Test case 2 — `DragAndDropMissingButton` (new):** Config omits `button`; asserts it defaults to `mir_pointer_button_primary`. Guards backward compatibility — existing configs behave exactly as before.
- [x] **Test case 3 — `DragAndDropButtonAlias` (new):** Parses `button: middle` and asserts it resolves to `mir_pointer_button_tertiary`. Confirms friendly aliases (`left`/`right`/`middle`) work, not just canonical Mir names.
- [x] **Test case 4 — `DragAndDropInvalidButtonFallsBackToDefault` (new):** Parses `button: not_a_button`; asserts it falls back to `primary` instead of crashing. Confirms the invalid-input path is safe.

> Status: all four are written and committed to the test suites. Suite has **not been executed yet** (test build OOM-killed on the VM — pending a low-parallelism rebuild).

### Integration Tests

- [x] **Scenario 1 — C-ABI round trip (`test_config_c.cpp::DragAndDropConfig`):** Sets `button = secondary` via `miracle_config_set_drag_and_drop`, reads it back via `miracle_config_get_drag_and_drop`, asserts equality. Confirms the field is wired through the public C struct used by plugins.
- [x] **Scenario 2 — Config merge + parse/serialize round trip:** Asserts a merged (layered) config carries the overriding `button`, and the serializer emits `button` only when non-default so a written-back config doesn't drift. Confirms the value survives merging and persistence.

> Status: written/committed; execution pending with the unit suite above.

### Manual Testing

- [ ] **Pending.** Plan: set `drag_and_drop: { button: secondary }` in `~/.config/miracle-wm/config.yaml`, launch nested via `WAYLAND_DISPLAY=wayland-99 miracle-wm`, and confirm a **right-click + modifier** drag moves a window while **left-click** does not — then revert to default and confirm left-click drag still works. Blocked on getting the test/compositor build through (OOM during test-binary link).

**Verified to date:** main (non-test) build compiles and links cleanly; `clang-format` reports no changes (diff already conformant to `.clang-format`).

---

## Implementation Notes

### Week 1 Progress — Environment & Planning

Built miracle-wm from source on an arm64 Parallels VM (no arm64 release on the PPA or stable snap). Worked through several environment blockers: a stale `libmirrenderer-dev` in the PPA (its headers moved into `libmirplatform-dev` in Mir 2.28), a WasmEdge 0.13-vs-0.14 API gap (disabled the plugin system to build), missing graphics drivers (`gbm-kms`/`wayland`), an `ldconfig` cache miss, and a nested `wayland-0` socket collision. Outcome: a working local compositor plus a `compile_commands.json`-backed IntelliSense setup.

Then analyzed issue #348 with the UMPIRE framework: traced that the drag button is hardcoded to `config->get_primary_button()` in `drag_and_drop_service.cpp`, and that a `drag_and_drop` config section (`enabled` + `modifiers`) already exists end-to-end. **Key decision:** a single configurable `button` (press-to-start / release-to-drop) rather than separate start/terminate buttons — preserves the existing hold-to-drag model and keeps config minimal.

### Week 2 Progress — Implementation

Implemented the feature by threading one `button` field through the entire existing config pipeline (struct → YAML parse → serialize → C ABI → compositor), reusing the `modifiers` machinery as a template. Challenges/decisions:

- Created a dedicated `buttons.h` name→enum map (mirroring `modifiers.h`) so the C++ parser and C-ABI layer share one source of truth.
- Added friendly aliases (`left`/`right`/`middle`) on top of Mir's canonical names for usability.
- Defaulted to `mir_pointer_button_primary` so the change is fully backward-compatible; serializer omits the key unless changed.
- Added 4 unit + 2 integration test cases and updated the config docs.
- **Open item:** test-binary build was OOM-killed; needs a low-parallelism build (`-j 2`) or more VM RAM before the suite runs and manual testing can proceed.

### Code Changes

- **Files modified:**
  - `miracle-wm-c/include/miracle/cpp/buttons.h` *(new)* — button name→`mir_pointer_button_*` map (incl. aliases)
  - `miracle-wm-c/include/miracle/cpp/config-cpp.h` — `button` field on `DragAndDropConfiguration` + include
  - `miracle-wm-c/src/config-cpp.cpp` — `try_parse_button` helper, parse in `read_drag_and_drop`, conditional serialize
  - `miracle-wm-c/include/miracle/config.h` — `button` field on C struct `miracle_drag_and_drop_config_t`
  - `miracle-wm-c/src/config-c.cpp` — wired `button` into the C get/set functions
  - `src/drag_and_drop_service.cpp` — use `config->drag_and_drop().button` for start and stop checks
  - `tests/test_filesystem_configuration.cpp` — 1 extended + 3 new test cases
  - `miracle-wm-c/tests/test_config_c.cpp` — C-ABI round-trip coverage
  - `miracle-wm-c/tests/test_config_cpp.cpp` — config merge coverage
  - `wiki/docs/configuration/drag_and_drop.md` — documented `button` (valid values, example, schema, default)
  - *(~105 insertions across 9 tracked files + 1 new header)*
- **Key commits:** _not yet committed — changes staged on branch `feature/348-configurable-dnd-button`; add commit/PR links here after committing._
- **Approach decisions:**
  - **Single button, not start+terminate** — preserves the hold/release model; smaller config surface.
  - **Reuse over reinvention** — followed the proven `enabled`/`modifiers` path, including the whole-struct merge (no merge-logic changes needed).
  - **Backward compatible by construction** — default preserves current behavior; key omitted from serialized output unless changed.

---


## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

- **Building a Wayland compositor from source.** Learned the CMake/Ninja toolchain for a large C++23 project, how feature flags (`-DFEATURE_PLUGIN_SYSTEM`, `-DBUILD_ERROR_REPORTER`, `-DENABLE_TESTS`) shape a build, and how to install a compositor system-wide alongside its Wayland session file.
- **How Mir/miracle-wm is layered.** Saw how the compositor selects graphics *platforms* at runtime (`gbm-kms`, `wayland`, `x11`, `stub`) and how a nested session differs from a real KMS session on a VT — including why an empty environment vs. an inherited `WAYLAND_DISPLAY` changes behavior.
- **Reading a real config pipeline end-to-end.** Traced a single option (`drag_and_drop`) through its C++ struct, YAML parser, serializer, a C-ABI bridge (`miracle-wm-c`), config merging, and the consuming service — and learned to *extend an existing pattern* rather than invent a new one.
- **Debian/apt internals.** Reading `Packages` lists directly, diagnosing version-pinned dependency conflicts, and understanding architecture-specific PPA gaps (arm64 vs amd64).
- **Tooling:** `compile_commands.json` + `c_cpp_properties.json` for IDE IntelliSense, `clang-format`, and GoogleTest/GoogleMock test structure.

### Challenges Overcome

- **No arm64 release.** The snap (edge-only) and the miracle-wm PPA (amd64-only) both failed on the arm64 VM, so I built from source instead.
- **Stale `libmirrenderer-dev` in the PPA.** It was pinned to Mir 2.23 while the rest of the stack was 2.28, making the set uninstallable. Discovered the renderer headers had moved into `libmirplatform-dev` in 2.28, so I dropped the obsolete package.
- **WasmEdge API mismatch.** The plugin code used `WasmEdge_ValTypeGenI32()` (0.14+) but the archive only had 0.13.5. Disabled the plugin system to get a working build.
- **Blank gray screen after login.** Diagnosed it as a *working* compositor with no windows — the default terminal (`gnome-terminal`) doesn't launch reliably outside GNOME. Fixed by installing `foot` and setting it in the config.
- **A chain of runtime failures:** missing shared lib (`ldconfig`), missing graphics drivers (`gbm-kms`/`wayland`), and a nested `wayland-0` socket collision (fixed with `WAYLAND_DISPLAY=wayland-99`).
- **OOM during the test build** — traced exit code 137 to the VM running out of memory when linking test binaries in parallel; the fix is a low-parallelism build.

### What I'd Do Differently Next Time

- **Read the logs first, guess later.** Several issues (stub graphics, socket lock, missing lib) were obvious once I ran the compositor directly and captured stderr — I'd go straight to that instead of reasoning from symptoms.
- **Confirm the platform/architecture up front.** Knowing it was arm64 immediately would have pointed me at "build from source" before trying the snap and PPA.
- **Build with tests enabled from the start** so the test suite is always runnable, and cap build parallelism on a memory-limited VM to avoid the OOM detour.
- **Verify manually before writing the reflection** — I documented the implementation before the test suite and manual drag test could run; next time I'd sequence verification earlier.

---

## Resources Used

- [miracle-wm wiki](https://wiki.miracle-wm.org/) — build and configuration docs, including the `drag_and_drop` config reference.
- [Mir on Launchpad — `ppa:mir-team/release`](https://launchpad.net/~mir-team/+archive/ubuntu/release) — the Mir 2.28 dev packages for arm64.
- [GitHub issue #348](https://github.com/miracle-wm-org/miracle-wm/issues/348) — the feature request itself ("good first issue").
- The miracle-wm source as its own best reference — the existing `modifiers` config path (`modifiers.h`, `read_drag_and_drop`, `try_parse_modifier`) was the template for the new `button` option.
- Mir's `MirPointerButton` enum (`/usr/include/mircore/mir_toolkit/events/enums.h`) — the canonical button values and bitmask semantics.
- [foot terminal](https://codeberg.org/dnkl/foot) — lightweight Wayland-native terminal used to get a usable session outside GNOME.
