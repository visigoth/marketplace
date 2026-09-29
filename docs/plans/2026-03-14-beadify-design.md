# beadify Design

**Date:** 2026-03-14 (rewritten 2026-09-29)
**Plugin:** starchitect
**Skill:** `starchitect:beadify`

## Purpose

Bridge the gap between product architecture (PRDs, floorplans, feature breakdowns, contracts, TDDs, artifacts) and actionable implementation tasks in `bd` (beads).

The original design optimized for one thing: maximum parallelism through component-scoped tasks. That produced graphs where every component got built and nothing got connected. The skill now optimizes for a different target: **a task graph whose completion yields something a human can run.** Parallelism is still valuable, but it is subordinate to the work actually adding up to a functioning artifact.

## Position in the starchitect pipeline

```
prd-create → floorplan → tech-plan → prd-feature-breakdown → contracts → tdd
  → beadify → test-plan → artifacts → tech-plan (revisit)
```

beadify consumes every upstream document. It runs before test-plan because it creates the verification *skeleton* that test-plan then fills with cases.

## The integration gap this design fixes

Running the original beadify, then test-plan, then handing the graph to implementation agents produced features that were implemented but not wired: components existed, contracts were honored in isolation, unit tests passed, and the feature did not work. Four structural causes, each addressed:

| Cause | Fix |
|-------|-----|
| **The single-COMP guardrail forbade wiring work.** Composition roots, route registration, DI, config plumbing, migration registration, and feature exposure are inherently cross-component, so under a strict single-COMP rule they belonged to no task | Bead kinds are split into **scoped** (one COMP, parallel-safe) and **exempt** (cross-component by nature). Wiring, harness, and verification beads are exempt |
| **Nothing checked seam-level coverage.** The coverage matrix tracked FR sub-items. An FR could be fully covered while the boundary between its two halves was never exercised | **Ledger B (seams)**: enumerate every place two things must meet, then answer two questions per row — which bead makes this seam *live*, and which bead *proves* it |
| **The test harness was nobody's job.** test-plan declared test infrastructure out of scope; beadify never mentioned it. Integration test specs had nowhere to run | `kind:harness` beads are beadify's responsibility, with an 11-item checklist (runner config, real dependency provisioning, fixtures, seed data, determinism, network policy, teardown, CI job, one-command local runner) |
| **Nothing consumed `docs/artifacts.*`.** Artifact specifications existed and no bead produced an artifact | **Ledger C** tracks journeys and artifacts with a "runnable output" column; `kind:verify-artifact` beads and the epic gate's runnability checklist enforce it |

## Design decisions

### The completeness contract

Four invariants the skill must be able to demonstrate, not merely assert:

1. **Nothing lost** — every identifier in every loaded document has a disposition (covered by beads, deliberately out of scope with a reason, or deferred with a trigger)
2. **Nothing unreachable** — every bead is reachable from the epic and every dependency it needs exists
3. **Nothing unverified** — every seam has a bead that makes it live and a bead that proves it
4. **Nothing unexplained** — every bead carries enough context to be picked up cold

Phase 8 validates all four with 22 numbered rules and a how-to-check column for each.

### Bead kinds replace the flat task model

Sixteen kinds, each mapping to a `bd` issue type and to a scoping rule:

`spike` · `decision` · `scaffold` · `contract-types` · `harness` · `skeleton` · `impl` · `wiring` · `verify-seam` · `verify-journey` · `verify-artifact` · `verify-runtime` · `ops` · `gate` · `deferred` · `docs`

Kinds carry a `kind:<name>` label. This is a separate namespace from test-plan's `test:*` labels — see *Division of labor* below.

### Task scoping: scoped by default, exempt where the work is a boundary

`kind:impl`, `kind:scaffold`, and `kind:contract-types` beads stay within a single COMP so agents avoid file conflicts. `kind:wiring`, `kind:harness`, `kind:skeleton`, `kind:verify-*`, and `kind:gate` beads are exempt — integration work is inherently cross-component, and forbidding it is precisely why plans produce unwired code.

