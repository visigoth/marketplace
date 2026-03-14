---
name: starchitect:prd-feature-breakdown
description: >
  Break a high-level PRD into epics and feature-level PRDs. Identifies epic boundaries from capabilities
  and floorplan clusters, decomposes epics into features, coalesces shared features, analyzes dependencies,
  and generates feature PRDs with typed dependency graphs.
  Triggers: "break down the PRD", "feature breakdown", "split PRD into features",
  "decompose PRD", "feature PRDs", "break this into features", "epic breakdown".
user-invocable: true
---

# PRD Feature Breakdown: Epics → Features → Feature PRDs

Break a high-level PRD into epics, then decompose each epic into independently implementable feature-level PRDs. Each feature PRD is self-contained and specific enough to convert directly into a task hierarchy.

<HARD-GATE>
Do NOT skip to writing output. Every epic list, feature list, coalesced list, dependency analysis, and feature PRD must be presented to the user for review and confirmation before proceeding or writing to disk.
</HARD-GATE>

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Discover inputs** — find parent PRD, floorplan, architecture docs, existing tech, existing feature PRDs, determine output format
2. **Identify epics** — analyze CAP items and floorplan clusters to define epic boundaries, present to user for confirmation
3. **Decompose epics into features** — work epic-by-epic, identify features with max depth 2, present all features across all epics for confirmation
4. **Coalesce and deduplicate** — detect overlapping features across epics, propose merge/extract/keep-separate, present coalesced list for confirmation
5. **Order and dependency analysis** — epic-level and feature-level ordering with typed dependencies, identify critical path, present for confirmation
6. **Generate output** — write epic-feature index, then produce each feature PRD one at a time with review gates

---

## Phase 0: Discover Inputs

### Search for the parent PRD

Search these locations:

| Document | Locations to check |
|----------|--------------------|
| **PRD** | `docs/prd.md`, `docs/prd.org`, `docs/prd/`, `docs/prds/` |
| **Floorplan** | `docs/floorplan.md`, `docs/floorplan.org` |
| **Architecture docs** | `docs/architecture.md`, `docs/architecture.org`, `docs/architecture/` |
| **Technology choices** | `docs/technology.md`, `docs/technology.org`, `docs/technology/` |
| **Existing feature PRDs** | `docs/features/` |

Use Glob to check all locations. Read any documents found.

### If no PRD is found:

- Tell the user: "I couldn't find a PRD. I recommend starting with the prd-create skill to create one first."
- Stop here unless the user provides a PRD or points to one

### If existing feature PRDs are found:

If `docs/features/` already contains feature PRDs from a prior run, ask the user:

- **Start fresh**: discard existing feature PRDs and regenerate from scratch
- **Refine**: use existing feature PRDs as a starting point and adjust based on new analysis

Do not proceed until the user has chosen.

### Scan for existing technology

Check for these files/directories and extract technology information:

