---
description: Verify implementation compliance against the feature specification and update spec.md to reflect validated deviations.
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

Validate that the delivered implementation matches the feature specification (`spec.md`) and supporting normative artifacts (`plan.md`, `data-model.md`, `contracts/`). For each discrepancy, determine whether it represents a spec gap (update spec.md) or an implementation gap (flag for remediation). This command runs AFTER `/speckit.implement` has completed the implementation tasks.

## Operating Constraints

**Incremental Spec Updates**: When the implementation intentionally deviates from the original spec and the user confirms the deviation is correct, update `spec.md` to reflect the as-built state. Use atomic writes after each accepted change.

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

Load the specification and supporting documents (progressive disclosure—minimal context first):

**From spec.md:**

- Functional Requirements (FR-XXX)
- User Stories with acceptance scenarios
- Success Criteria (SC-XXX)
- Edge Cases
- Key Entities (if present)

**From plan.md (if exists):**

- Tech stack and architecture decisions
- File structure and component organization
- Phase breakdown

**From data-model.md (if exists):**

- Entities with fields, types, relationships
- Constraints and validation rules

**From contracts/ (if exists):**

- API endpoint definitions (OpenAPI, GraphQL, etc.)
- Request/response schemas

**From constitution (if exists):**

- Load `/memory/constitution.md` for principle validation

### 3. Analyze Implementation

Examine the codebase to build an implementation inventory:

- **Implemented files**: Identify source files created or modified during implementation
- **Implemented features**: Map code to functional requirements by:
  - Searching for FR-XXX references in comments/docstrings
  - Matching function/class names to requirement keywords
  - Tracing task completion markers in tasks.md to corresponding files
- **API endpoints**: Extract actual endpoint definitions and compare to contracts
- **Data models**: Extract entity implementations and compare to data-model.md
- **Tests**: Identify test files and map to acceptance scenarios

### 4. Build Traceability Matrix

Create an internal mapping (do not output raw mapping unless explicitly requested):

| Requirement | Task(s) | Implemented File(s) | Status |
|-------------|---------|---------------------|--------|
| FR-001      | T-001   | src/auth/login.py   | ✓ Covered |
| FR-002      | T-003   | (none found)        | ✗ Missing |

Track coverage for:

- Functional Requirements → Tasks → Code
- User Stories → Acceptance Scenarios → Tests
- Key Entities → Data Model → Implementation
- API Contracts → Endpoint Implementations

### 5. Detect Discrepancies

Compare normative artifacts against implementation. Categorize findings:

#### A. Implementation Gaps (code missing or incomplete)

- Requirements with no implementing code found
- Acceptance scenarios without corresponding test coverage
- API endpoints defined in contracts but not implemented
- Entities missing required fields or relationships

#### B. Spec Gaps (spec doesn't reflect intentional implementation decisions)

- Implemented features not described in spec
- API endpoints added beyond contract scope
- Data model fields added for technical reasons
- Behavior changes made during implementation

#### C. Constitution Violations

- Implementation patterns conflicting with MUST principles
- Security or quality attributes below mandated thresholds
- Architectural decisions violating stated constraints

#### D. Contract Mismatches

- Endpoint signatures differing from OpenAPI/GraphQL definitions
- Request/response shapes not matching schemas
- Missing or extra endpoints

### 6. Generate Verification Report

Output a structured Markdown report:

```markdown
## Verification Report: [FEATURE NAME]

### Summary

| Metric | Value |
|--------|-------|
| Functional Requirements | X/Y covered (Z%) |
| User Stories | X/Y implemented |
| API Endpoints | X/Y compliant |
| Data Model Entities | X/Y match |
| Constitution Violations | N |

### Requirement Coverage

| Req ID | Description | Status | Evidence |
|--------|-------------|--------|----------|
| FR-001 | [requirement text] | ✓ PASS | src/path/file.py |
| FR-002 | [requirement text] | ✗ MISSING | No implementation found |
| FR-003 | [requirement text] | ⚠ PARTIAL | Implemented but missing edge case handling |

### Discrepancies Found

#### Implementation Gaps (require code changes)

| ID | Severity | Description | Recommendation |
|----|----------|-------------|----------------|
| IG-001 | HIGH | FR-002 has no implementing code | Implement in src/... |

#### Spec Gaps (spec may need update)

| ID | Description | Implementation | Suggested Spec Update |
|----|-------------|----------------|----------------------|
| SG-001 | Extra validation added | src/validators.py | Add FR-XXX for input sanitization |

#### Constitution Violations (must be fixed in code)

| ID | Principle | Violation | Required Fix |
|----|-----------|-----------|--------------|
| CV-001 | Security-first | API lacks authentication | Add auth middleware |
```

### 7. Interactive Discrepancy Resolution

For each **Spec Gap** (one at a time, maximum 10 per session):

1. Present the discrepancy with context:
   - What the spec says (or doesn't say)
   - What the implementation does
   - Why this might be intentional (if discernible)

2. Ask the user:
   > "The implementation includes [feature/behavior] not described in the spec.
   > **Options:**
   > - **A**: Update spec to document this (it's intentional)
   > - **B**: Flag as implementation gap (it should be removed/changed)
   > - **Skip**: Defer decision for now
   >
   > Which would you like?"

3. If user chooses **A** (update spec):
   - Determine the appropriate spec section (Functional Requirements, Edge Cases, Key Entities, etc.)
   - Draft a concise addition (e.g., new FR-XXX or clarification bullet)
   - Present the proposed change for confirmation
   - On confirmation, apply the change to spec.md immediately (atomic write)

4. If user chooses **B** (implementation gap):
   - Add to the Implementation Gaps list with recommended fix

5. Continue until all spec gaps are resolved or user says "done"/"skip all"

### 8. Update Spec File (when changes accepted)

For each accepted spec update:

1. Ensure a `## Verification Notes` section exists (create after Success Criteria if missing)
2. Under it, create (if not present) a `### Session YYYY-MM-DD` subheading for today
3. Append a bullet: `- Verified: [brief description of what was confirmed/added]`
4. Apply the actual content change to the appropriate section:
   - New requirement → Add FR-XXX to Functional Requirements
   - New entity field → Update Key Entities
   - Behavior clarification → Add to Edge Cases or update User Story
   - API addition → Note in relevant section
5. Save spec.md after each integration (atomic overwrite)
6. Preserve formatting: do not reorder unrelated sections

### 9. Validation Pass

After all updates:

- Verify no duplicate requirements introduced
- Confirm FR-XXX numbering is sequential
- Check that Verification Notes session contains exactly one bullet per accepted change
- Validate Markdown structure remains intact

### 10. Final Report

Output completion summary:

```markdown
## Verification Complete

**Spec Updates Applied**: N changes to spec.md
**Implementation Gaps Identified**: M issues requiring code changes
**Constitution Violations**: P issues (must fix before release)

### Changes Made to spec.md
- Added FR-007: Input sanitization for user fields
- Updated Edge Cases: Added rate limiting behavior

### Remaining Action Items
1. [HIGH] Implement FR-002 (user notification system)
2. [CRITICAL] Add authentication to /api/admin endpoints (constitution violation)

### Suggested Next Steps
- Run `/speckit.implement` to address implementation gaps
- Review and merge updated spec.md
- Re-run `/speckit.verify` after fixes to confirm compliance
```

## Behavior Rules

- If no discrepancies found, output: "✓ Implementation fully complies with specification. No updates needed."
- If spec.md is missing, instruct user to run `/speckit.specify` first
- If tasks.md shows incomplete tasks, warn user and ask whether to proceed with partial verification
- Never accept a spec update that would violate constitution principles
- Respect user termination signals ("done", "stop", "skip all")
- Maximum 10 interactive resolution questions per session
- For implementation gaps, always provide actionable recommendations

## Context

{ARGS}
