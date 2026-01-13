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

## Outline

Goal: Validate delivered code against normative artifacts (`spec.md`, `plan.md`, `data-model.md`, `contracts/`) and update spec.md to reflect intentional implementation deviations. This command runs AFTER `/speckit.implement` has completed implementation tasks.

1. **Setup**: Run `{SCRIPT}` from repo root and parse JSON for FEATURE_DIR and AVAILABLE_DOCS. Derive absolute paths:
   - SPEC = FEATURE_DIR/spec.md
   - PLAN = FEATURE_DIR/plan.md
   - DATA_MODEL = FEATURE_DIR/data-model.md (if exists)
   - CONTRACTS = FEATURE_DIR/contracts/ (if exists)
   - TASKS = FEATURE_DIR/tasks.md
   - Abort with error if `spec.md` or `tasks.md` is missing (instruct user to run prerequisite commands).
   - For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Load normative artifacts** (progressive disclosure—minimal context first):
   - **From spec.md**: Functional Requirements (FR-XXX), Non-Functional Requirements, User Stories with acceptance scenarios, Data Requirements (DR-XXX), Edge Cases, Technical Constraints
   - **From plan.md**: Architecture/stack choices, File structure expectations, Phases and implementation approach, Component responsibilities
   - **From data-model.md (if exists)**: Entity definitions and attributes, Relationships and constraints, Type definitions
   - **From contracts/ (if exists)**: API endpoint specifications, Request/response schemas, Error codes and handling
   - **From constitution (if exists)**: Load `/memory/constitution.md` for principle validation
   - **From tasks.md**: Task IDs and descriptions, Completion status ([X] vs [ ]), File paths referenced, Phase grouping

3. **Discover implemented code**: Scan the workspace to identify implemented artifacts:
   - **File Discovery**: Use plan.md's file structure as primary guide, scan directories mentioned in tasks.md, identify source/test/config files
   - **Code Analysis**: Extract exported functions, classes, types, interfaces; identify API endpoints and routes; parse test files for coverage patterns; note error handling patterns

4. **Build traceability matrix**: Create mapping between requirements and implementation:

   | Requirement | Task(s) | File(s) | Status | Notes |
   |-------------|---------|---------|--------|-------|
   | FR-001 | T003, T005 | oee-display.tsx | ✓ Implemented | |
   | FR-002 | T004 | oee.ts | ✓ Implemented | |
   | FR-003 | T003 | oee-display.tsx | ? Partial | Missing loading state |

   **Status Categories**: ✓ Implemented (fully satisfies), ? Partial (exists but incomplete), ✗ Missing (no implementation), Δ Deviation (differs from spec)

   Track coverage for: Functional Requirements → Tasks → Code, User Stories → Acceptance Scenarios → Tests, Key Entities → Data Model → Implementation, API Contracts → Endpoint Implementations

5. **Compliance analysis**: Perform systematic verification across all normative dimensions:
   - **A. Functional Requirement Verification**: For each FR-XXX, locate implementing code, verify behavior matches description, check edge case handling, confirm test coverage
   - **B. Data Model Verification** (if data-model.md exists): Compare entity definitions against implemented types/schemas, verify attributes match, check relationships
   - **C. Contract Verification** (if contracts/ exists): Verify API endpoints match specs, check request/response schema compliance, confirm error handling
   - **D. Non-Functional Verification**: Performance targets, error handling states, accessibility implementations
   - **E. Test Coverage Verification**: Map test files to requirements, identify gaps, check acceptance scenario coverage

6. **Classify discrepancies**:

   | Type | Description | Resolution Path |
   |------|-------------|------------------|
   | **Spec Gap** | Implementation is correct but spec is incomplete/outdated | Update spec.md |
   | **Implementation Gap** | Spec is correct but implementation is missing/incomplete | Run `/speckit.implement` |
   | **Design Deviation** | Intentional change from spec requiring documentation | Update spec.md with rationale |
   | **Bug** | Implementation error that violates specification | Flag for fix |

7. **Generate verification report**: Output structured Markdown report with:
   - Feature name, verification date, spec path
   - Coverage summary table (category, total, verified, partial, missing, coverage %)
   - Traceability matrix (requirement, status, tasks, files)
   - Spec updates applied, implementation gaps found, deferred items
   - Verification metrics summary

8. **Interactive discrepancy resolution** (one at a time, maximum 10 total):
   - Present each discrepancy with: Requirement, Expected (Spec), Found (Implementation), Location
   - Provide **Recommended** option with reasoning
   - Options table: A (Spec Gap), B (Implementation Gap), C (Design Deviation), D (Skip)
   - After response: A → queue spec update, B → add to remediation list, C → queue spec update with rationale, D → mark deferred
   - Respect early termination signals ("done", "stop", "proceed")

9. **Spec update phase** (for Options A or C):
   - Ensure `## Verification Notes` section exists (create after `## Clarifications` if missing)
   - Add `### Verification YYYY-MM-DD` subheading for today's session
   - Append entry: `- **FR-XXX**: [requirement] → Implementation: [description], Resolution: [type], Rationale: [why acceptable]`
   - Save spec.md immediately after each update (atomic write)
   - Preserve existing formatting and content

10. **Report completion**: Based on results:
    - If implementation gaps exist: Recommend running `/speckit.implement` then re-run `/speckit.verify`
    - If all verified: Report "Verification Complete ✓", suggest `/speckit.checklist`
    - If critical gaps: Flag as "Critical Issues Found", prioritize fix order

Behavior rules:

- **Interactive Discrepancy Resolution**: For each mismatch, ask user whether it's a spec gap (update spec) or implementation gap (flag for remediation). Follow the one-question-at-a-time pattern from `/speckit.clarify`.
- **Atomic Writes**: Update spec.md incrementally after each confirmed deviation, preserving formatting.
- **Remediation Handoff**: If implementation gaps identified, recommend running `/speckit.implement`.
- **Constitution Authority**: The project constitution (`/memory/constitution.md`) is **non-negotiable**. Deviations violating constitution principles MUST be flagged as implementation gaps—never accepted as spec updates.
- **Progressive Loading**: Load code context only as needed for each discrepancy.
- **Token-Efficient Output**: Summarize large code blocks rather than dumping full content.
- **Deterministic Results**: Rerunning without changes should produce consistent findings.
- NEVER fabricate implementation details—only report what exists in code.
- NEVER modify implementation code—this command only updates spec.md.
- NEVER skip required validations—all FR-XXX must be checked.
- NEVER hallucinate coverage—if code doesn't exist, report it missing.

Context for verification: {ARGS}
