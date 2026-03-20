# beadify Design

**Date:** 2026-03-14
**Plugin:** starchitect
**Skill:** `starchitect:beadify`

## Purpose

Bridge the gap between product architecture (PRDs, floorplans, feature breakdowns, contracts) and actionable implementation tasks in `bd` (beads). Decomposes features into component-scoped task hierarchies that expose maximum parallelism for agents working concurrently without file conflicts.

## Position in the starchitect pipeline

```
prd-create → floorplan → prd-feature-breakdown → contracts → beadify → agent implementation
```

## Design decisions

### Task scoping: component-scoped by default

Each task stays within a single COMP boundary from the floorplan. Agents can work on different COMPs in parallel without stepping on each other's files. Contracts define the interfaces between components — agents code against those contracts.

**Vertical-slice detection:** When an FR is thin plumbing across multiple COMPs (e.g., adding a data field end-to-end), the skill detects this and prompts the user to collapse it into a single task rather than splitting across components.

### FRs are goals, not tasks

A single FR may require multiple tasks. Multiple FRs may share tasks. The skill decomposes FRs into implementation tasks — it doesn't mechanically map 1:1. The invariant is at the **feature level**: when all tasks for a feature are complete, every FR and FR sub-item assigned to that feature must be covered.

### Goal-driven decomposition with guardrails

The skill uses goal-driven decomposition ("analyze the FRs, contracts, and floorplan and produce tasks that maximize parallelism") rather than prescriptive templates. Guardrails:

1. Each task stays within a single COMP boundary
2. Each task references which contract elements (ENT, API, EVT) it implements
3. Vertical-slice detection with collapse prompting
4. Tasks must be sized for a single agent session

### Lazy loading

Context is expensive. The skill loads documents incrementally:

- **Always loaded:** feature index (`docs/features/index.{org,md}`) — loaded once in Phase 0
- **Per-feature:** that feature's PRD — loaded when decomposing, released after review
- **On-demand:** relevant sections of contracts, floorplan, technology choices — loaded only as referenced by the feature PRD

### No auto-init

The skill checks for `.beads/` and stops with guidance if missing. It does not run `bd init`.

## Traceability mapping

| `bd` field | Content |
|------------|---------|
| `issue_type` | `epic`, `feature`, or `task` |
| `labels` | `ep:EPX`, `ft:FTY`, `fr:FRZ`, `comp:COMPN` |
| `external_ref` | Primary FR identifier |
| `description` | Narrative: what the task is and why |
| `design` | Rich pointer set: feature PRD sections, floorplan diagrams (BD/SL/DF), contract elements (ENT/API/EVT), technology choices |
| `acceptance_criteria` | Specific FR sub-items this task covers, contract compliance, "all existing tests pass", "new tests cover all acceptance criteria" |
| `parent-child` dep | EP → FT → task hierarchy |
| `blocks` dep | Execution ordering between tasks |
| `blocks` dep `--metadata` | `{"strength": "hard"}` or `{"strength": "soft"}` |

### Acceptance criteria structure

Each task's acceptance criteria includes:

1. The specific FR sub-items this task covers (may be a subset of the FR)
2. Contract compliance for the elements this task touches (ENT schemas, API operations)
3. "All existing tests pass"
4. "New tests cover all acceptance criteria above"

Coverage is tracked across tasks within a feature. When presenting a feature's task hierarchy, the skill shows a coverage matrix mapping FR sub-items to tasks and flags any gaps.

## Phases

### Phase 0: Discover & scope

1. Check for `.beads/` — if missing, tell user to run `bd init` and stop
2. Load **only** `docs/features/index.{org,md}` — do not load feature PRDs, contracts, floorplan, or technology choices yet
3. Query `bd` for existing epic-type and feature-type issues to determine what's already been taskified
4. From the index's dependency graph + existing issues in `bd`, identify the next un-taskified epic in dependency order
5. Present recommendation with context: which epics are done, which are blocked, what's next. Let user confirm or override scope

### Phase 1: Decompose (per feature, lazy-loaded)

For each in-scope feature:

1. Load that feature's PRD from `docs/features/<name>.{org,md}`
2. Load only the relevant sections of contracts, floorplan, and technology choices as referenced by the feature PRD
3. Analyze FRs + contract elements (ENT, API, EVT) + COMP boundaries + tech choices
4. Goal-driven decomposition with guardrails (see above)
5. Detect vertical-slice situations and prompt user to collapse
6. For each task, produce all structured fields (title, labels, description, design, acceptance_criteria, dependencies)
7. Build coverage matrix: FR sub-items → tasks. Flag any gaps.

### Phase 2: Review (per feature, with commit checkpoints)

After decomposing each feature's tasks:

1. Present the task hierarchy: tasks, dependencies, parallelism groups, coverage matrix
2. Prompt: "Commit these N tasks for FTY — [name] to bd now? (M features remaining in EPX, K tasks generated so far across L features)"
3. User can:
   - **Commit now** — tasks are written to `bd` immediately
   - **Defer** — tasks are held until more features are reviewed or the full epic is done
   - **Adjust** — modify tasks before committing

### Phase 3: Write to `bd`

When user commits:

1. Create epic issue (`issue_type=epic`) if it doesn't exist in `bd` yet
2. Create feature issue (`issue_type=feature`, `parent-child` dep to epic) if it doesn't exist
3. Create task issues via `bd create` with all structured fields, `parent-child` dep to feature
4. Add `blocks` dependencies between tasks via `bd dep add` with `--metadata` for hard/soft strength
5. After all tasks for the epic are committed, offer to run `bv --robot-plan` to validate the dependency graph

## Scope selection logic

The skill determines the next epic to taskify by:

1. Reading the implementation ordering / suggested phases from the feature index
2. Querying `bd` for existing epic and feature issues
3. An epic is "fully taskified" if all its features have corresponding feature-type issues in `bd`
4. The next epic is the first in dependency order that is not fully taskified
5. Within that epic, only features without existing feature-type issues are in scope

The user can override this at any time.
