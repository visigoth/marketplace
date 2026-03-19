---
name: starchitect:bv-taskify
description: >
  Break features into implementation task hierarchies in bd (beads). Reads the feature index,
  then lazily loads feature PRDs, floorplan, contracts, and technology choices as needed.
  Produces component-scoped tasks with dependencies that expose parallelizable work for agents.
  Triggers: "taskify", "create tasks", "break into tasks", "implementation tasks",
  "task hierarchy", "decompose into tasks".
user-invocable: true
---

# bv-taskify: Features to Implementation Tasks

Decompose features into implementation task hierarchies in `bd` (beads). Each task is scoped to a single architectural component (COMP) so that agents can work in parallel without file conflicts. Contracts define the interfaces agents code against.

Your output is `bd` issues — epics, features, and tasks with structured fields, dependencies, and traceability back to the PRD, floorplan, and contracts.

<HARD-GATE>
Do NOT skip to creating tasks. Every task hierarchy must be presented to the user for review and confirmation before writing to `bd`. Do NOT load all documents upfront — load lazily as each feature is visited.
</HARD-GATE>

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Discover & scope** — check for beads, load feature index only, determine which epic to taskify next
2. **Decompose features into tasks** — per feature: lazy-load docs, analyze FRs + contracts + floorplan, produce component-scoped tasks
3. **Review & commit** — present per-feature, offer commit checkpoints, write to `bd` when user confirms
4. **Validate** — after epic is fully committed, offer `bv --robot-plan` validation

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

### Determine what's already taskified

Query `bd` for existing issues:

```bash
bd list --type epic --json
bd list --type feature --json
```

From the results, determine which epics and features already have corresponding issues in `bd`. A feature is considered taskified if a `feature`-type issue with a matching `ft:FTN` label exists.

### Identify the next epic to taskify

Using the feature index's implementation ordering (suggested phases and dependency graph):

1. Walk the epics in dependency order
2. For each epic, check if all its features are taskified
3. The first epic with un-taskified features is the recommendation

### Present scope recommendation

Present to the user:

- Which epics are fully taskified (if any)
- Which epic is recommended next and why (dependency order)
- Which features within that epic need taskification
- Which epics are blocked by the recommended one

Let the user confirm the recommended scope or override (e.g., select a different epic, or specific features within an epic).

<HARD-GATE>
Do NOT proceed to Phase 1 until the user has confirmed the scope.
</HARD-GATE>

---

## Phase 1: Decompose Features into Tasks

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
| **Technology choices** (`docs/technology.{org,md}`) | Feature's COMPs have technology decisions | Tech stack for the relevant components |

Do NOT read these documents in full. Read them and extract only the sections relevant to the current feature's identifiers.

### Step 3: Analyze and decompose

Using the feature's FRs, contract elements, COMP boundaries, and tech choices, decompose the feature into implementation tasks.

**Decomposition approach — goal-driven with guardrails:**

Produce tasks that are component-scoped, independently implementable, and expose maximum parallelism. Apply these guardrails:

1. **Single-COMP boundary**: each task's implementation must stay within a single COMP's file/code boundary. This is the primary mechanism for preventing agent conflicts during parallel execution.

2. **Contract element references**: each task must reference which contract elements (ENT schemas, API operations, EVT events) it implements or consumes. This connects the task to the agreed-upon interfaces.

3. **Vertical-slice detection**: if an FR requires thin, uniform changes across multiple COMPs (e.g., adding a data field that threads through API gateway → service → database, or renaming a concept end-to-end), prompt the user:
   - "FR[N] threads through COMP[X], COMP[Y], COMP[Z] as a [describe the change]. Collapse to a single task?"
   - If the user confirms, produce one task that spans those COMPs. Label it with all relevant `comp:` labels.

4. **Agent-session sized**: each task should be completable by an agent in a single session. If a task feels too large, split it. If too small, merge with a sibling task within the same COMP.

### Step 4: Produce task specifications

For each task, produce:

- **Title**: action-oriented, descriptive (e.g., "Implement user creation endpoint in API service")
- **Type**: `task`
- **Priority**: P0–P4 based on the task's position in the dependency graph and the criticality of the FRs it covers (see priority rules below)
- **Labels**: `ep:EPX`, `ft:FTY`, `fr:FRZ` (all relevant FRs), `comp:COMPN`
- **External ref**: the primary FR identifier this task advances
- **Description**: narrative context — what this task does and why, in the context of the feature
- **Design**: reference pointers to all relevant documentation sections (identifiers and file paths, not full content):
  - Feature PRD: which sections, which FRs (e.g., "See FR3.1–FR3.3 in docs/features/user-auth.org")
  - Floorplan: which COMP, which BD edges, which SL diagrams (e.g., "COMP4 boundary, BD1 edge COMP3→COMP4, SL2 in docs/floorplan.org")
  - Contracts: which ENT schemas, API operations, EVT events (e.g., "ENT1 in docs/contracts/entities.org, API1.3 in docs/contracts/api1-user-service.org")
  - Technology choices: which tech stack applies (e.g., "Go + PostgreSQL per docs/technology.org § Backend")
