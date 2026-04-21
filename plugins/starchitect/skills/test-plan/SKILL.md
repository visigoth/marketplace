---
name: starchitect:test-plan
description: >
  Produce test specifications from PRDs, contracts, and task hierarchies. Adds unit test specs
  to existing implementation tasks and creates separate test tasks for integration, e2e, and UX tests.
  Reads the feature index, then lazily loads feature PRDs, contracts, floorplan, and task issues as needed.
  Triggers: "test plan", "test strategy", "testing", "write tests for", "test specs",
  "test coverage", "add tests".
user-invocable: true
---

# test-plan: Task Hierarchies to Test Specifications

Produce test specifications from PRDs, contracts, and task hierarchies in `bd` (beads). For each feature, analyze functional requirements and contracts to determine what needs testing, then add unit test specs directly to existing implementation tasks and create separate test tasks for integration, e2e, and UX tests. The result is a complete test plan with traceability from every FR back to specific test specifications.

<HARD-GATE>
Do NOT skip to writing — every test specification must be presented to the user for review and confirmation before writing to `bd` or documents. Do NOT load all documents upfront — load lazily as each feature is visited.
</HARD-GATE>

## Autopilot Mode

When invoked with the word **"autopilot"** (e.g., "test plan on autopilot", "test plan autopilot"), all confirmation gates below become **soft**: the skill still presents its output at each checkpoint but proceeds immediately without waiting for user confirmation. The user can interrupt at any point to adjust.

Autopilot does NOT skip content — it skips waiting. You still show your work at every gate.

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Discover & scope** — check for beads, load feature index, check for existing test plan, determine scope
2. **Analyze test needs** — per feature: lazy-load docs, identify test types (unit, integration, e2e, UX), present for confirmation
3. **Produce test specifications** — per feature: unit test specs for impl tasks, test task specs for integration/e2e/UX, coverage matrix
4. **Write documents** — create/update test plan index and per-feature detail files
5. **Write to bd** — update impl tasks with unit test specs, create test tasks for integration/e2e/UX, add dependencies
6. **Validate** — offer `bv --robot-plan` validation, suggest next steps

---

## Phase 0: Discover & Scope

### Check for beads

Check for a `.beads/` directory in the project root.

If not found:
- Tell the user: "No beads workspace found. Run `bd init` to initialize one, then re-run this skill."
- Stop here.

### Load the feature index

Search for the feature index:

| Document | Locations to check |
|----------|--------------------|
| **Feature index** | `docs/features/index.org`, `docs/features/index.md` |

Use Glob to check both locations. Read the one that exists.

If not found:
- Tell the user: "No feature index found. Run the prd-feature-breakdown skill first to create one."
- Stop here.

**Do NOT load feature PRDs, contracts, floorplan, or technology choices at this point.** The feature index contains the epic/feature hierarchy, dependency graph, and implementation phases — that is sufficient for scoping.

### Check for existing test plan

Search for existing test plan artifacts:

| Document | Locations to check |
|----------|--------------------|
| **Test plan index** | `docs/test-plan.org`, `docs/test-plan.md` |
| **Test plan directory** | `docs/test-plan/` |

Use Glob to check all locations.

If found:
- Read the test plan index and summarize current state: which features have been test-planned, which test types are covered, and any gaps noted.

If not found:
- Note this is the first run. No existing test plan to build on.

### Query `bd` for existing hierarchy

Query `bd` for existing issues to understand what has been taskified and what already has test coverage:

```bash
bd list --type epic --json
bd list --type feature --json
bd list --type task --json
```

From the results, specifically look for tasks with `test:*` labels. A feature is considered **test-planned** if it has at least one task with a `test:` label (e.g., `test:unit`, `test:integration`, `test:e2e`, `test:ux`).

### Determine scope

Using the feature index's implementation ordering (suggested phases and dependency graph):

1. Walk the epics in dependency order
2. For each epic, check if all its features are test-planned
3. The first epic with un-test-planned features is the recommendation
4. The user can override to: a specific epic, a specific feature, or a specific task