Exempt beads need a different collision check: two wiring beads editing the same composition root cannot run in parallel even though neither names a COMP. Phase 5 checks exempt beads for file collisions explicitly, since this is the most common source of false parallelism.

**Vertical-slice detection** is retained: when an FR is thin plumbing across COMPs, collapse it into one bead rather than splitting it.

### Walking skeleton first

The first beads for an epic build an end-to-end path that runs — even returning hardcoded values — before any component is built out. Every later bead replaces a piece of the skeleton rather than adding an unconnected piece. A `stub:<what>` label on the skeleton bead names each placeholder, and a ledger records which bead replaces it.

### Gate beads

One `kind:gate` bead per feature and one per epic, blocked by all siblings, with acceptance criteria that require running a demonstration command and posting the output as a comment. The epic gate carries a 7-item runnability checklist. A feature cannot be closed on the strength of its parts being closed.

### FRs are goals, not tasks

Unchanged from the original design. A single FR may require multiple beads; multiple FRs may share beads. The invariant is at the feature level. What changed is that the capture ledger records one row per FR *sub-item*, and every row must have an explicit disposition rather than being implicitly covered.

### Self-documenting beads

Every bead carries a mandatory **Background & Intent** comment answering six prompts: where this comes from, what we're actually trying to achieve, how it serves the larger goal, the thinking that led here, what we considered and didn't do, and what future-you should know.

The test is **orphan context**: if an agent opened this bead with no other document available, could it do the work correctly? The balance rule keeps this from becoming transcription — inline what changes the implementation, point to what merely explains it. A Background comment that would apply equally to any bead in the feature is a wasted comment.

### Reasoning budget

The skill instructs the model to use its highest available reasoning mode (ultrathink on Claude Code) and marks three **[THINK DEEPLY]** points: the seam ledger, the decomposition, and the dependency graph. These are the three places where a cheap answer produces a graph that looks complete and isn't.

### Lazy loading

Retained and extended. Epic-level context (journeys, floorplan structure, artifacts, technology) loads once and is cached across features. Per-feature context (feature PRD, the contract and TDD sections it references) loads when decomposing that feature. What could not be found is recorded rather than silently skipped.

### No auto-init

The skill checks for `.beads/` and stops with guidance if missing. It does not run `bd init`.

## Division of labor with test-plan

| | beadify | test-plan |
|---|---------|-----------|
| Owns | Which seams, journeys, and artifacts must be proven; the harness to prove them in | The cases: inputs, expected outputs, error paths, edge cases |
| Labels | `kind:verify-*`, `kind:harness` | `test:unit`, `test:integration`, `test:e2e`, `test:ux` |
| Granularity | One bead per seam / journey / artifact / TR | Case-level specs attached to beads |

**beadify must never emit a `test:*` label.** test-plan skips any feature that already has one, so a `test:*` label from beadify would suppress the case specification entirely.

test-plan **enriches** beadify's verify beads — appending a `## Test Cases` section and adding the `test:*` label to the existing bead — rather than creating a parallel test task. A seam with two half-done beads is worse than one complete bead.

## Traceability mapping

