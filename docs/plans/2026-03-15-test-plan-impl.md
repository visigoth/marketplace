# test-plan Skill Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create the `test-plan` skill for the starchitect plugin — the final stage of the pipeline that produces test specifications from PRDs, contracts, and the existing task hierarchy.

**Architecture:** Single SKILL.md file following the starchitect convention (frontmatter, HARD-GATEs, phased pipeline with lazy loading and user review). Split output: compact index + per-feature detail files. Updates to existing implementation tasks via `bd update`, plus new test tasks via `bd create`.

**Tech Stack:** Markdown skill definition, `bd` CLI for issue management, `bv` for validation.

---

### Task 1: Create SKILL.md — Frontmatter and Introduction

**Files:**
- Create: `plugins/starchitect/skills/test-plan/SKILL.md`

**Step 1: Write the frontmatter, intro, HARD-GATE, and checklist**

Follow the pattern from beadify/SKILL.md:
- Frontmatter: `name: starchitect:test-plan`, `user-invocable: true`, description with trigger words ("test plan", "test strategy", "testing", "write tests for")
- One-paragraph intro: what the skill does (produces test specifications from PRDs, contracts, and task hierarchies)
- Primary HARD-GATE: do NOT skip to writing — every test specification must be presented for user review
- Checklist with the 6 phases from the design doc

**Step 2: Commit**

```bash
git add plugins/starchitect/skills/test-plan/SKILL.md
git commit -m "feat(starchitect): add test-plan skill skeleton"
```

---

### Task 2: Write Phase 0 — Discover & Scope

**Files:**
- Modify: `plugins/starchitect/skills/test-plan/SKILL.md`

**Step 1: Write Phase 0**

This phase mirrors beadify Phase 0 with additions for test context:

1. **Check for beads** — same pattern as beadify (check `.beads/`, stop if not found)
2. **Load the feature index** — same locations as beadify (`docs/features/index.{org,md}`)
3. **Check for existing test plan** — search `docs/test-plan.{org,md}` and `docs/test-plan/` directory. If found, summarize current state. If not, note this is the first run (index will be created).
4. **Query `bd` for existing hierarchy** — list epics, features, and tasks. Specifically look for tasks with `test:*` labels to identify what's already test-planned.
5. **Determine scope** — follow dependency order (like beadify). First epic with un-test-planned features is the recommendation. User can override to epic/feature/task scope.
6. **Present scope recommendation** — show what's done, what's next, let user confirm.
7. HARD-GATE before proceeding.

**Step 2: Commit**

```bash
git add plugins/starchitect/skills/test-plan/SKILL.md
git commit -m "feat(starchitect): test-plan Phase 0 — discover and scope"
```

---

### Task 3: Write Phase 1 — Analyze Test Needs

**Files:**
- Modify: `plugins/starchitect/skills/test-plan/SKILL.md`

**Step 1: Write Phase 1**

Per feature (lazy-load docs):

1. **Load feature PRD** — from `docs/features/<feature-name>.{org,md}`. Extract FRs, COMP identifiers, contract references.
2. **Load supporting documents on-demand** — same table pattern as beadify (contracts index → detail files, floorplan, technology choices). Only load sections relevant to this feature.
3. **Load implementation tasks** — `bd list --labels "ft:FTY" --type task --json`. These are the tasks from beadify.
4. **Identify applicable test types** — core four are always present. Check for conditional signals:
   - Contract tests: multiple COMPs implement the same API boundary
   - Performance tests: PRD has throughput/latency AC constraints
   - Security tests: ENT sensitivity flags exist, API auth fields defined
5. **Present activated test types** — table showing which types activate and why. User confirms/adjusts.
6. HARD-GATE before proceeding.

**Step 2: Commit**

```bash
git add plugins/starchitect/skills/test-plan/SKILL.md
git commit -m "feat(starchitect): test-plan Phase 1 — analyze test needs"
```

---

### Task 4: Write Phase 2 — Produce Test Specifications

**Files:**
- Modify: `plugins/starchitect/skills/test-plan/SKILL.md`

**Step 1: Write Phase 2 — unit test specifications**

For each implementation task:
- Draft test scenario descriptions with contract references
- Format: natural language scenario + the ENT/API/EVT identifiers being validated
- Example: "Test that creating a user with duplicate email returns 409 Conflict (API1.2 error case, ENT1.email uniqueness constraint)"
- These will be added to the task's description field via `bd update`

**Step 2: Write Phase 2 — integration/e2e/UX test task specifications**

For tests that span COMPs or need different environments, produce separate task specs:
- Title, type, priority, labels (including `test:<type>`), external-ref
- Description with contract/PRD references
- Acceptance criteria: specific test scenarios with expected outcomes
- Design: pointers to test-plan feature doc, contracts, swim lanes
- Dependencies: which implementation tasks this test task blocks on

**Step 3: Write Phase 2 — coverage matrix**

After all specs for a feature:
- Build FR → implementation task → test coverage matrix
- Flag any FRs without test coverage
- Every FR sub-item must have at least unit test coverage via its implementation task

**Step 4: Write Phase 2 — complexity heuristics and feature doc candidates**

Include the heuristics table from the design doc:
- 3+ COMPs → document in feature file
- Async events (EVT) → document setup/ordering
- PRD-called-out edge cases → document
- UX test scenarios → always document
- Routine CRUD → task-only

**Step 5: Write the review checkpoint**

Present per-feature:
1. Unit test specs for each implementation task
2. Integration/e2e/UX test task specs
3. Coverage matrix
4. Which scenarios are candidates for the feature doc