### Present scope recommendation

Present to the user:

- Which features are fully test-planned (if any)
- Which epic is recommended next and why (dependency order)
- Which features within that epic need test planning
- Let the user confirm or override the recommended scope

<HARD-GATE>
Do NOT proceed to Phase 1 until the user has confirmed the scope.

**Autopilot:** Present the scope recommendation, then proceed immediately.
</HARD-GATE>

---

## Phase 1: Analyze Test Needs

Work through each in-scope feature one at a time.

### Step 1: Load the feature PRD

Load the feature's PRD from `docs/features/<feature-name>.{org,md}`.

Read its contents and identify:
- The FRs assigned to this feature (and their sub-items)
- The COMP identifiers this feature touches
- The contract elements referenced (ENT, API, EVT identifiers)
- Dependencies on other features

### Step 2: Load supporting documents (on-demand)

Load **only** the sections of these documents that the feature PRD references:

| Document | Load when | What to extract |
|----------|-----------|-----------------|
| **Contracts index** (`docs/contracts.{org,md}`) | Feature references ENT, API, or EVT identifiers | Load the compact index first for identifier lookup; then load specific detail files from `docs/contracts/` (e.g., `entities.{org,md}` for ENT schemas, `<api-name>.{org,md}` for API operations) as needed |
| **Floorplan** (`docs/floorplan.{org,md}`) | Feature references COMP, BD, SL, or DF identifiers | Component boundaries, relevant block diagram edges, swim-lane interactions |
| **Technology choices** (`docs/technology.{org,md}`) | Feature's COMPs have technology decisions | Tech stack for the relevant components, **plus** production container image requirements and devcontainer configuration sections |

Do NOT read these documents in full. Read them and extract only the sections relevant to the current feature's identifiers.

#### Review container and devcontainer configuration for testing adequacy

When loading technology choices, **always** check the "Production Container Image Requirements" and "Development Container (Devcontainer) Configuration" sections (if they exist). Assess whether:

1. **Devcontainer services cover test dependencies**: every database, queue, cache, or external service that integration/e2e tests will need must appear in the devcontainer's "Services for Testing" table. If a feature introduces a new service dependency (e.g., a feature uses Redis but Redis is absent from the devcontainer config), flag this as a gap.

2. **Test tooling is available**: the test runners, assertion libraries, and coverage tools needed for the feature's test types are either installable via the language toolchain listed in the devcontainer or explicitly listed as build tools.

3. **Production image constraints are testable**: if the production image has specific requirements (e.g., distroless, no shell, non-root), note any implications for e2e or integration tests that might need to run against the production image (e.g., smoke tests in CI).

If gaps are found, include them in the test type presentation (Phase 1, Step 5) as action items: "The devcontainer configuration in `docs/technology.{org,md}` should be updated to include [service/tool] to support [test type] for this feature." These are recommendations — the test-plan skill does not modify the technology choices document.

### Step 3: Load implementation tasks

Query `bd` for tasks associated with this feature:

```bash
bd list --labels "ft:FTY" --type task --json
```

These are the implementation tasks from beadify. Extract: task IDs, titles, COMP labels, FR labels, acceptance criteria, design pointers.

### Step 4: Identify applicable test types

Determine which test types apply to this feature. There are two categories: core (always present) and conditional (activated by signals in the inputs).

**Core test types** (always present for every feature):

| Test type | What it validates | Derived from |
|-----------|-------------------|--------------|
| Unit tests | Individual functions/methods within a COMP | Task acceptance criteria, ENT schemas |
| Integration tests | API boundaries between COMPs work correctly | Contracts (API operations, ENT schemas) |
| E2E tests | Full user journeys work end-to-end | PRD use cases (UC/UJ), swim-lane diagrams (SL) |
| UX tests | UI behavior, accessibility, visual correctness | PRD use cases, feature PRDs |

**Conditional test types** (surface when inputs have signals):

