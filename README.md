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

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

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
