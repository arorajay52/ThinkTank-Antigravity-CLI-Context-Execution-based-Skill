# ThinkTank Templates & Context File Examples

Use these templates to structure alignment, execution, and persistent context files.

---

## 1. Discovery Phase Template (Context-Aware)

**BEFORE using this template**, you must have already scanned the codebase.

```markdown
### 🔍 Discovery — Codebase Scan Results

I scanned your workspace and found:
- **Existing Components**: [list reusable components found, e.g., ConfirmModal, Card, Header]
- **State Pattern**: [e.g., Single AppData state in App.tsx, passed via props]
- **Styling**: [e.g., StyleSheet.create with inline color constants]
- **Navigation**: [e.g., Bottom tabs with persistent mounting]

I'll reuse these existing patterns. Here are the questions I couldn't answer from the code:

1. **[Genuine decision point]**:
   - Option A: [with trade-off explanation]
   - Option B: [with trade-off explanation]

2. **[Another genuine decision point]**:
   - Option A: [with trade-off]
   - Option B: [with trade-off]
```

**Rules for questions:**
- ❌ Never ask "Should we use your existing modal?" — just state you'll reuse it
- ❌ Never ask about tech stack — read package.json
- ❌ Never ask about styling approach — read existing styles
- ✅ Ask about UX behavior ("Should X go in tab A or tab B?")
- ✅ Ask about data modeling ("Should this be a flat list or nested?")
- ✅ Ask about scope ("Full implementation or MVP first?")

---

## 2. Practice Audit Template

```markdown
### 🛡️ Practice Audit

> [!WARNING]
> **[N] issues flagged in the proposed approach:**
>
> 1. **[Issue name]**: [Description of the anti-pattern]
>    **Risk**: [What goes wrong if we don't fix this]
>    **Recommended**: [The correct approach]
>
> 2. **[Issue name]**: [Description]
>    **Risk**: [Impact]
>    **Recommended**: [Fix]

Please confirm these corrections, or override with your reasoning.
```

```markdown
### 🛡️ Practice Audit

✅ No anti-patterns detected. Proceeding to Understanding Lock.
```

---

## 3. Understanding Lock Template

Present this to the user immediately after Discovery + Practice Audit and wait for confirmation:

```markdown
### 🔒 UNDERSTANDING LOCK

*   **Goal**: [What are we building/refactoring?]
*   **Existing Patterns to Reuse**: [Components, modals, state patterns found in codebase]
*   **Constraints**: [e.g., TypeScript, folder structure, mobile, Android-first]
*   **Priorities**: [e.g., Rich design, zero dependency bloat, fast load times]
*   **Risks**: [e.g., Version mismatches, Metro caching, overflow on small screens]
*   **Practice Audit**: [Summary of flagged items + agreed resolutions, or "Passed"]
*   **Review Mode**: [Frequently | Occasionally | Never | Let's gamble on code] (Prompt user to select)
*   **Proposed Direction**: [General flow of implementation]

***

**Does this match your expectations? Confirm to proceed to Planning and select your preferred Review Mode (Frequently, Occasionally, Never, or Let's gamble on code).**
```

---

## 4. Implementation Plan Template (`implementation_plan.md`)

```markdown
# Implementation Plan — [Plan Name]

[Brief description of the changes]

## User Review Required

> [!IMPORTANT]
> [Critical design choices or breaking changes requiring review]

## Proposed Changes

### [Component / Layer]

#### [NEW] / [MODIFY] [filename](file:///absolute/path/to/file)
*   [Clear explanation of modifications]

---

## Verification Plan

### Automated Tests
- `tsc --noEmit`

### Manual Verification
- [Steps to test]
```

---

## 5. Task Checklist Template (`task.md`)

```markdown
- [ ] [Category / Component Name]
  - [ ] Specific item 1
  - [ ] Specific item 2
- [/] [Category / Component in progress]
  - [x] Done item
  - [/] Active item
```

---

## 6. Context File Templates {#context-files}

### `.thinktank/product.md`