| Test type | Activates when | Derived from |
|-----------|---------------|--------------|
| Contract tests | Multiple COMPs implement the same API boundary | Contracts (API boundary definitions) |
| Performance tests | PRD has throughput/latency AC constraints | PRD non-functional requirements |
| Security tests | ENT sensitivity flags or auth requirements exist | Contracts (ENT sensitivity, API auth fields) |
| Environment tests | Production image requirements or devcontainer config exists in technology choices | Technology choices (container image requirements, devcontainer configuration) |

Scan the feature PRD, contracts, and task acceptance criteria for the activation signals listed above. A conditional test type is activated only when its signal is present — do not assume it applies without evidence.

### Step 5: Present activated test types

Show a table of which test types are activated for this feature and why:

| Test type | Status | Rationale |
|-----------|--------|-----------|
| Unit tests | ✓ Active (core) | Always present |
| Integration tests | ✓ Active (core) | Always present |
| E2E tests | ✓ Active (core) | Always present |
| UX tests | ✓ Active (core) | Always present |
| Contract tests | ✓ Active | COMP3 and COMP5 both implement API2 boundary |
| Performance tests | ✗ Inactive | No throughput/latency constraints in PRD |
| Security tests | ✓ Active | ENT1 has sensitivity=PII, ENT3 has sensitivity=financial |

For each activated conditional type, cite the specific signal that activated it (identifiers, constraint text, or sensitivity flags). For inactive conditional types, briefly note why the signal was not found.

Let the user confirm or adjust the activated test types. The user may:
- Add a conditional test type that was not activated (override)
- Remove a core or conditional test type (with acknowledgment of reduced coverage)
- Adjust the rationale

### Step 6: HARD-GATE

<HARD-GATE>
Do NOT proceed to Phase 2 until the user has confirmed the activated test types for this feature.

**Autopilot:** Present the activated test types, then proceed immediately.
</HARD-GATE>

---

## Phase 2: Produce Test Specifications

For each in-scope feature, after test types are confirmed, produce test specifications.

### Step 1: Produce unit test specifications

For each implementation task in the feature, draft test scenario descriptions with contract references.

Format each scenario as: natural language scenario + the ENT/API/EVT identifiers being validated.

Example: "Test that creating a user with duplicate email returns 409 Conflict (API1.2 error case, ENT1.email uniqueness constraint)"

Cover these categories for each task:
- **Happy path**: the primary success scenario from the task's acceptance criteria
- **Error cases**: failure modes from the API operations and ENT constraints the task implements
- **Edge cases**: boundary conditions called out in the PRD acceptance criteria or contract schemas (e.g., max-length fields, empty collections, concurrent modifications)

These scenarios will be added to the task's description field via `bd update` in Phase 4.

Present as a table or list per implementation task showing the scenario descriptions:

| Implementation task | Scenario | Contract refs |
|---------------------|----------|---------------|
| "Implement user model" | Test that user creation with valid fields persists and returns the entity | ENT1 schema |
| "Implement user model" | Test that duplicate email is rejected | ENT1.email uniqueness |
| "Implement user model" | Test that email exceeding max length is rejected | ENT1.email maxLength |
| "Implement user endpoint" | Test that POST /users with valid body returns 201 with ENT1-shaped response | API1.2 success, ENT1 |
| "Implement user endpoint" | Test that POST /users with missing required fields returns 400 | API1.2 error case |

### Step 2: Produce integration/e2e/UX test task specifications

For tests that span COMPs or need different environments, produce separate task specs. Each test task gets:

- **Title**: descriptive, prefixed with test type (e.g., "Integration Test: API1 user service boundary", "E2E Test: user registration flow", "UX Test: dashboard accessibility")
- **Type**: `task`
- **Priority**: inherit from the highest-priority implementation task it depends on
- **Labels**: `ep:EPX`, `ft:FTY`, `test:<type>` (e.g., `test:integration`, `test:e2e`, `test:ux`), `comp:COMPN` (if COMP-scoped)
- **Description**: what's being tested and why, with contract/PRD references
- **Acceptance criteria**: specific test scenarios with expected outcomes, referencing contract elements (ENT, API, EVT identifiers)
- **Design**: reference pointers to test-plan feature doc section, contracts, swim lanes
- **Dependencies**: which implementation tasks this test task blocks on (hard dependency)