- `package.json` (Node.js ecosystem, dependencies)
- `go.mod` (Go modules)
- `Cargo.toml` (Rust crates)
- `Gemfile` (Ruby gems)
- `requirements.txt`, `pyproject.toml`, `Pipfile` (Python)
- `Podfile` (iOS/CocoaPods)
- `build.gradle`, `build.gradle.kts` (JVM/Android)
- `pom.xml` (Java/Maven)
- `mix.exs` (Elixir)
- `composer.json` (PHP)
- `pubspec.yaml` (Dart/Flutter)
- `CMakeLists.txt`, `Makefile` (C/C++)
- `Package.swift` (Swift)
- `*.csproj`, `*.sln` (C#/.NET)
- `docker-compose.yml`, `Dockerfile` (containerization)
- `terraform/`, `*.tf` (infrastructure)
- `third-party/` (third party repositories)

Use Glob to find these files. Note what exists for context when writing feature PRDs.

### Determine output format

- If the parent PRD is `.org`, output feature PRDs as `.org`
- If the parent PRD is `.md`, output feature PRDs as `.md`
- If ambiguous, scan `docs/` for dominant format and follow that
- Tell the user which format you'll use

---

## Phase 1: Identify Epics

### Analyze PRD for epic boundaries

Epics represent major product capability areas. Use three signal layers to identify them:

1. **Primary signal — CAP items**: Each top-level CAP or tightly related cluster of CAPs is an epic candidate. CAPs that share user personas, domain concepts, or data are candidates for the same epic.

2. **Secondary signal — Floorplan subgraph clusters (BD1)**: If a floorplan exists, its block diagram clusters reveal architectural groupings. Components that form a connected subgraph with few external edges are strong epic candidates. Use the floorplan's swim-lane diagrams (SL) to see which components collaborate on which journeys.

3. **Cross-cutting signal — User Journeys (UJ)**: UJs that span multiple CAPs reveal cross-cutting concerns. These may become their own epic (e.g., "Onboarding" spanning auth + setup + first-use) or indicate that certain CAPs belong in the same epic.

If no floorplan exists, rely on CAP items and UJs. Note to the user: "No floorplan found. Epic boundaries would be more precise with a floorplan. Consider running the floorplan skill first."

### Guardrail: 3–7 epics

- **Target 3–7 epics.** This is the sweet spot for most products.
- If analysis yields **>7 epics**: consolidate related epics. Present the consolidation rationale.
- If analysis yields **<3 epics**: suggest to the user that the product may not need an epic layer — offer to skip directly to features. If the user agrees, skip Phase 1 and go directly to Phase 2 with the entire PRD as the scope.

### Present the proposed epic list

For each proposed epic, show:

- **EP identifier**: EP1, EP2, ...
- **Epic name**: short, descriptive (e.g., "User Authentication", "Billing & Payments")
- **CAP coverage**: which CAP items this epic covers
- **UJ coverage**: which user journeys touch this epic
- **Component coverage**: which COMP identifiers belong to this epic (if floorplan exists)
- **Brief description**: 1–2 sentences explaining scope

<HARD-GATE>
Do NOT proceed to Phase 2 until the user has confirmed the epic list.
</HARD-GATE>

---

## Phase 2: Decompose Epics into Features

### Work epic-by-epic

For each confirmed epic, identify features by clustering the epic's PRD items:

- **Capabilities** (CAP items assigned to this epic)
- **Use Cases** (UC items) relevant to this epic
- **Functional Requirements** (FR items) that implement this epic's capabilities
- **Components** (COMP identifiers) from the floorplan that belong to this epic

### Use the floorplan to inform grouping

If a floorplan exists, use its component inventory (COMP identifiers), block diagrams (BD), and data flows (DF) to inform feature boundaries within the epic. Components that are tightly connected in the block diagram or share data flows are strong candidates for the same feature. Swim-lane diagrams (SL) reveal which components collaborate on specific user journeys — these collaborations often map to feature boundaries.

### Guardrail: 3–7 features per epic

- **Target 3–7 features per epic.**
- **Size heuristic**: if a feature can be described as a single user story with 2–3 acceptance criteria, it is too small — merge it with a sibling.
- **Max depth 2**: EP → FT → FT.N (sub-feature). Most features should NOT have sub-features. Sub-features (depth 2) are only for genuinely large features that cannot be scoped as a single implementable unit.

### Feature identifiers

- Features use the **FT** prefix: FT1, FT2, FT3, ...
- Sub-features extend the parent: FT3.1, FT3.2 (use sparingly)
- FT identifiers are globally unique across all epics — do not restart numbering per epic
- Flag features that might span multiple epics (same FR/UC/CAP coverage, same COMP ownership) for coalescing in Phase 3

### Present all features across all epics

After decomposing all epics, present the complete feature list organized by epic:

For each feature, show:

- **FT identifier**: FT1, FT2, ...
- **Feature name**: short, descriptive
- **Epic membership**: which EP this feature belongs to
- **Parent PRD coverage**: which CAP, UC, UJ, FR items this feature covers
- **Components**: which COMP identifiers belong to this feature (if floorplan exists)
- **Brief description**: one sentence explaining the feature's scope
- **Cross-epic flag**: if this feature's PRD item coverage or COMP ownership overlaps with features in other epics, flag it (e.g., "⚠ overlaps with FT7 in EP3 — shared FR4, COMP5")

<HARD-GATE>
Do NOT proceed to Phase 3 until the user has confirmed all features across all epics.
</HARD-GATE>

---

## Phase 3: Coalesce and Deduplicate

### Detect overlap

For each pair of features flagged as potentially overlapping in Phase 2, analyze:

- **Shared PRD item coverage**: do they cover the same FR, UC, or CAP items?
- **Shared COMP ownership**: do they operate on the same floorplan components?
- **Shared data flows**: do they read/write the same DF paths?

### Propose resolution for each overlap

For each detected overlap, propose one of three resolutions:

1. **Merge**: combine into a single feature under the most natural epic. Other epics that previously contained the merged feature now reference it as a dependency. Use when: the overlap is substantial (>50% shared items) and the features are doing essentially the same thing from different angles.

2. **Extract**: pull the shared concern into a standalone "foundation" feature that multiple epics depend on. Use when: the shared part is a coherent, independently valuable capability (e.g., "notification delivery" used by both "billing alerts" and "team invitations").

3. **Keep separate**: the features remain independent. Use when: overlap is superficial — they touch the same component but do unrelated things (e.g., both read the user database but for different purposes).

For each proposal, explain the rationale.

### Reassign FT identifiers

After coalescing, renumber FT identifiers to be sequential and gap-free. Any extracted foundation features should appear early in the sequence since they are dependencies.

### Present coalesced feature list

Show the updated feature list with:

- Final FT identifiers
- Epic membership (including any changes from merging/extraction)
- Shared feature markers (for features that serve multiple epics)
- Rationale for each merge/extract/keep-separate decision

<HARD-GATE>
Do NOT proceed to Phase 4 until the user has confirmed the coalesced feature list.
</HARD-GATE>

---

## Phase 4: Order and Dependency Analysis

### Epic-level ordering

Determine which epics depend on which. An epic depends on another if any of its features have hard dependencies on features in the other epic.

### Feature-level dependency graph

For each feature, identify dependencies on other features with typed relationships:

- **Hard dependency**: cannot start implementation without the other feature being complete. The feature consumes APIs, data structures, or behaviors that don't exist yet. Example: "Billing depends (hard) on Authentication — billing endpoints require authenticated sessions."

- **Soft dependency**: can develop in parallel but needs interface agreement upfront. The features will interact at runtime but can be built independently if they agree on contracts. Example: "Team Management depends (soft) on Notification Service — needs to agree on notification event schema, but can stub it during development."

Cite which component interactions (from the floorplan's block diagram edges) or PRD items create each dependency.

### Identify critical path and parallelizable work

- **Critical path**: the longest chain of hard dependencies from start to finish
- **Parallelizable work**: features (or groups of features) that can be developed concurrently because they have no hard dependencies on each other
- Present as an ordered list showing which features can start immediately, which must wait, and what can run in parallel

### Present ordering

Show:

- Epic-level dependency summary
- Feature-level dependency graph (FT → FT with hard/soft typing)
- Critical path
- Suggested implementation phases (groups of features that can be built in parallel)

<HARD-GATE>
Do NOT proceed to Phase 5 until the user has confirmed the ordering and dependencies.
</HARD-GATE>

---

## Phase 5: Generate Output

### 1. Write the index file

Create `docs/features/index.org` (or `.md` based on Phase 0).

Present the index to the user for review before writing to disk.

<HARD-GATE>
Do NOT write the index to disk until the user has reviewed and approved it.
</HARD-GATE>

**Org-mode template (`docs/features/index.org`):**

```org
#+TITLE: Feature PRDs Index
#+DATE: YYYY-MM-DD

* Overview

Feature-level PRDs derived from [[file:../prd.org][parent PRD]].

* Epic Summary

| EP | Epic | Description | Features | PRD Coverage |
|----+------+-------------+----------+--------------|
| EP1 | Authentication | User identity and session management | FT1, FT2 | CAP1, UC1 |
| EP2 | Team Management | Organization and membership | FT3, FT4, FT5 | CAP2, UC2, UC4 |

* Features by Epic

** EP1: Authentication

| FT | Feature | File | PRD Coverage | Components | Dependencies |
|----+---------+------+--------------+------------+--------------|
| FT1 | Login & Registration | [[file:login-registration.org]] | CAP1, UC1, FR1–FR3 | COMP2, COMP4 | None |
| FT2 | Session Management | [[file:session-management.org]] | CAP1.2, FR4–FR5 | COMP4 | FT1 (hard) |

** EP2: Team Management

| FT | Feature | File | PRD Coverage | Components | Dependencies |
|----+---------+------+--------------+------------+--------------|
| FT3 | Team CRUD | [[file:team-crud.org]] | CAP2, UC2, FR6–FR8 | COMP3, COMP5 | FT1 (hard) |
| FT4 | Member Invitations | [[file:member-invitations.org]] | CAP2.1, UC4, FR9 | COMP5 | FT3 (hard), FT5 (soft) |
| FT5 | Notifications | [[file:notifications.org]] | FR10–FR11 | COMP8 | None |

* Shared Features

Features that serve multiple epics:

| FT | Feature | Epics | Rationale |
|----+---------+-------+-----------|
| FT5 | Notifications | EP2, EP3 | Extracted: notification delivery used by invitations and billing alerts |

* Implementation Ordering

** Critical Path

FT1 → FT2 → FT3 → FT4

** Suggested Phases

| Phase | Features | Can parallelize |
|-------+----------+-----------------|
| 1 | FT1, FT5 | Yes — no shared dependencies |
| 2 | FT2, FT3 | Yes — FT2 and FT3 both depend only on FT1 |
| 3 | FT4 | No — depends on FT3 and FT5 |

* Dependency Graph

| Feature | Hard Dependencies | Soft Dependencies |
|---------+-------------------+-------------------|
| FT1 | None | None |
| FT2 | FT1 | None |
| FT3 | FT1 | None |
| FT4 | FT3 | FT5 |
| FT5 | None | None |
```

**Markdown template (`docs/features/index.md`):**

```markdown
# Feature PRDs Index

**Date:** YYYY-MM-DD
**Parent PRD:** [PRD](../prd.md)

## Epic Summary

| EP | Epic | Description | Features | PRD Coverage |
|----|------|-------------|----------|--------------|
| EP1 | Authentication | User identity and session management | FT1, FT2 | CAP1, UC1 |
| EP2 | Team Management | Organization and membership | FT3, FT4, FT5 | CAP2, UC2, UC4 |

## Features by Epic

### EP1: Authentication

| FT | Feature | File | PRD Coverage | Components | Dependencies |
|----|---------|------|--------------|------------|--------------|
| FT1 | Login & Registration | [login-registration.md](login-registration.md) | CAP1, UC1, FR1–FR3 | COMP2, COMP4 | None |
| FT2 | Session Management | [session-management.md](session-management.md) | CAP1.2, FR4–FR5 | COMP4 | FT1 (hard) |

### EP2: Team Management

| FT | Feature | File | PRD Coverage | Components | Dependencies |
|----|---------|------|--------------|------------|--------------|
| FT3 | Team CRUD | [team-crud.md](team-crud.md) | CAP2, UC2, FR6–FR8 | COMP3, COMP5 | FT1 (hard) |
| FT4 | Member Invitations | [member-invitations.md](member-invitations.md) | CAP2.1, UC4, FR9 | COMP5 | FT3 (hard), FT5 (soft) |
| FT5 | Notifications | [notifications.md](notifications.md) | FR10–FR11 | COMP8 | None |

## Shared Features

Features that serve multiple epics:

| FT | Feature | Epics | Rationale |
|----|---------|-------|-----------|
| FT5 | Notifications | EP2, EP3 | Extracted: notification delivery used by invitations and billing alerts |

## Implementation Ordering

### Critical Path

FT1 → FT2 → FT3 → FT4

### Suggested Phases

| Phase | Features | Can parallelize |
|-------|----------|-----------------|
| 1 | FT1, FT5 | Yes — no shared dependencies |
| 2 | FT2, FT3 | Yes — FT2 and FT3 both depend only on FT1 |
| 3 | FT4 | No — depends on FT3 and FT5 |

### Dependency Graph

| Feature | Hard Dependencies | Soft Dependencies |
|---------|-------------------|-------------------|
| FT1 | None | None |
| FT2 | FT1 | None |
| FT3 | FT1 | None |
| FT4 | FT3 | FT5 |
| FT5 | None | None |
```

### Fill in the template

- Replace placeholder data with actual epics, features, and dependencies from Phases 1–4
- Use today's date
- Link to the parent PRD
- Ensure all FT and EP identifiers match the confirmed lists

### 2. Generate feature PRDs (one at a time)

For **each confirmed feature**, one at a time:

#### Produce the feature PRD

Follow this structure. The feature PRD uses the project's flat identifier namespace — items from the parent PRD keep their original identifiers, and any new items introduced in the feature PRD must use identifiers that don't collide with the parent or any other feature PRD.

##### Epic Membership & Feature Identifier

State the feature's position in the hierarchy:

> **Epic:** EP2 — Team Management
> **Feature:** FT4 — Member Invitations
> **Dependencies:** FT3 (hard), FT5 (soft)

##### Provenance

State which parent PRD items this feature covers:

> Covers: CAP2.1, UC4, FR9

##### Overview

- What this feature is
- Scoped description — what it does and what it doesn't do
- Why it exists (the user problem it solves)

##### User Personas

- Relevant subset from the parent PRD
- Include only personas that interact with this feature

##### User Stories

- Organized into Use Cases (UC) and User Journeys (UJ), same convention as parent PRD
- Items from the parent PRD keep their original identifiers (e.g., if this feature covers UC3 and UC5, use UC3 and UC5 — do not renumber to UC1, UC2)
- New items not in the parent PRD continue from the highest existing number across the project (e.g., if UC5 is the highest in the parent, new use cases start at UC6)
- Each story has: identifier, title, description, acceptance criteria
- Sub-items extend the parent identifier (UC3.1, UC3.2) and are naturally unique

##### Functional Requirements

- Detailed, implementable requirements — inherit FR identifiers from parent PRD, continue numbering for new items
- Verifiable sub-items extend the parent (FR3.1, FR3.2, etc.)
- Specific enough to convert directly into tasks

##### Interface Boundaries

- Which COMP identifiers are inside this feature
- Which block diagram (BD) edges cross this feature's boundary, and their nature (e.g., "queries", "authenticates")
- Which data flow (DF) paths connect this feature to other features
- What data this feature exposes to other features (described by intent, e.g., "provides user profile information" — not by schema)
- What data this feature consumes from other features (described by intent)
- If no floorplan exists, describe boundaries in terms of the PRD's capabilities and functional requirements

##### Dependencies

- **Hard dependencies**: features that must be complete before this one can start, with rationale
- **Soft dependencies**: features that can be developed in parallel but need interface agreement, with rationale
- External dependencies (third-party services, APIs)
- Cite which component interactions (from the floorplan's block diagram edges) create the dependency (e.g., "Depends on FT1 (hard) because COMP7 interacts with COMP4 (Auth Service) as shown in BD1")

##### Architectural Constraints

- Inherited from the parent PRD's architectural constraints (AC items) — keep original identifiers
- Feature-specific constraints continue numbering from the parent (e.g., if parent has AC1–AC4, new constraints start at AC5)

##### Non-Goals

- What is explicitly out of scope for this feature
- Inherit parent NG identifiers where applicable; new non-goals continue numbering from the parent

#### Present for review

Show the complete feature PRD to the user. Ask for confirmation before writing to disk.

<HARD-GATE>
Do NOT write the feature PRD to disk until the user has reviewed and approved it.
</HARD-GATE>

#### Write to disk

- Write to `docs/features/<feature-name>.org` (or `.md` based on Phase 0)
- Use kebab-case for the filename (e.g., `member-invitations.org`)

#### Move to the next feature

Repeat for each remaining feature.

### 3. Summary & next steps

After writing all feature PRDs and the index, tell the user:

- "Your PRD has been broken into epics and feature-level PRDs. Next steps you might consider:"
  - Review the dependency graph to identify parallelizable work and plan sprints
  - For large features with sub-features (FT.N): consider running this skill again on that feature PRD for deeper decomposition
  - Run the contracts skill to define entities, APIs, and protocols — the feature boundaries and typed dependencies will help contracts prioritize inter-feature interfaces
  - Run the tech-plan skill to make technology decisions
  - Begin implementation planning for features on the critical path with no dependencies first

---

## Identifier Reference Guide

All identifiers live in a **flat, project-wide namespace**. Every identifier is globally unique — "FR3" means the same thing whether referenced from the parent PRD, a feature PRD, or a task.

**Rules:**

1. **Inherited items** keep their original identifiers from the parent PRD
2. **New items** in a feature PRD continue numbering from the highest existing number across the entire project (parent + all feature PRDs)
3. **Sub-items** extend the parent identifier (FR3.1, FR3.2) and are naturally unique
4. Before assigning a new identifier, check the parent PRD and all existing feature PRDs to avoid collisions

| Entity | Prefix | Example | Notes |
|---|---|---|---|
| Epic | EP | EP1, EP2 | Major product capability area |
| Feature | FT | FT1, FT2 | Independently implementable unit |
| Sub-feature | FT | FT3.1, FT3.2 | Depth-2 only, use sparingly |
| Capability | CAP | CAP1 | |
| Persona | P | P1 | |
| Use Case | UC | UC1 | |
| User Journey | UJ | UJ1 | |
| User Story | parent.N | UC1.1, UJ2.3 | |
| Functional Req | FR | FR1 | |
| Goal | G | G1 | |
| Non-Goal | NG | NG1 | |
| Arch Constraint | AC | AC1 | |
| Verifiable sub-item | parent.N | FR1.1, CAP1.2, G1.1 | |

## Guardrails Against Over-Decomposition

These guardrails prevent the breakdown from becoming too granular:

1. **3–7 epics** — if outside this range, justify or adjust
2. **3–7 features per epic** — merge small features with siblings
3. **Max depth 2** — EP → FT → FT.N, no deeper
4. **Size heuristic** — if a feature is describable as a single user story with 2–3 acceptance criteria, it is too small — merge with a sibling
5. **Sub-features are the exception** — most features should not have sub-features; only use when a feature is genuinely too large for a single implementable unit
