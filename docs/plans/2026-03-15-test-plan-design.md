# test-plan Skill Design

**Date:** 2026-03-15
**Status:** Approved

## Position in Pipeline

```
prd-create → floorplan → prd-feature-breakdown → contracts → bv-taskify → test-plan
```

Runs after bv-taskify. Reads the existing task hierarchy plus all upstream documents (PRDs, contracts, floorplan, technology choices). Can be scoped to epic, feature, or individual task level.

## Inputs

- Feature index and feature PRDs
- Contracts (index + detail files)
- Floorplan (COMP boundaries, BD edges, SL diagrams)
- Technology choices
- Existing `br` task hierarchy (epics, features, implementation tasks)
- Existing codebase (test frameworks, existing tests)

## Test Types

### Core four (always present)

| Test type | What it validates | Derived from |
|-----------|-------------------|--------------|
| Unit tests | Individual functions/methods within a COMP | Task acceptance criteria, ENT schemas |
| Integration tests | API boundaries between COMPs work correctly | Contracts (API operations, ENT schemas) |
| E2E tests | Full user journeys work end-to-end | PRD use cases (UC/UJ), swim-lane diagrams (SL) |
| UX tests | UI behavior, accessibility, visual correctness | PRD use cases, feature PRDs |

### Conditional (surface when inputs have signals)

| Test type | Activates when | Derived from |
|-----------|---------------|--------------|
| Contract tests | Multiple services implement the same API boundary | Contracts (API boundary definitions) |
| Performance tests | PRD has throughput/latency constraints | PRD non-functional requirements, AC constraints |
| Security tests | ENT sensitivity flags or auth requirements exist | Contracts (ENT sensitivity, API auth fields) |

The skill presents which test types are relevant and lets the user confirm/adjust before proceeding (starchitect interview pattern).

## Output: Documents

Split structure like contracts:

```
docs/test-plan.{org,md}              ← compact index (strategy + coverage dashboard)
docs/test-plan/
  <feature-name>.{org,md}            ← per-feature, complex scenarios only
```

### Index contains

- Testing philosophy and coverage expectations
- Per-feature coverage summary table (feature → test types → status)
- Risk-based testing priorities
- Links to feature detail files

### Feature files contain only non-obvious scenarios

Routine unit/integration tests are NOT enumerated — they're implied by acceptance criteria and contracts. Feature files document only:

- Multi-COMP integration test setups (3+ COMPs involved)
- E2E user journeys (derived from PRD UC/UJ + swim lanes)
- UX test scripts
- Tricky edge cases, race conditions, async event flow testing (EVT-related)
- Scenarios where the PRD calls out specific edge cases or error flows

### Not in scope for this document

Test infrastructure (frameworks, environments, CI configuration) — that's floorplan and tech-plan territory.

## Output: `br` Tasks

### Unit tests: added to existing implementation tasks

Unit test specifications are added to the implementation task's description field. Each spec is a test scenario description with contract references (e.g., "Test that API1.2 returns 409 when ENT1.email is duplicate per ENT1 uniqueness constraint").

bv-taskify already includes "New tests cover all acceptance criteria above" in acceptance criteria. test-plan adds the specifics of *what* tests to write.

### Integration, E2E, UX tests: separate `br` tasks

```
Feature (feature)
  ├── Impl Task A (task)
  ├── Impl Task B (task)
  └── Integration Test: API1 boundary (task)
        depends on: Impl Task A, Impl Task B (blocks, hard)
```

Test tasks are siblings of implementation tasks under the feature, with `blocks` dependencies on the implementation tasks they require. Feature can't close until all children (impl + test) are done.

### Test task fields

- **Type**: `task`
- **Labels**: `ep:EPX`, `ft:FTY`, `test:<type>` (e.g., `test:integration`, `test:e2e`, `test:ux`), `comp:COMPN` (if COMP-scoped)
- **Description**: what's being tested and why, with contract/PRD references
- **Acceptance criteria**: specific test scenarios with expected outcomes, referencing contract elements (ENT, API, EVT identifiers)
- **Design**: pointers to test-plan feature doc section, contracts, swim lanes
- **Dependencies**: blocks on implementation tasks that must exist first

## Phases

### Phase 0: Discover & Scope

- Load feature index, check `br` for existing tasks and test tasks
- Determine scope (epic/feature/task) — follow dependency order like bv-taskify
- Check for existing test plan documents
- Present recommendation to user

HARD-GATE: user confirms scope before proceeding.

### Phase 1: Analyze Test Needs

Per feature (lazy-load docs as needed):

- Load feature PRD, contracts, floorplan sections relevant to this feature
- Load existing implementation tasks from `br`
- Identify which test types apply (core four always, conditional based on signals)
- Present activated test types with reasoning; user confirms/adjusts

HARD-GATE: user confirms test types before proceeding.

### Phase 2: Produce Test Specifications

Per feature:

- **Unit test specs**: draft scenario descriptions for each implementation task, with contract references
- **Integration/E2E/UX test task specs**: draft separate task specifications
- **Coverage matrix**: map FRs → implementation tasks → test coverage
- Present for user review

Heuristics for what goes in the feature doc vs. stays task-only:
- 3+ COMPs involved → document in feature file
- Async events (EVT) → document setup/ordering
- PRD-called-out edge cases → document
- UX test scenarios → always document

HARD-GATE: user confirms test specs before proceeding.

### Phase 3: Write Documents

- Create/update test plan index
- Create/update per-feature test plan files
- Present for review before writing to disk

HARD-GATE: user approves before writing.

### Phase 4: Write to `br`

- Update implementation tasks with unit test specifications (via `br update`)
- Create test tasks for integration/e2e/UX tests
- Add `blocks` dependencies from test tasks to implementation tasks
- Report results

HARD-GATE: user confirms before writing to `br`.

### Phase 5: Validate

- Offer `bv --robot-plan` to check dependency graph integrity
- Report any issues (cycles, orphans, coverage gaps)
- Suggest next steps

## Scoping Behavior

- **First run (any scope)**: creates the index with strategy + coverage dashboard for whatever's in scope
- **Subsequent runs**: update the relevant feature file and the index's coverage table; don't recreate the index
- **Epic scope**: walks all features in the epic
- **Feature scope**: processes just that feature
- **Task scope**: adds unit test specs to a single implementation task (no document output, just `br` update)

## Complexity Heuristics

The skill uses these heuristics to seed which scenarios warrant feature-doc entries vs. staying task-only:

| Signal | Implication |
|--------|-------------|
| 3+ COMPs involved in a test scenario | Likely complex, document in feature file |
| Async events (EVT) in the flow | Needs setup/ordering documentation |
| PRD calls out specific edge cases or error flows | Worth documenting |
| UX test scenarios | Always documented (often manual or semi-automated) |
| Routine CRUD against a single API boundary | Task-only, not documented in feature file |

The user reviews and can adjust — "that's obvious, drop it" or "add a scenario for X."
