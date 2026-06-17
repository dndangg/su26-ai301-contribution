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

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

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