For each test type, the following drives scenario identification:

**Integration tests**: each API boundary (API identifier) between COMPs in the feature needs at least one integration test task. Scenarios come from API operations + error cases. Walk the floorplan's block diagram edges (BD) for the feature's COMPs — each edge with an API label is a candidate integration test boundary.

**E2E tests**: each UC/UJ in the feature PRD that involves this feature gets an e2e test task. Scenarios come from swim-lane diagrams (SL) — each message in the SL maps to a test step. The full sequence of messages in an SL diagram becomes the test's step-by-step flow.

**UX tests**: UI-facing features get UX test tasks. Scenarios cover interaction flows, accessibility, visual correctness. These are derived from the feature PRD's user-facing acceptance criteria and any UI wireframes or mockups referenced.

**Contract tests** (if activated): verify both sides of an API boundary agree on schemas. Produce one test task per shared API boundary where multiple COMPs implement or consume the same API identifier.

**Performance tests** (if activated): verify throughput/latency constraints from PRD AC items. Produce one test task per constraint, referencing the specific PRD non-functional requirement.

**Security tests** (if activated): verify auth, input validation, sensitive data handling for ENTs with sensitivity flags. Produce one test task per security concern, referencing the ENT sensitivity flags and API auth fields from contracts.

**Environment tests** (if activated): verify that the production container image and devcontainer configuration satisfy the requirements from the technology choices document. Scenarios include:
- **Production image smoke tests**: the built image starts, passes health checks, and serves traffic with only the declared runtime dependencies (validates the "Production Container Image Requirements" section)
- **Devcontainer completeness tests**: a fresh devcontainer build succeeds, all declared services are reachable, and the full test suite passes (validates the "Devcontainer Configuration" section)
- **Parity checks**: language runtime versions, system dependencies, and service versions match between the devcontainer and production image where applicable

Produce one test task for production image validation and one for devcontainer validation. These are typically CI-level tasks rather than feature-scoped, so attach them to the epic level if no single feature owns them.

### Step 3: Build coverage matrix

After all specs for the feature, build a coverage matrix mapping every FR sub-item to its test coverage:

| FR Sub-item | Implementation task | Unit test specs | Integration/E2E/UX test task |
|-------------|--------------------|-----------------|-----------------------------|
| FR3.1 | "Implement user model" | 3 scenarios | Integration: API1 boundary |
| FR3.2 | "Implement user endpoint" | 5 scenarios | E2E: registration flow |
| FR3.3 | ⚠ NOT COVERED | — | — |

**Every FR and FR sub-item must have at least unit test coverage through its implementation task.** If gaps exist, add test scenarios to cover them or flag the gap for discussion with the user.

### Step 4: Identify feature-doc candidates

Use these heuristics to determine which scenarios warrant a feature-doc entry vs. staying task-only:

| Signal | Implication |
|--------|-------------|
| 3+ COMPs involved in a test scenario | Likely complex, document in feature file |
| Async events (EVT) in the flow | Needs setup/ordering documentation |
| PRD calls out specific edge cases or error flows | Worth documenting |
| UX test scenarios | Always documented (often manual or semi-automated) |
| Routine CRUD against a single API boundary | Task-only, not documented in feature file |

Present which scenarios are candidates for the feature doc and which are task-only:

- **Feature-doc candidates**: list each scenario with the signal that qualifies it
- **Task-only**: list scenarios that stay in the task description only

### Step 5: Review checkpoint

Present per-feature:
1. Unit test specs for each implementation task (with scenario descriptions)
2. Integration/e2e/UX test task specs (with full task fields)
3. Coverage matrix
4. Feature-doc candidates

Offer commit checkpoint (following the beadify pattern):

