---
description: Validate delivered code against normative artifacts (spec, plan, data-model, contracts) and update spec.md to reflect intentional implementation deviations.
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
handoffs:
  - label: Fix Implementation Gaps
    agent: speckit.implement
    prompt: Address the implementation gaps identified during verification
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Goal

Validate delivered code against normative artifacts (`spec.md`, `plan.md`, `data-model.md`, `contracts/`) and update spec.md to reflect intentional implementation deviations. This command MUST run only after `/speckit.implement` has completed implementation tasks.

## Operating Constraints

**Interactive Discrepancy Resolution**: For each mismatch found, ask the user whether it's a spec gap (update spec) or implementation gap (flag for remediation). Follow the one-question-at-a-time pattern from `/speckit.clarify`.

**Atomic Writes**: Update spec.md incrementally after each confirmed deviation, preserving formatting and existing content.

**Remediation Handoff**: If implementation gaps are identified, recommend running `/speckit.implement` to address them.

**Constitution Authority**: The project constitution (`/memory/constitution.md`) remains **non-negotiable**. If an implementation deviation violates a constitution principle, it MUST be flagged as an implementation gap—never accepted as a spec update.

## Execution Steps

### 1. Initialize Verification Context

Run `{SCRIPT}` once from repo root and parse JSON for FEATURE_DIR and AVAILABLE_DOCS. Derive absolute paths:

- SPEC = FEATURE_DIR/spec.md
- PLAN = FEATURE_DIR/plan.md
- DATA_MODEL = FEATURE_DIR/data-model.md (if exists)
- CONTRACTS = FEATURE_DIR/contracts/ (if exists)
- TASKS = FEATURE_DIR/tasks.md

Abort with an error message if `spec.md` or `tasks.md` is missing (instruct the user to run the prerequisite commands).
For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

### 2. Load Normative Artifacts

Load minimal necessary context from each artifact:

**From spec.md:**

- Functional Requirements (FR-XXX)
- Non-Functional Requirements
- User Stories with acceptance scenarios
- Data Requirements (DR-XXX)
- Edge Cases
- Technical Constraints

**From plan.md:**

- Architecture/stack choices
- File structure expectations
- Phases and implementation approach
- Component responsibilities

**From data-model.md (if exists):**

- Entity definitions and attributes
- Relationships and constraints
- Type definitions

**From contracts/ (if exists):**

- API endpoint specifications
- Request/response schemas
- Error codes and handling

**From constitution (if exists):**

- Load `/memory/constitution.md` for principle validation

**From tasks.md:**

- Task IDs and descriptions
- Completion status ([X] vs [ ])
- File paths referenced
- Phase grouping

### 3. Discover Implemented Code

Scan the workspace to identify implemented artifacts:

**File Discovery:**

- Use plan.md's file structure as the primary guide for expected files
- Scan directories mentioned in tasks.md for actual implementation
- Identify source files, test files, and configuration files

**Code Analysis:**

- Extract exported functions, classes, types, and interfaces
- Identify API endpoints and routes
- Parse test files for test cases and coverage patterns
- Note error handling patterns implemented

### 4. Build Traceability Matrix

Create a mapping between requirements and implementation:

| Requirement | Task(s) | File(s) | Status | Notes |
|-------------|---------|---------|--------|-------|
| FR-001 | T003, T005 | oee-display.tsx | ✓ Implemented | |
| FR-002 | T004 | oee.ts | ✓ Implemented | |
| FR-003 | T003 | oee-display.tsx | ? Partial | Missing loading state |

**Status Categories:**

- ✓ **Implemented**: Code fully satisfies the requirement
- ? **Partial**: Code exists but may not fully satisfy requirement
- ✗ **Missing**: No implementation found for this requirement
- Δ **Deviation**: Implementation differs from specification

Track coverage for:

- Functional Requirements → Tasks → Code
- User Stories → Acceptance Scenarios → Tests
- Key Entities → Data Model → Implementation
- API Contracts → Endpoint Implementations

### 5. Compliance Analysis

Perform systematic verification across all normative dimensions:

#### A. Functional Requirement Verification

For each FR-XXX in spec.md:

- Locate implementing code in expected files
- Verify behavior matches requirement description
- Check edge case handling
- Confirm test coverage exists

#### B. Data Model Verification (if data-model.md exists)

- Compare entity definitions against implemented types/schemas
- Verify attribute names, types, and constraints match
- Check relationship implementations

#### C. Contract Verification (if contracts/ exists)

- Verify API endpoints match contract specifications
- Check request/response schema compliance
- Confirm error handling matches contract

#### D. Non-Functional Verification

- Performance: Check for implementation of specified targets
- Error Handling: Verify error states match specification
- Accessibility: Check for required accessibility implementations

#### E. Test Coverage Verification

- Map test files to requirements
- Identify requirements without corresponding tests
- Check acceptance scenario coverage

### 6. Discrepancy Classification

Classify each discrepancy found:

| Type | Description | Resolution Path |
|------|-------------|------------------|
| **Spec Gap** | Implementation is correct but spec is incomplete/outdated | Update spec.md |
| **Implementation Gap** | Spec is correct but implementation is missing/incomplete | Run `/speckit.implement` |
| **Design Deviation** | Intentional change from spec requiring documentation | Update spec.md with rationale |
| **Bug** | Implementation error that violates specification | Flag for fix |

### 7. Generate Verification Report

