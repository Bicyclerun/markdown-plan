# Markdown-Plan DSL Specification

## Enhanced Domain-Specific Language

This document describes the **complete and current** markdown-plan DSL, including:
- Core task syntax
- Task kinds (new feature)
- Calendar and scheduling (new feature)
- Inventory rules (explicit)
- Invalid cases and error handling

---

## 1. Core Task Definition

### 1.1 Basic Task Syntax

Any list item or heading is a **task**:

```markdown
# Project Title          (Heading = Task)
- Task 1               (List item = Task)
  - Subtask 1.1       (Nested = Subtask)
- Task 2
```

**Parsing Rule**: Only list items (`-`, `*`, `+`, `1.`) and headers (`#`, `##`, etc.) are parsed. Everything else (prose, code blocks) is ignored.

### 1.2 Hierarchy

Nesting represents sub-tasks:

```markdown
1. Phase 1
   - Setup
   - Initialize
2. Phase 2
   - Build
     - Backend
     - Frontend
```

**Ordering Semantics**:
- Ordered lists (`1.`) suggest **sequential execution**
- Unordered lists (`-`) suggest **parallel/concurrent** work

---

## 2. Task Properties

### 2.1 Task Status

Use Markdown checkboxes:

```markdown
- [x] Completed task
- [ ] Pending task
- [~] In Progress (may be supported)
```

### 2.2 Task Kind (NEW)

Specify task type using `{kind: ...}` notation:

```markdown
- Design API {kind: design}
- Implement backend {kind: development}
- Write tests {kind: testing}
- Deploy to prod {kind: deployment}
- Code review {kind: review}
```

**Valid Task Kinds**:
- `design` - Planning, architecture, specification
- `development` - Implementation, coding
- `testing` - QA, unit/integration/e2e tests
- `documentation` - Writing docs, API specs
- `deployment` - Release, ops, devops
- `review` - Code review, quality gates
- `infrastructure` - Setup, CI/CD, monitoring
- `other` - Default/fallback kind

**Parser Behavior**: Ignores `{kind: ...}` if not recognized (backward compatible).

### 2.3 Dependencies

Explicit dependencies using `@(...)` syntax:

```markdown
- Requirement A
- Requirement B
- Deploy @(Requirement A, Requirement B)
```

**Rules**:
- Reference is a unique **substring** of task name
- Cannot include commas (ambiguity)
- Prefer hierarchy for implicit ordering

---

## 3. Calendar & Scheduling (NEW)

### 3.1 Dates and Deadlines

Specify dates using ISO format in task metadata:

```markdown
- Feature X {start: 2025-12-20, due: 2025-12-27}
- Bug fix {due: 2025-12-25}
- Deployment {start: 2025-12-28, end: 2025-12-29}
```

**Supported Fields**:
- `start: YYYY-MM-DD` - Start date
- `due: YYYY-MM-DD` - Deadline/due date
- `end: YYYY-MM-DD` - End date
- `duration: Nd` - Duration (e.g., `5d` = 5 days)

### 3.2 Effort Estimation

```markdown
- Backend API {effort: 8h}   (hours)
- Testing {effort: 3d}        (days)
- Documentation {effort: 2w}  (weeks)
```

**Units**:
- `Nh` = hours
- `Nd` = days (default 8h per day)
- `Nw` = weeks (default 5d per week)

### 3.3 Calendar Integration

The calendar system:
1. Parses all `start`, `due`, `end` dates
2. Builds a project timeline
3. Calculates critical path
4. Predicts completion dates
5. Flags deadline conflicts

**Example**:
```markdown
# 2025-Q4 Release

- Design {due: 2025-12-20}
  - UI mockups {due: 2025-12-20}
  - API spec {due: 2025-12-20}
- Development {start: 2025-12-21, due: 2025-12-27} @(Design)
  - Backend {effort: 5d}
  - Frontend {effort: 5d}
- Testing {start: 2025-12-28, due: 2025-12-30} @(Development)
  - Unit tests {effort: 1d}
  - Integration {effort: 1d}
- Deployment {due: 2025-12-31} @(Testing)
```

---

## 4. Inventory Rules (EXPLICIT)

These rules govern DSL parsing and task validity:

### 4.1 Parsing Inventory

What the parser **extracts** from Markdown:

| Element | Extracted? | Example |
|---------|-----------|----------|
| List items | ✓ YES | `- Task name` |
| Headers | ✓ YES | `# Phase 1` |
| Checkboxes | ✓ YES | `[x]` or `[ ]` |
| Dependencies | ✓ YES | `@(Task A, Task B)` |
| Metadata | ✓ YES | `{kind: dev, due: 2025-12-20}` |
| Prose | ✗ NO | Regular paragraphs |
| Code blocks | ✗ NO | ```...``` |
| Tables | ✗ NO | Markdown tables |
| Comments | ✗ NO | Prose or code |

### 4.2 Task Inventory Constraints

**Each task MUST have**:
- Unique name (substring-based)
- Valid hierarchy level (nesting depth)
- Status: `[x]`, `[ ]`, or unspecified

**Each task MAY have**:
- Description (prose after task name)
- Task kind: `{kind: design|development|testing|...}`
- Dates: `{start, due, end}`
- Effort: `{effort: Nh|Nd|Nw}`
- Dependencies: `@(...)` references

### 4.3 Metadata Inventory

Valid metadata fields:

```
kind:       design | development | testing | documentation | deployment | review | infrastructure | other
start:      YYYY-MM-DD
due:        YYYY-MM-DD
end:        YYYY-MM-DD
effort:     Nh | Nd | Nw
owner:      username or email
priority:   high | medium | low
tags:       comma-separated
```

### 4.4 Backward Compatibility

Unknown metadata fields are **silently ignored**:

```markdown
- Task with custom field {custom: value, kind: dev}
# Parsed as: kind=dev, custom field ignored
```

This ensures old plans work with new parsers and vice versa.

---

## 5. Invalid Cases

These patterns are **INVALID** and should be rejected or handled:

### 5.1 Malformed Syntax

```markdown
# INVALID: Unmatched brackets
- Task [x [incomplete

# INVALID: Malformed dependency
- Task @(
Task A, Task B)

# INVALID: Invalid date format
- Task {due: 12/25/2025}  # Should be YYYY-MM-DD

# INVALID: Invalid effort unit
- Task {effort: 5x}  # Should be 5d, 5h, or 5w
```

### 5.2 Circular Dependencies

```markdown
# INVALID: Circular dependency
- Task A @(Task C)
- Task B @(Task A)
- Task C @(Task B)  # Creates cycle: A -> C -> B -> A
```

**Handling**: Parser should detect and report circular dependencies.

### 5.3 Impossible Scheduling

```markdown
# INVALID: Deadline before start
- Task {start: 2025-12-31, due: 2025-12-01}

# INVALID: Deadline before dependency completes
- Design {due: 2025-12-20}
- Dev {due: 2025-12-20} @(Design)  # Impossible: Dev can't finish when Design finishes
```

### 5.4 Ambiguous References

```markdown
# INVALID: Ambiguous substring
- Design Phase 1
- Design Phase 2
- Implement @(Design)  # Which "Design"? Ambiguous!
```

**Rule**: Dependency substring must uniquely identify ONE task.

### 5.5 Invalid Task Kinds

```markdown
# INVALID: Unknown kind
- Task {kind: unknown_kind}  # Not in valid list

# VALID: Ignored if parser doesn't recognize
- Task {kind: custom}  # Backward compat: silently ignored
```

### 5.6 Missing Required Fields

```markdown
# INVALID in strict mode: Task without name
- {kind: dev}  # Task name is empty

# INVALID: Task with only metadata
- {start: 2025-12-20, due: 2025-12-27}  # Missing task name
```

---

## 6. Complete Example

```markdown
# Q4 2025 Release Plan

Quarterly release with design, development, testing, and deployment phases.

## Phase 1: Design

- [x] Requirements {kind: design, due: 2025-12-15}
- [ ] Architecture {kind: design, due: 2025-12-20}
  - [ ] API specification {kind: design, due: 2025-12-20}
  - [ ] Database schema {kind: design, due: 2025-12-20}
  - [ ] UI mockups {kind: design, due: 2025-12-20}

## Phase 2: Development

- [ ] Backend API {kind: development, start: 2025-12-21, due: 2025-12-27, effort: 5d} @(Architecture)
  - [ ] User service {kind: development, effort: 2d}
  - [ ] Product service {kind: development, effort: 2d}
  - [ ] Order service {kind: development, effort: 1d}
- [ ] Frontend {kind: development, start: 2025-12-21, due: 2025-12-27, effort: 5d} @(Architecture)
  - [ ] Components {kind: development, effort: 2d}
  - [ ] Pages {kind: development, effort: 2d}
  - [ ] Integration {kind: development, effort: 1d}

## Phase 3: Testing

- [ ] Unit tests {kind: testing, start: 2025-12-28, due: 2025-12-29, effort: 1d} @(Backend API, Frontend)
- [ ] Integration tests {kind: testing, start: 2025-12-28, due: 2025-12-29, effort: 1d} @(Backend API, Frontend)
- [ ] E2E tests {kind: testing, due: 2025-12-30, effort: 1d} @(Integration tests)

## Phase 4: Deployment

- [ ] Code review {kind: review, due: 2025-12-30} @(Unit tests)
- [ ] Staging deployment {kind: deployment, due: 2025-12-31} @(Code review)
- [ ] Production deployment {kind: deployment, due: 2026-01-01} @(Staging deployment)
```

---

## 7. Summary: DSL Syntax Reference

| Feature | Syntax | Example |
|---------|--------|----------|
| Task | List/header | `- Task` or `# Phase` |
| Subtask | Indentation | `  - Subtask` |
| Done | Checkbox | `- [x] Done` |
| Pending | Empty checkbox | `- [ ] Pending` |
| Kind | `{kind: X}` | `{kind: development}` |
| Start date | `{start: YYYY-MM-DD}` | `{start: 2025-12-20}` |
| Due date | `{due: YYYY-MM-DD}` | `{due: 2025-12-27}` |
| Effort | `{effort: Nh\|Nd\|Nw}` | `{effort: 5d}` |
| Dependency | `@(Task)` | `@(Requirement A, B)` |
| Ignored | Prose, code, etc. | Descriptions, details |

---

## 8. Design Principles

1. **Markdown as Input** - Only Markdown, no formulas or logic
2. **Human-First** - Prioritize readability over strict syntax
3. **Backward Compatible** - Unknown fields ignored, not errors
4. **Extensible** - New fields can be added without breaking old parsers
5. **Minimal Friction** - Familiar to Markdown users
6. **Explicit Inventory Rules** - Clear parsing and validation rules
7. **Error Handling** - Clear error messages for invalid cases