| `bd` field | Content |
|------------|---------|
| `issue_type` | `epic`, `feature`, `task`, `chore`, `spike`, `decision`, `milestone` |
| `labels` | `proj:<slug>`, `ep:EPX`, `ft:FTY`, `fr:FRZ`, `comp:COMPN`, `kind:<kind>`, `stub:<what>` |
| `external_ref` | Primary FR identifier |
| `description` | What / Why it matters / Scope boundaries / Traceability, with a `Serves:` line |
| `design` | Approach / Interfaces / Files & locations / Technology constraints / Wiring points |
| `acceptance_criteria` | Checkboxes, a runnable verification command, and the evidence to post |
| `notes` | Considerations / Alternatives rejected / Gotchas / Deferred / Open questions |
| `metadata` | JSON: `frs`, `acs`, `trs`, `entities`, `apis`, `events`, `comps`, `edges`, `flows`, `sequences`, `journeys`, `artifacts`, `seam`, `kind` |
| `parent` field | EP → FT → bead hierarchy |
| `blocks` dep | Hard ordering — the blocked bead is excluded from `bd ready` |
| `related` dep | Soft ordering — advisory only |
| `validates` dep | A verification bead proves an implementation bead |
| comment | Dependency rationale, background & intent, sequencing notes |

**Dependency strength is encoded in the edge type, not in metadata.** `bd dep add` has no `--metadata` flag, and only one edge may exist between any pair of issues — a second `bd dep add` with a different type fails rather than replacing. The reason for a non-obvious edge goes in a comment.

## Phases

| Phase | What happens |
|-------|--------------|
| **0. Discover & scope** | Check `.beads/`; inventory 8 upstream documents and state the consequence of each missing one; discover the `proj:` slug; classify already-beadified features as not started / partial / complete; confirm scope |
| **1. Load context** | Epic-level context once, per-feature context on demand; record what could not be found |
| **2. Ledgers** *[THINK DEEPLY]* | **A** capture (one row per identifier, explicit disposition), **B** seams (live via / proven by), **C** journeys & artifacts (demonstrated by / runnable output) |
| **3. Decompose** *[THINK DEEPLY]* | Assign bead kinds; walking skeleton first; 13-item wiring checklist; 11-item harness checklist; verification beads with the no-mocks-at-the-seam rule; stub ledger; gate beads; granularity and sanity bands |
| **4. Author** | Full field set per bead against the orphan-context test; mandatory Background & Intent comment; sequencing rationale; feature and epic context comments |
| **5. Dependencies** *[THINK DEEPLY]* | Map intent to edge type; one edge per pair; canonical skeleton (scaffold → contract-types → skeleton → impl → wiring → verify-seam → gate); parallelism waves; file-collision check on exempt beads |
| **6. Review & commit** | Five-part presentation; per-feature checkpoints by default |
| **7. Write to bd** | `bd create --graph plan.json --json`, then comments keyed off the returned id map; per-bead fallback; bulk dependency wiring via NDJSON; idempotency via deterministic node keys; re-beadification rules |
| **8. Validate** | 22 numbered rules plus `bd lint`, `bd dep cycles`, `bv --robot-plan`; re-state the completeness contract with evidence per invariant |
| **9. Handoff** | What was created, where to start, demonstration commands, what's still open, next skills |

## Sanity bands

A feature with 3–6 FRs across 2–4 COMPs should produce roughly 15–40 beads. If a feature of that shape yields fewer than 10, the wiring or the verification collapsed. A healthy mix is roughly 35–45% implementation, 15–20% wiring, 20–30% verification.

These are diagnostics, not quotas. A band miss is a prompt to re-examine ledger B, not a number to pad toward.

## Autopilot mode

Gates become soft: the skill proceeds without waiting for confirmation and reports what it did. Autopilot does **not** skip content — it skips waiting. The ledgers, the comments, and the validation rules all still run.

## Scope selection logic

1. Read the implementation ordering from the feature index
2. Query `bd` for existing epic and feature issues, filtered by `proj:<slug>`
3. Classify each feature as **not started**, **partial**, or **complete** by which `kind:*` labels are present
4. The next epic is the first in dependency order that is not complete
5. Partial features get a completion pass, which adds missing kinds without duplicating existing beads

The user can override this at any time.

## Re-beadification

Never delete beads. Create the new bead first, then reconcile the old one:

- `open` → update in place, or close with a reason naming the superseding bead
- `in_progress` → update fields without closing
- link with `bd dep add <old-id> <new-id> --type supersedes`