- **Acceptance criteria**:
  - The specific FR sub-items this task covers (may be a subset of the full FR)
  - Contract compliance for elements this task touches (e.g., "API1.3 response matches ENT2 schema")
  - "All existing tests pass"
  - "New tests cover all acceptance criteria above"
- **Dependencies**: which other tasks this blocks or is blocked by, with hard/soft strength

#### Priority rules

Assign priority based on two factors — dependency position and FR criticality:

| Priority | Criteria |
|----------|----------|
| P0 (Critical) | On the critical path AND covers FRs tied to core capabilities (CAP items) |
| P1 (High) | On the critical path OR blocks 2+ other tasks |
| P2 (Medium) | Not on the critical path but covers primary FRs |
| P3 (Low) | Supports secondary functionality, no tasks depend on it |
| P4 (Backlog) | Nice-to-have, could be deferred without blocking the feature |

The feature index's implementation ordering and the feature PRD's dependency graph inform which tasks are on the critical path. Epics and features inherit the highest priority of their child tasks.

### Step 5: Build coverage matrix

After producing all tasks for the feature, build a coverage matrix:

| FR Sub-item | Covered by task(s) |
|-------------|-------------------|
| FR3.1 | Task: "Implement user model" |
| FR3.2 | Task: "Implement user creation endpoint" |
| FR3.3 | ⚠ NOT COVERED |

**Every FR and FR sub-item assigned to this feature must be covered by at least one task.** If any gaps exist, either add tasks to cover them or flag and discuss with the user.

### Step 6: Determine task dependencies and parallelism groups

#### Determine intra-feature dependencies

Use the contracts and floorplan to identify which tasks depend on which within the feature:

1. **Data dependencies**: if task A implements an ENT schema that task B's API operations reference, B depends on A (hard).
2. **API dependencies**: if task A implements an API that task B consumes (visible in BD edges and SL diagrams), B depends on A (hard).
3. **Event dependencies**: if task A emits an EVT that task B handles, B depends on A (soft — can develop in parallel with a stub).
4. **Ordering from the feature index**: the feature PRD's dependency section and the index's suggested implementation phases provide additional ordering signals. Use these to confirm or supplement contract-derived dependencies.

If two tasks within the same feature touch the same COMP (which should be rare given the single-COMP guardrail), they must be sequenced — one blocks the other.

#### Group into parallelism phases

Group the feature's tasks by what can run in parallel:

- Tasks with no inter-dependencies can run simultaneously
- Tasks blocked by others must wait
- Present as ordered phases showing what can be parallelized

---

## Phase 2: Review & Commit

### Present the feature's task hierarchy

After decomposing a feature, present to the user:

1. **Task list**: each task with title, COMP scope, FR coverage, dependencies
2. **Dependency graph**: which tasks block which (with hard/soft typing)
3. **Parallelism groups**: which tasks can run simultaneously
4. **Coverage matrix**: FR sub-items → tasks, with gaps flagged

### Offer commit checkpoint

After the user reviews and confirms the feature's tasks, prompt:

"Commit these **N tasks** for **FTY — [feature name]** to bd now?
(**M features** remaining in EPX, **K tasks** generated so far across **L features**)"

The user can:
- **Commit now** — tasks are written to `bd` immediately (proceed to Phase 3 for this batch)
- **Defer** — tasks are held; continue to the next feature
- **Adjust** — modify tasks before committing or deferring

### Repeat for each feature

Move to the next in-scope feature and repeat from Phase 1, Step 1.

After reviewing all features in the epic, if there are uncommitted tasks, prompt one final time:

"**N tasks** across **M features** are ready to commit for **EPX — [epic name]**. Commit all now?"

<HARD-GATE>
Do NOT write any issues to `bd` until the user has explicitly confirmed the commit.
</HARD-GATE>

---

## Phase 3: Write to `bd`

When the user confirms a commit (either per-feature or per-epic):

### Create the epic issue (if needed)

If no epic-type issue exists in `bd` for this EP:

```bash
bd create --type epic --title "EPX: [epic name]" \
  --labels "ep:EPX,slug:[epic-slug]" \
  --priority [P0-P4] \
  --description "[epic description from feature index]" \
  --silent
```

**Slug generation**: derive `[epic-slug]` by slugifying the epic name — lowercase, hyphens for spaces, no special characters. Shorten long titles to keep slugs concise (e.g., "User Authentication and Authorization System" → `user-auth`, "Real-time Notification Infrastructure" → `realtime-notifs`).

Set the epic's priority to the highest priority of its child tasks.

