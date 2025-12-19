# Markdown-Plan DSL (Domain-Specific Language)

## Overview

The **markdown-plan DSL** is a minimalist, human-friendly syntax for defining project plans, tasks, dependencies, and milestones using standard Markdown. It is designed to be:

- **Human-readable**: Plain English mixed with simple Markdown
- **Version-control friendly**: Works seamlessly with Git
- **Language-agnostic**: Tools can parse and extend it independently
- **Extensible**: Supports custom annotations without breaking compatibility

## Core Syntax Rules

### 1. Task Definition

Any list item or heading is a **task**:

```markdown
# Project Title (Heading = Task)

- Task 1 (List item = Task)
  - Subtask 1.1
  - Subtask 1.2
- Task 2
```

**Key Rule**: Only list items (`-` or `1.`) and headers (`#`, `##`, etc.) are parsed as tasks. Everything else is ignored.

### 2. Hierarchies and Nesting

Tasks can be nested using Markdown list indentation:

```markdown
1. Phase 1: Foundation
   - Setup environment
   - Create initial structure
2. Phase 2: Development
   - Build core features
   - Write tests
   - Create documentation
3. Phase 3: Release
   - Final testing
   - Deploy to production
```

**Ordered vs Unordered**:
- Ordered lists (`1. 2. 3.`) suggest sequential execution
- Unordered lists (`-`) can represent parallel work

### 3. Task Status: Done Marker

Mark tasks as **completed** using Markdown checkboxes:

```markdown
- [x] This task is done
- [ ] This task is pending
- [ ] This one is in progress
```

**Parsing**:
- `[x]` = completed
- `[ ]` = pending/incomplete
- Status is extracted by the parser

### 4. Dependencies

Explicit task dependencies use the `@(...)` syntax:

```markdown
- Requirement A
- Requirement B
- Deploy system @(Requirement A, Requirement B)
```

**Rules**:
- Reference is a **unique substring** of the task name
- Cannot include commas in the substring (ambiguity risk)
- Use as a last resort; prefer Markdown hierarchy for implicit ordering

### 5. Descriptions and Metadata

Any text between tasks that is **not a list item** is ignored:

```markdown
- Task 1: Build API
  The API should support REST endpoints for CRUD operations.
  
  This paragraph is IGNORED by the parser.
  
- Task 2: Write tests
```

**Key**: Only tasks (list items/headers) are parsed. Prose, code blocks, etc. are metadata for human readers.

### 6. Code Blocks and Comments

Code blocks and other Markdown elements are **completely ignored**:

```markdown
- Deploy application

```bash
echo "This is ignored by the parser"
```

- Post-deployment checks
```

The code block does not interfere with task parsing.

## Complete Example

```markdown
# E-Commerce Platform

A scalable platform for online retail.

## Phase 1: Backend

- [x] Set up database schema
- [ ] Implement user service
  - [ ] Authentication
  - [ ] User profiles
- [ ] Build product catalog
  - [ ] Create product model
  - [ ] Implement search API @(Implement user service)

## Phase 2: Frontend

- [ ] Create UI mockups
- [ ] Implement React components @(Build product catalog)
  - [ ] Header component
  - [ ] Product list
  - [ ] Shopping cart
- [ ] Connect to backend API

## Phase 3: Testing & Deployment

- [ ] Integration testing
- [ ] User acceptance testing
- [ ] Deploy to production @(Integration testing, User acceptance testing)
```

## Parsing Strategy

The parser follows this algorithm:

1. **Load** the Markdown file
2. **Extract** all list items (`-`, `*`, `+`, or numbered `1.`)
3. **Preserve** hierarchy using indentation levels
4. **Parse** task metadata:
   - Checkbox status: `[x]` or `[ ]`
   - Dependencies: Extract `@(...)` patterns
   - Description: Task text after the checkbox/marker
5. **Build** an Abstract Syntax Tree (AST) or Directed Acyclic Graph (DAG)
6. **Ignore** everything else (prose, code blocks, tables, etc.)

## Design Principles

### 1. Markdown as Input Only
The `.md` files are **human-written input**. They contain no computation, no logic. They are purely descriptive.

### 2. No Logic in Input
No formulas, no macros, no conditional statements in Markdown. The Python engine handles all calculations.

### 3. Backward Compatibility
Any "extended" DSL (e.g., custom fields, priorities) must still parse as valid vanilla markdown-plan. This ensures:
- Old tools can still read new plans
- Plans are future-proof

### 4. Minimal Friction
The DSL should feel natural to anyone familiar with Markdown. No special training required.

## Extensions (Future)

Possible extensions while maintaining backward compatibility:

```markdown
- Task with priority {priority: high}
- Task with estimate {days: 5}
- Task with assigned owner {owner: Alice}
- Task with deadline {due: 2024-12-31}
```

These would be parsed as metadata but **would not break** vanilla markdown-plan parsers (they'd just ignore the extra text).

## Summary

| Feature | Syntax | Example |
|---------|--------|----------|
| Task | List item or header | `- Build API` or `# Phase 1` |
| Subtask | Indented list item | `  - Implement auth` |
| Done | Checkbox | `- [x] Completed task` |
| Pending | Empty checkbox | `- [ ] Pending task` |
| Dependency | @(...) notation | `Deploy @(Test, Build)` |
| Ignored | Prose, code, etc. | Descriptions between tasks |

The beauty of markdown-plan is its **simplicity**. It lets planners focus on tasks and dependencies, while tools handle the complexity of visualization, forecasting, and reporting.