User can adjust before confirming. HARD-GATE.

**Step 6: Commit**

```bash
git add plugins/starchitect/skills/test-plan/SKILL.md
git commit -m "feat(starchitect): test-plan Phase 2 — produce test specifications"
```

---

### Task 5: Write Phase 3 — Write Documents

**Files:**
- Modify: `plugins/starchitect/skills/test-plan/SKILL.md`

**Step 1: Write index document structure**

The index (`docs/test-plan.{org,md}`) contains:
- Testing philosophy and coverage expectations
- Per-feature coverage summary table (feature → test types activated → task count → status)
- Risk-based testing priorities (derived from task priority + complexity heuristics)
- Links to feature detail files

Include both org-mode and markdown templates (like contracts skill).

**Step 2: Write feature detail file structure**

Per-feature files (`docs/test-plan/<feature-name>.{org,md}`) contain only non-obvious scenarios:
- Complex multi-COMP integration test setups
- E2E user journeys (derived from UC/UJ + SL diagrams)
- UX test scripts
- Async event flow testing (EVT setup/ordering)
- PRD-called-out edge cases

Each scenario: title, description, which COMPs/contracts involved, setup requirements, expected behavior.

**Step 3: Write the output format determination**

Same pattern as contracts: scan `docs/` for `.org` vs `.md` files, tell user which format.

**Step 4: Write the scoping behavior**

- First run: create index + feature file(s) for scope
- Subsequent runs: update feature file + index coverage table
- Task scope: no document output

**Step 5: Present-and-write flow with HARD-GATE**

Present documents for review before writing to disk.

**Step 6: Commit**

```bash
git add plugins/starchitect/skills/test-plan/SKILL.md
git commit -m "feat(starchitect): test-plan Phase 3 — write documents"
```

---

### Task 6: Write Phase 4 — Write to `bd`

**Files:**
- Modify: `plugins/starchitect/skills/test-plan/SKILL.md`

**Step 1: Write the `bd update` flow for unit test specs**

For each implementation task that received unit test specs:

```bash
bd update [task-id] --description "[updated description with test scenarios appended]"
```

Show the pattern: append test scenarios to existing description, don't overwrite.

**Step 2: Write the `bd create` flow for test tasks**

For each integration/e2e/UX test task:

```bash
bd create --type task --title "[test task title]" \
  --parent [feature-issue-id] \
  --labels "ep:EPX,ft:FTY,test:integration,comp:COMPN" \
  --priority [P0-P4] \
  --description "[test description]" \
  --silent

bd update [test-task-id] --acceptance-criteria "[test scenarios]"
bd update [test-task-id] --design "[reference pointers]"
```

**Step 3: Write the dependency creation flow**

```bash
bd dep add [test-task-id] [impl-task-id] \
  --type blocks \
  --metadata '{"strength": "hard", "reason": "test requires implementation complete"}'
```

**Step 4: Write the commit checkpoint and HARD-GATE**

Same pattern as beadify: present count of updates/creates, let user confirm before writing.

**Step 5: Write the success report**

Report: tasks updated, test tasks created, dependencies added, any issues.

**Step 6: Commit**

```bash
git add plugins/starchitect/skills/test-plan/SKILL.md
git commit -m "feat(starchitect): test-plan Phase 4 — write to bd"
```

---

### Task 7: Write Phase 5 — Validate, Constraints, and Closing

**Files:**
- Modify: `plugins/starchitect/skills/test-plan/SKILL.md`

**Step 1: Write Phase 5 — validation**

Offer `bv --robot-plan` to validate dependency graph after writing. Report cycles, orphans, coverage gaps.

**Step 2: Write the "suggest next steps" section**

- Run `bv --robot-priority` for task ordering
- Use `bd ready` to find unblocked tasks
- Assign tasks to agents
- Run test-plan again for the next epic

**Step 3: Write the "Important Constraints" section**

- Output is `bd` updates/issues and test plan documents only
- Do NOT write test code, test frameworks, or CI configuration
- Lazy-load per-feature
- Do NOT create test specs without user review
- Do NOT invent test scenarios — trace back to FRs, contracts, floorplan
- When a feature already has test tasks (tasks with `test:*` labels), skip unless user asks to re-plan
- Test infrastructure is out of scope (floorplan/tech-plan territory)

**Step 4: Commit**

```bash
git add plugins/starchitect/skills/test-plan/SKILL.md
git commit -m "feat(starchitect): test-plan Phase 5 — validate and constraints"
```

---

### Task 8: Update supporting files

**Files:**
- Modify: `AGENTS.md`
- Modify: `CHANGELOG.md`

**Step 1: Update AGENTS.md**

Update the starchitect description in the Current Plugins table to include test-plan in the pipeline:
```
Pipeline: PRD generation (prd-create) → architectural floorplans (floorplan) → feature decomposition (prd-feature-breakdown) → entity/API/protocol contracts (contracts) → task decomposition (beadify) → test planning (test-plan). Also includes technology research (tech-plan).
```

**Step 2: Bump version and update CHANGELOG**

Run `./scripts/release.sh starchitect minor` to bump to 0.7.0.

Add CHANGELOG entry for 0.7.0 with the test-plan skill addition.

**Step 3: Commit and push**

```bash
git add AGENTS.md CHANGELOG.md plugins/starchitect/.claude-plugin/plugin.json .claude-plugin/marketplace.json
git commit -m "chore(release): starchitect 0.7.0"
git push origin main
```