```markdown
# Product Context

## App Overview
- **Name**: [App name]
- **Purpose**: [One-line description]
- **Target Audience**: [Who uses this]
- **Platform**: [Expo/React Native, React Vite, etc.]

## Tech Stack
- **Framework**: [e.g., Expo SDK 52, React Native 0.76]
- **Language**: TypeScript
- **State Management**: [e.g., useState in App.tsx, prop drilling]
- **Navigation**: [e.g., Bottom tabs, manual state-based]
- **Storage**: [e.g., AsyncStorage, expo-file-system]

## Core Features
1. [Feature 1] — [brief description]
2. [Feature 2] — [brief description]
3. [Feature 3] — [brief description]

## Key Files
- `App.tsx` — [role]
- `src/components/` — [what's here]
- `src/types.ts` — [data schemas]
```

### `.thinktank/design.md`

```markdown
# Design System

## Color Tokens
| Token | Value | Usage |
|-------|-------|-------|
| `bg` | `#0f172a` | App background |
| `cardBg` | `#1e293b` | Card surfaces |
| `accent` | `#38bdf8` | Primary accent, buttons |
| `textPrimary` | `#f1f5f9` | Main text |
| `textSecondary` | `#94a3b8` | Muted text |
| `danger` | `#ef4444` | Delete, errors |
| `success` | `#22c55e` | Completions, streaks |

## Spacing Scale
`4 | 8 | 12 | 16 | 20 | 24 | 32`

## Border Radius
| Element | Radius |
|---------|--------|
| Small elements | `8` |
| Cards | `12` |
| Buttons, pills | `20-24` |

## Typography
| Style | Size | Weight |
|-------|------|--------|
| Title | `24` | `bold` |
| Subtitle | `18` | `600` |
| Body | `15` | `normal` |
| Caption | `12` | `normal` |

## Component Inventory
| Component | Location | Purpose |
|-----------|----------|---------|
| [e.g., ConfirmModal] | [file path] | [Reusable confirmation dialog] |
| [e.g., SubtabRow] | [file path] | [Horizontal tab switcher] |

## Layout Patterns
- **Screen root**: SafeAreaView → ScrollView with padding
- **Tab switching**: Persistent mount with display toggle
- **Cards**: Consistent padding (16), radius (12), elevation (2)
```

### `.thinktank/plans.md`

```markdown
# Plans Index

| Plan | Status | Date | Details |
|------|--------|------|---------|
| [Plan Name] | ✅ Complete | YYYY-MM-DD | [link to plan file] |
| [Plan Name] | 🔄 In Progress | YYYY-MM-DD | [link to plan file] |
| [Plan Name] | 📋 Planned | YYYY-MM-DD | [link to plan file] |
```

### `.thinktank/brain.md`

```markdown
# Brain — Persistent Knowledge Base

## Major Decisions
- **[Date]**: [Decision description and rationale]
- **[Date]**: [Decision description and rationale]

## User Preferences
- [Preference, e.g., "Dislikes obvious delete buttons — prefers nested in edit mode"]
- [Preference, e.g., "Prefers following existing system design, not inventing new patterns"]

## Error Registry

### [Error Title] — [Date]
- **Symptom**: [What was reported]
- **Root Cause**: [Actual cause]
- **Fix**: [What was changed]
- **File(s)**: [Affected files]

## Anti-Pattern Log
- [Pattern to avoid, e.g., "Never use conditional mount for tabs — causes lag"]
- [Pattern to avoid, e.g., "Never hardcode colors — always use design tokens"]
```

### `.thinktank/plans/[plan-name].md`

```markdown
# Plan: [Plan Name]

**Status**: 🔄 In Progress | ✅ Complete
**Created**: YYYY-MM-DD
**Updated**: YYYY-MM-DD

## Phases

### Phase 1: [Name]
- [x] Task 1
- [x] Task 2

### Phase 2: [Name]
- [/] Task 3
- [ ] Task 4

## Notes
- [Any important notes from execution]
```
