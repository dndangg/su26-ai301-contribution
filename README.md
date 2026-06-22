# Contribution #1: 	Allow users to specify the drag-and-drop button in the configuration

**Contribution Number:** 1 

**Student:** Dylan Dang

**Issue:** https://github.com/miracle-wm-org/miracle-wm/issues/348

**Status:** Phase II - In Progress

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

- **Commit showing reproduction:** [Link to commit in your fork]
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

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

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

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