Capture the issue ID for parent-child linking.

### Create feature issues (if needed)

For each feature being committed that doesn't already have a feature-type issue:

```bash
bd create --type feature --title "FTY: [feature name]" \
  --parent [epic-issue-id] \
  --labels "ep:EPX,ft:FTY,slug:[feature-slug]" \
  --priority [P0-P4] \
  --description "[feature description from feature index]" \
  --silent
```

**Slug generation**: derive `[feature-slug]` by slugifying the feature name — lowercase, hyphens for spaces, no special characters. Shorten long titles to keep slugs concise (e.g., "OAuth2 Provider Integration" → `oauth2-provider`, "Password Reset via Email" → `password-reset`).

Set the feature's priority to the highest priority of its child tasks.

Capture the issue ID for parent-child linking.

### Create task issues

For each task, create the issue and then set fields that `bd create` doesn't support directly:

```bash
# Step 1: Create the issue (captures the issue ID via --silent)
bd create --type task --title "[task title]" \
  --parent [feature-issue-id] \
  --labels "ep:EPX,ft:FTY,fr:FRZ,comp:COMPN" \
  --external-ref "FRZ" \
  --priority [P0-P4] \
  --description "[task description]" \
  --silent

# Step 2: Set design and acceptance criteria (not available on bd create)
bd update [task-issue-id] --design "[reference pointers]"
bd update [task-issue-id] --acceptance-criteria "[acceptance criteria]"
```

Capture each task's issue ID for dependency linking.

### Add dependencies between tasks

For each dependency relationship between tasks:

```bash
bd dep add [blocked-task-id] [blocking-task-id] \
  --type blocks \
  --metadata '{"strength": "hard"}'
```

Or for soft dependencies:

```bash
bd dep add [blocked-task-id] [blocking-task-id] \
  --type blocks \
  --metadata '{"strength": "soft"}'
```

### Add cross-feature dependencies

If a task depends on work in a different feature (from the feature-level dependency graph), add a `blocks` dependency. To find the right target:

1. If the other feature has already been taskified, find the specific task that produces the interface this task consumes — match on shared contract elements (ENT/API/EVT identifiers) in the `fr:` labels or design field.
2. If no specific task can be identified, or the other feature hasn't been taskified yet, **fall back to the feature-level issue itself** as the dependency target. This ensures the dependency is tracked even before the blocking feature's tasks exist.

```bash
bd dep add [this-task-id] [other-feature-issue-id] \
  --type blocks \
  --metadata '{"strength": "hard", "reason": "requires FT2 API1.3 interface"}'
```

### Confirm success

After writing all issues, report:
- Number of issues created (epics, features, tasks)
- Number of dependencies added
- Any issues encountered

---

## Phase 4: Validate

After all tasks for the epic are committed, offer:

"All tasks for **EPX — [epic name]** are committed. Would you like me to run `bv --robot-plan` to validate the dependency graph?"

If the user confirms, run:

```bash
bv --robot-plan --label "ep:EPX" --format json
```

Report any issues found:
- Dependency cycles
- Orphaned tasks (no dependencies and not in the first parallelism group)
- Missing coverage

### Suggest next steps

After validation (or if the user skips it):

- "Your tasks are in bd. Next steps you might consider:"
  - Run `bv --robot-priority` to see recommended task ordering
  - Use `bd ready` to find tasks with no blockers — these can start immediately
  - Assign tasks to agents with `bd update [id] --assignee [agent]`
  - Run this skill again for the next epic in dependency order
  - Use `bv` TUI for an interactive view of the task graph

---

## Important Constraints

- Your ONLY output is `bd` issues (or interview/review questions when gathering context)
- Do NOT write code, create implementation plans, or produce any artifact other than `bd` issues
- Do NOT load all starchitect documents upfront — lazy-load per-feature to conserve context
- Do NOT create tasks without user review and confirmation
- Do NOT invent requirements — tasks must trace back to FRs, contracts, and floorplan elements. If coverage is incomplete, flag the gap rather than filling it with assumptions
- When a feature has already been taskified (feature-type issue exists in `bd`), skip it unless the user explicitly asks to re-taskify
- **Re-taskification**: when the user asks to re-taskify a feature, do NOT delete existing tasks. Instead:
  1. Create new tasks first (so references exist before updating old ones)
  2. For tasks not yet started (`open` status): update them in place if the change is minor, or close them with `close_reason` noting the superseding task (e.g., "Superseded by [new-task-id]")
  3. For tasks in progress (`in_progress` status): update their description, design, and acceptance criteria to reflect changed requirements. Do not close active work without user confirmation.
  4. Use `bd dep add [old-task-id] [new-task-id] --type supersedes` to link old → new when replacing tasks
- Prefer precision over verbosity in task descriptions — the design field carries the detailed pointers, the description carries the narrative