Output a structured Markdown report:

```markdown
## Verification Report

**Feature**: [Feature name from spec.md]
**Verification Date**: YYYY-MM-DD
**Spec Path**: [FEATURE_DIR]/spec.md

### Coverage Summary

| Category | Total | Verified | Partial | Missing | Coverage |
|----------|-------|----------|---------|---------|----------|
| Functional Requirements | 11 | 9 | 1 | 1 | 82% |
| Data Requirements | 8 | 8 | 0 | 0 | 100% |
| User Stories | 1 | 1 | 0 | 0 | 100% |
| Edge Cases | 5 | 3 | 1 | 1 | 60% |
| **Overall** | 25 | 21 | 2 | 2 | **84%** |

### Traceability Matrix

| Requirement | Status | Task(s) | File(s) |
|-------------|--------|---------|----------|
| FR-001 | ✓ | T003 | oee-display.tsx |
| FR-002 | ✓ | T004 | oee.ts |
| ... | ... | ... | ... |

### Spec Updates Applied

1. FR-003: Documented performance approach via Next.js defaults
2. FR-007: Updated timestamp format specification

### Implementation Gaps (Requires `/speckit.implement`)

1. FR-005: Missing "No Data Available" timeout message
2. Edge Case 4: Non-production hours handling not implemented

### Deferred Items

1. Edge Case 5: Component data missing handling (marked Phase 2 in spec)

### Verification Metrics

- **Requirements Verified**: 21/25 (84%)
- **Spec Updates Applied**: 2
- **Implementation Gaps Found**: 2
- **Deferred Items**: 1
```

### 8. Interactive Discrepancy Resolution

For each discrepancy (maximum 10 total), present ONE at a time:

**Format:**

```markdown
## Discrepancy 1 of N

**Requirement**: FR-003 - System MUST render the OEE value visible within 1 second of page load

**Expected (Spec)**: OEE value visible within 1 second of page load

**Found (Implementation)**: No explicit performance optimization or loading state implemented

**Location**: components/dashboard/oee-display.tsx

**Question**: How should this discrepancy be resolved?

**Recommended:** Option A - This is a spec gap; the implementation meets the requirement through Next.js default performance

| Option | Description |
|--------|-------------|
| A | Spec Gap: Implementation is acceptable, update spec to document approach |
| B | Implementation Gap: Add explicit performance optimization (flag for `/speckit.implement`) |
| C | Design Deviation: Document intentional deviation with rationale |
| D | Skip: Do not address this discrepancy now |

Reply with option letter (A, B, C, D) or accept recommendation by saying "yes".
```

**After User Response:**

- **Option A (Spec Gap)**: Queue spec update for `## Verification Notes` section
- **Option B (Implementation Gap)**: Add to remediation list for `/speckit.implement`
- **Option C (Design Deviation)**: Queue spec update with user-provided rationale
- **Option D (Skip)**: Mark as deferred, continue to next discrepancy

### 9. Spec Update Phase

For each confirmed spec update (Options A or C):

**Ensure `## Verification Notes` Section Exists:**

- Create it after `## Clarifications` if not present
- Add `### Verification YYYY-MM-DD` subheading for today's session

**Append Verification Entry:**

```markdown
- **FR-XXX**: [Original requirement text]
  - **Implementation**: [Description of actual implementation]
  - **Resolution**: [Spec Gap | Design Deviation]
  - **Rationale**: [Why implementation is acceptable]
```

**Atomic Write After Each Entry:**

- Save spec.md immediately after each update
- Preserve existing formatting and content

### 10. Provide Next Actions

Based on verification results:

**If Implementation Gaps Exist:**

```markdown
### Recommended Next Steps

1. Run `/speckit.implement` to address the N identified implementation gaps
2. After implementation, run `/speckit.verify` again to confirm compliance
```

**If All Requirements Verified:**

```markdown
### Verification Complete ✓

All requirements have been verified against implementation.
- Spec updated with N verification notes
- Feature is ready for review/release

Suggested: Run `/speckit.checklist` to confirm all quality gates are met.
```

**If Critical Gaps Exist:**

```markdown
### Critical Issues Found

The following critical gaps must be addressed before proceeding:
1. [Critical gap description]

Run `/speckit.implement` with focus on critical items first.
```

## Operating Principles

### Verification Philosophy

- **Implementation as Source of Truth**: When implementation intentionally deviates from spec, the spec should be updated to reflect reality
- **Traceability**: Every requirement should map to specific code artifacts
- **Incremental Updates**: Update spec immediately after each confirmed change

### Interaction Guidelines

- **One Discrepancy at a Time**: Never present multiple discrepancies simultaneously
- **Provide Recommendations**: Always suggest the most appropriate resolution
- **Respect User Decisions**: Accept user's classification without excessive questioning
- **Early Termination**: Allow user to stop with "done", "stop", or "proceed"

### Context Efficiency

- **Progressive Loading**: Load code context only as needed for each discrepancy
- **Token-Efficient Output**: Summarize large code blocks rather than dumping full content
- **Deterministic Results**: Rerunning without changes should produce consistent findings

### Integrity Rules

- **NEVER fabricate implementation details**: Only report what exists in code
- **NEVER modify implementation code**: This command only updates spec.md
- **NEVER skip required validations**: All FR-XXX must be checked
- **NEVER hallucinate coverage**: If code doesn't exist, report it missing