"Confirm test specs for **FTY — [feature name]**? (**N unit test scenarios** across **M impl tasks**, **K new test tasks**, **L feature-doc scenarios**. **P features** remaining in EPX.)"

The user can:
- **Confirm** — proceed to the next feature or to Phase 3 if all features are done
- **Adjust** — modify test specs, add/remove scenarios, change task fields
- **Defer** — hold specs; continue to the next feature

### Step 6: Repeat for each feature

Move to the next in-scope feature and repeat from Phase 1, Step 1 (lazy-load that feature's documents fresh).

After reviewing all features in the epic, if there are unconfirmed test specs, prompt one final time:

"**N unit test scenarios** and **K test tasks** across **M features** are ready for **EPX — [epic name]**. Confirm all now?"

<HARD-GATE>
Do NOT proceed to Phase 3 until the user has confirmed test specs for all in-scope features.

**Autopilot:** Present the test specs, then proceed immediately.
</HARD-GATE>

---

## Phase 3: Write Documents

### Step 1: Determine output format

Scan `docs/` to determine the document format:

- Glob for `docs/**/*.org` and `docs/**/*.md`
- If org-mode files are present, use `.org`; otherwise use `.md`
- Tell the user which format you'll use

### Step 2: Create or update documents

Output is always a split structure — a compact index file plus per-feature detail files:

```
docs/test-plan.{org,md}              ← compact index (strategy + coverage dashboard)
docs/test-plan/
  <feature-name>.{org,md}            ← one file per feature (non-obvious scenarios only)
```

#### Index document structure

The index (`docs/test-plan.{org,md}`) is compact — strategy + coverage dashboard.

Contents:
- **Overview**: brief summary, links to source docs (PRD, floorplan, contracts)
- **Testing philosophy**: coverage expectations per test type (e.g., "every API operation has at least one happy-path and one error-path integration test")
- **Risk-based testing priorities**: which areas need most attention and why (derived from task priority + complexity heuristics)
- **Per-feature coverage summary table**:

| Feature | Unit | Integration | E2E | UX | Other | Detail |
|---------|------|-------------|-----|----|-------|--------|
| FT1: User Auth | 12 scenarios | 2 tasks | 1 task | 1 task | security: 1 | [link to detail file] |
| FT2: Dashboard | 8 scenarios | 1 task | — | 2 tasks | — | [link to detail file] |

#### Feature detail file structure

Per-feature files (`docs/test-plan/<feature-name>.{org,md}`) contain ONLY non-obvious scenarios. Routine unit/integration tests are NOT enumerated — those live in the `bd` task descriptions.

Each documented scenario includes:
- **Title**
- **Test type** (integration, e2e, UX, etc.)
- **Description**: what's being tested and why
- **COMPs involved**
- **Contract/PRD references** (ENT, API, EVT, UC/UJ, SL identifiers)
- **Setup requirements** (if any)
- **Expected behavior**
- **Why this is documented** (which complexity heuristic triggered it)

### Step 3: Scoping behavior

- **First run (any scope)**: create index + feature file(s) for whatever's in scope
- **Subsequent runs**: update the relevant feature file and the index's coverage table. Don't recreate the index strategy sections.
- **Task scope**: no document output (just bd updates in Phase 4)

### Step 4: Present and write

Present the documents for review. If the user has already confirmed individual features in Phase 2, only present the index (strategy sections + coverage table) as new material.

Prompt the user: "Would you like to review the test plan documents before I write them, or should I go ahead and write to disk?"

If the user wants to review, present them and wait for approval. If they choose to skip, write directly. **Autopilot:** skip review and write directly.

---

## Phase 4: Write to `bd`

When user confirms a commit (either per-feature from Phase 2 or batch):

### Step 1: Update implementation tasks with unit test specs

For each implementation task that received unit test specs, append the test scenarios to the task's description.

First, read the existing description:

```bash
bd show [task-id] --json
```

Then append a "## Test Scenarios" section to the existing description. Do NOT overwrite the existing description — preserve it and append the new section.

```bash
bd update [task-id] --description "[existing description + appended test scenarios section]"
```

Format the appended section:

```
## Test Scenarios

- [scenario description] (validates [ENT/API/EVT identifier])
- [scenario description] (validates [ENT/API/EVT identifier])
```

### Step 2: Create test tasks

For each integration/e2e/UX test task, follow a two-step create+update pattern (same as beadify — `bd create` doesn't support `--acceptance-criteria` or `--design` flags):

```bash
# Step 1: Create the issue
bd create --type task --title "[test task title]" \
  --parent [feature-issue-id] \
  --labels "ep:EPX,ft:FTY,test:integration,comp:COMPN" \
  --external-ref "FRZ" \
  --priority [P0-P4] \
  --description "[test description]" \
  --silent

# Step 2: Set acceptance criteria and design (not available on bd create)
bd update [test-task-id] --acceptance-criteria "[test scenarios with expected outcomes]"
bd update [test-task-id] --design "[reference pointers to test-plan doc, contracts, swim lanes]"
```

### Step 3: Add dependencies

For each test task, add a blocks dependency on the implementation tasks it requires:

```bash
bd dep add [test-task-id] [impl-task-id] \
  --type blocks \
  --metadata '{"strength": "hard", "reason": "test requires implementation complete"}'
```

For cross-feature test dependencies (e.g., an e2e test that spans features), follow the same fallback pattern as beadify: if the other feature's tasks exist, depend on the specific task; otherwise fall back to the feature-level issue.

```bash
bd dep add [test-task-id] [other-feature-task-or-issue-id] \
  --type blocks \
  --metadata '{"strength": "hard", "reason": "test requires cross-feature implementation complete"}'
```

### Step 4: Commit checkpoint

Present before writing:

"Write **N unit test spec updates** and **M new test tasks** with **K dependencies** for **FTY — [feature name]** to bd now?"

Prompt the user: "Would you like to review the bd issues before I write them, or should I go ahead and write?"

If the user wants to review, present them and wait for approval. If they choose to skip, write directly. **Autopilot:** skip review and write directly.

### Step 5: Confirm success

After writing, report:
- Number of implementation tasks updated with test scenarios
- Number of test tasks created (broken down by type)
- Number of dependencies added
- Any issues encountered

---

## Output Document Structure

Output is always split into a compact **index file** and a **detail directory**. Templates below show both org-mode and markdown variants. Only produce the format determined in Step 1.

### Index file template — org-mode (`docs/test-plan.org`):

```org
#+TITLE: Test Plan
#+DATE: YYYY-MM-DD

* Overview

Brief summary of the test plan.

PRD: [[file:prd.org][PRD]]
Floorplan: [[file:floorplan.org][Floorplan]]
Contracts: [[file:contracts.org][Contracts]]
Scope: System-wide | Epic: <epic-name> | Feature: <feature-name>

* Testing Philosophy

Coverage expectations per test type:

- *Unit tests*: every public function/method has at least happy-path + one error-path test
- *Integration tests*: every API operation has at least one happy-path and one error-path integration test
- *E2E tests*: every use case (UC/UJ) has at least one end-to-end test covering the full flow
- *UX tests*: every user-facing feature has accessibility + interaction flow tests
- (Conditional types as applicable: contract, performance, security, environment)
- *Environment tests* (when applicable): production image starts and passes health checks; devcontainer builds cleanly and runs the full test suite

* Risk-Based Testing Priorities

| Priority | Area | Rationale |
|----------+------+-----------|
| High | <area> | <why — derived from task priority + complexity heuristics> |
| Medium | <area> | <why> |
| Low | <area> | <why> |

* Per-Feature Coverage Summary

| Feature | Unit | Integration | E2E | UX | Other | Detail |
|---------+------+-------------+-----+----+-------+--------|
| FT1: <name> | N scenarios | N tasks | N tasks | N tasks | <type>: N | [[file:test-plan/<feature-name>.org]] |
| FT2: <name> | N scenarios | N tasks | — | N tasks | — | [[file:test-plan/<feature-name>.org]] |
```

### Index file template — markdown (`docs/test-plan.md`):

```markdown
# Test Plan

**PRD:** [PRD](prd.md)
**Floorplan:** [Floorplan](floorplan.md)
**Contracts:** [Contracts](contracts.md)
**Date:** YYYY-MM-DD
**Scope:** System-wide | Epic: <epic-name> | Feature: <feature-name>

## Overview

Brief summary of the test plan.

## Testing Philosophy

Coverage expectations per test type:

- **Unit tests**: every public function/method has at least happy-path + one error-path test
- **Integration tests**: every API operation has at least one happy-path and one error-path integration test
- **E2E tests**: every use case (UC/UJ) has at least one end-to-end test covering the full flow
- **UX tests**: every user-facing feature has accessibility + interaction flow tests
- (Conditional types as applicable: contract, performance, security, environment)
- **Environment tests** (when applicable): production image starts and passes health checks; devcontainer builds cleanly and runs the full test suite

## Risk-Based Testing Priorities

| Priority | Area | Rationale |
|----------|------|-----------|
| High | <area> | <why — derived from task priority + complexity heuristics> |
| Medium | <area> | <why> |
| Low | <area> | <why> |

## Per-Feature Coverage Summary

| Feature | Unit | Integration | E2E | UX | Other | Detail |
|---------|------|-------------|-----|----|-------|--------|
| FT1: <name> | N scenarios | N tasks | N tasks | N tasks | <type>: N | [<feature-name>.md](test-plan/<feature-name>.md) |
| FT2: <name> | N scenarios | N tasks | — | N tasks | — | [<feature-name>.md](test-plan/<feature-name>.md) |
```

### Detail file template — feature (`docs/test-plan/<feature-name>.{org,md}`):

Contains only non-obvious scenarios identified by the complexity heuristics in Phase 2, Step 4. Each scenario includes title, test type, description, COMPs involved, contract/PRD references, setup requirements, expected behavior, and which heuristic triggered documentation. Routine unit/integration tests are NOT included — those live in `bd` task descriptions.

---

## Phase 5: Validate

After all test specs for the epic are written to `bd`, offer:

"All test specifications for **EPX — [epic name]** are committed. Would you like me to run `bv --robot-plan` to validate the dependency graph?"

If the user confirms, run:

```bash
bv --robot-plan --label "ep:EPX" --format json
```

Report any issues found:
- Dependency cycles (especially between implementation and test tasks)
- Orphaned test tasks (no dependencies)
- Missing test coverage (features with no `test:*` labels)

### Suggest next steps

After validation (or if the user skips it):

- "Your test specifications are in bd. Next steps you might consider:"
  - Run `bv --robot-priority` to see recommended task ordering (implementation before tests)
  - Use `bd ready` to find tasks with no blockers
  - Assign tasks to agents with `bd update [id] --assignee [agent]`
  - Run this skill again for the next epic in dependency order
  - Use `bv` TUI for an interactive view of the task graph

---

## Important Constraints

- Your output is `bd` updates/issues and test plan documents ONLY
- Do NOT write test code, test frameworks, CI configuration, or any implementation artifacts
- Do NOT load all starchitect documents upfront — lazy-load per-feature to conserve context
- Do NOT create test specs without user review and confirmation
- Do NOT invent test scenarios — trace back to FRs, contracts, floorplan elements, and swim lanes. If coverage is incomplete, flag the gap rather than filling it with assumptions
- When a feature already has test tasks (tasks with `test:*` labels exist in `bd`), skip it unless the user explicitly asks to re-plan
- Test infrastructure (frameworks, environments, CI) is out of scope — that's floorplan and tech-plan territory
- Prefer precision over verbosity in test scenario descriptions — cite the specific contract elements, don't repeat the contract content
- Unit test specs augment existing implementation tasks; they do NOT create new tasks
- Integration, e2e, and UX test specs create new tasks as siblings of implementation tasks under the feature
