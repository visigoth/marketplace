---
name: starchitect:tdd
description: >
  Create Technical Design Documents (TDDs) that describe how a system or subsystem
  fulfills functional requirements. Defines internal technical approaches: algorithms,
  data structures, behavioral semantics, storage strategies, concurrency models, and
  error recovery. Introduces Technical Requirements (TRs) in a global namespace.
  Can target a specific component, a subsystem spanning components, or a cross-cutting
  concern not represented in the floorplan. Supports recursive decomposition — TDDs
  can identify sub-TDD candidates that are elaborated in the same or a later session.
  Triggers: "tdd", "technical design", "technical design document",
  "how should we build this", "design the internals of", "technical requirements".
user-invocable: true
---

# Technical Design Documents

Define how the system fulfills its functional requirements. PRDs define *what* to build. Contracts define the *interfaces* between components. TDDs define *how* — the internal technical approach: algorithms, data structures, behavioral semantics, storage strategies, concurrency models, and error recovery.

TDDs introduce **Technical Requirements (TRs)** — testable statements numbered in a global flat namespace (TR1, TR2, ... TRn). Downstream skills (beadify, test-plan) consume TRs alongside FRs and contract elements.

Your sole output is TDD documents. You do not create tasks, write code, generate implementation plans, or perform any action other than producing or refining TDDs.

<HARD-GATE>
Do NOT skip to writing output. Every design decision must be presented to the user for review and confirmation before recording. The complete TDD must be reviewed before writing to disk.
</HARD-GATE>

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Discover inputs** — find PRD, floorplan, contracts, technology choices, existing TDDs; scan codebase; determine output format
2. **Scope & complexity assessment** — user states subject, skill maps to inputs and assesses lightweight vs deep
3. **Technical design** — adaptive: draft-and-refine (lightweight) or interactive interview (deep)
4. **Extract technical requirements** — derive TR-numbered testable statements from the design
5. **Validate & write** — build traceability matrix, present complete TDD for approval, write to disk, offer sub-TDD continuation

---

## Phase 0: Discover Inputs

### Search for existing documents

Search these locations:

| Document | Locations to check |
|----------|--------------------|
| **PRD** | `docs/prd.md`, `docs/prd.org`, `docs/prd/`, `docs/prds/` |
| **Floorplan** | `docs/floorplan.md`, `docs/floorplan.org` |
| **Contracts** | `docs/contracts.md`, `docs/contracts.org`, `docs/contracts/` |
| **Technology choices** | `docs/technology.md`, `docs/technology.org`, `docs/technology/` |
| **Feature PRDs** | `docs/features/` |
| **Existing TDDs** | `docs/tdd.md`, `docs/tdd.org`, `docs/tdd/` |

Use Glob to check all locations. Read any documents found.

### If existing TDDs are found:

- Load the index and summarize its current state to the user
- Determine the highest existing TR number — new TRs will continue from there
- Identify sub-TDD candidates that are still unelaborated
- Ask the user what they want to do: add a new TDD, elaborate a sub-TDD candidate, or update an existing TDD

### If no floorplan is found:

- Tell the user: "I couldn't find a floorplan. The floorplan's component inventory is a primary input for TDDs. I recommend running the floorplan skill first."
- Stop here unless the user provides direction

### If no contracts are found:

- Tell the user: "I couldn't find contracts. Entity schemas, API boundaries, and event contracts inform TDD design decisions. I recommend running the contracts skill first."
- Stop here unless the user provides direction

### If no PRD is found:

- Tell the user: "I couldn't find a PRD. I recommend starting with the prd-create skill."
- Stop here unless the user provides a PRD or points to one

### Scan the codebase for existing implementation patterns

Existing code may already establish patterns that should inform the TDD. Scan for:

**Architectural patterns:**
- Module/package structure — how is the codebase organized?
- Framework conventions — what patterns does the framework enforce?
- Configuration management — how is config loaded and accessed?
- Dependency injection — is there a DI container or convention?

**State management:**
- Database access patterns (ORM, raw SQL, query builders)
- Cache implementations (Redis, in-memory, CDN)
- Session/state storage
- Queue/message broker usage

**Concurrency patterns:**
- Thread pool configurations
- Async runtime usage (tokio, asyncio, etc.)
- Locking strategies (mutexes, RWLocks, lock-free structures)
- Worker/job patterns

**Error handling:**
- Error types and hierarchies
- Retry/backoff implementations
- Circuit breaker patterns
- Logging and observability patterns

Use Glob and Grep to find these patterns. Read representative files to understand conventions.

### Load feature index

Check for `docs/features/index.org` or `docs/features/index.md`. If found, load it for context on feature boundaries and component-to-feature mappings.

**Do NOT load individual feature PRDs at this point.** Load them on-demand when their components come into scope during design.

### Determine output format

- Scan `docs/` for `.org` vs `.md` files (Glob for `docs/**/*.org` and `docs/**/*.md`)
- If org-mode files are present, use `.org`; otherwise use `.md`
- Tell the user which format you'll use

Output is always a split structure — a compact index file plus detail files (see Output Document Structure).

### Present discovered context

Summarize what you found:
- "From the floorplan, I see these components: [summary]"
- "From the contracts, I see these entities, APIs, and events: [summary]"
- "From the codebase, I see these existing patterns: [summary]"
- "Existing TDDs cover: [summary] (highest TR: TRn)" (if applicable)

---

## Phase 1: Scope & Complexity Assessment

### User states the subject

Ask the user what this TDD should cover. The subject may be:

- **A specific COMP** from the floorplan (e.g., "the cache service", "the auth gateway")
- **A subsystem** spanning multiple COMPs (e.g., "the real-time messaging subsystem")
- **A cross-cutting concern** not represented as a COMP (e.g., "the caching layer", "the retry framework", "the encryption strategy")
- **A sub-TDD candidate** identified by a parent TDD

If existing TDDs have unelaborated sub-TDD candidates, present them as options.

### Map to inputs

Once the subject is stated:

1. Identify which floorplan COMPs are relevant (may be none for cross-cutting concerns)
2. Identify which contract elements (ENTs, APIs, EVTs) are in play
3. Identify which FRs and architectural constraints (ACs) apply
4. Identify which technology choices constrain the design
5. If the subject relates to a feature, load that feature's PRD on-demand

### Assess complexity

Evaluate whether the subject warrants the **lightweight** or **deep** design path:

**Lightweight indicators:**
- Simple delegation or thin wrapper
- CRUD-only operations
- No concurrency concerns
- No state machine or complex lifecycle
- Few contract elements involved
- Well-established patterns in the codebase

**Deep indicators:**
- State machines or complex lifecycles
- Concurrency, parallelism, or distributed coordination
- Caching strategies with invalidation concerns
- Consensus protocols or distributed agreement
- Complex algorithms (scheduling, matching, ranking, optimization)
- Many contract elements or cross-component coordination
- Storage with encryption, partitioning, or retention requirements
- Novel patterns not established in the codebase

### Present scope summary

<HARD-GATE>
Present the following and get user confirmation before proceeding:

- **Subject**: what this TDD covers
- **Related COMPs**: floorplan components involved (or "cross-cutting" / "internal")
- **Relevant FRs**: functional requirements in scope
- **Relevant contract elements**: ENTs, APIs, EVTs that this design must satisfy
- **Relevant technology choices**: constraints from technology decisions
- **Complexity assessment**: lightweight or deep, with rationale
- **Design path**: draft-and-refine or interactive interview

The user may adjust scope, override the complexity assessment, or request a different design path.
</HARD-GATE>

---

## Phase 2: Technical Design

This phase is adaptive based on the complexity assessment from Phase 1. Both paths produce the same document sections.

### Lightweight path (draft-and-refine)

1. Draft a complete TDD based on inputs, codebase patterns, and technology choices
2. Present the draft for user review
3. Iterate until the user is satisfied
4. If the draft reveals more complexity than expected, the user can switch to the deep path

### Deep path (interactive interview)

Walk through each applicable design area, presenting 2-3 options with trade-offs at each decision point. Only cover areas relevant to the subject — skip sections that don't apply.

#### Core Design

The fundamental approach: algorithms, data structures, state management.

- What data structures best fit the access patterns implied by the contract elements?
- What algorithms are needed? (sorting, searching, matching, scheduling, etc.)
- How is state managed? (stateless, in-memory, persisted, distributed)
- What are the key invariants the design must maintain?

Present 2-3 options grounded in the FRs and contract elements. Recommend one with rationale.

#### Behavioral Semantics

How the system behaves under real conditions — not algorithms, not business logic:

- **Retry policies** — which operations are retried? What backoff strategy? Max attempts?
- **Timeout behavior** — what are the timeout thresholds? What happens on timeout?
- **Backpressure** — how does the system respond when overwhelmed? (shed load, queue, reject, degrade)
- **Cache invalidation** — what invalidation strategy? (TTL, event-driven, write-through, write-behind)
- **Ordering guarantees** — are operations ordered? Within what scope? (per-user, per-entity, global)
- **Graceful degradation** — what degrades first? What is the minimal viable behavior?
- **Circuit breaking** — which external dependencies are circuit-broken? Thresholds? Recovery?

Only cover what's relevant. A stateless computation has no cache invalidation section.

#### Storage & Data-at-Rest (conditional)

Only present when the subject stores data at rest:

- **Encoding formats** — how is data serialized for storage? (JSON, Protocol Buffers, Avro, binary, columnar)
- **Encryption** — is data encrypted at rest? What scheme? Key management strategy?
- **Compression** — is data compressed? What algorithm? Trade-off between CPU and storage.
- **Partitioning/sharding** — how is data distributed? Partition key? Rebalancing strategy?
- **Retention policies** — how long is data kept? Archival strategy? Deletion semantics.
- **Backup/recovery** — backup frequency? Recovery point objective? Recovery time objective?

#### Concurrency Model (where applicable)

Only present when concurrency is relevant:

- **Threading model** — thread-per-request, thread pool, event loop, actor model?
- **Async patterns** — futures/promises, callbacks, channels, streams?
- **Locking strategy** — mutexes, RWLocks, lock-free, optimistic concurrency?
- **Isolation** — how are concurrent operations isolated? (transactions, CAS, versioning)

#### Error Recovery

How the system handles failure:

- **Failure mode enumeration** — what can go wrong? (network, disk, OOM, corrupt data, upstream failure, timeout)
- **Detection** — how is each failure detected? (health checks, heartbeats, error codes, timeouts)
- **Response** — what happens for each failure? (retry, failover, degrade, alert, halt)
- **Recovery** — how does the system return to normal? (automatic, manual intervention, replay)

### Both paths also produce:

#### Trade-offs & Alternatives Considered

What other approaches were evaluated, why this approach was chosen, and what conditions would change the decision.

#### Deferred Improvements

Acknowledged enhancements that are intentionally deferred. For each:
- What the improvement is
- Why it's deferred (not needed yet, insufficient data, acceptable cost for now)
- **Trigger condition** — what would make this improvement necessary (e.g., "if write volume exceeds 10k/sec", "if cache miss rate exceeds 15%")

#### Sub-TDD Candidates

Areas identified as needing deeper design in a separate TDD:
- What the sub-TDD would cover
- Why it warrants separate treatment (complexity, independence, different expertise needed)
- Which COMPs or concerns it relates to

---

## Phase 3: Technical Requirements Extraction

Derive TRs from the technical design produced in Phase 2.

### TR format

Each TR is a testable, numbered statement:

- **Identifier**: TR1, TR2, ... (continue from highest existing TR determined in Phase 0)
- **Statement**: a testable technical requirement. Use RFC 2119 language (MUST, SHOULD, MAY) for precision. Example: "The cache MUST invalidate entries within 5 seconds of the source record being updated."
- **Source section**: which Technical Approach subsection this TR derives from (Core Design, Behavioral Semantics, Storage, Concurrency, Error Recovery)
- **FR/contract traces**: which FRs, ENTs, APIs, or EVTs this TR supports. May be empty for purely technical TRs.

### Purely technical TRs

TRs that have no FR or contract trace are expected and valid. These exist for technical correctness — performance guarantees, resource limits, operational constraints. Flag them clearly so downstream skills know they don't trace to business requirements:

- `TR14: The connection pool MUST NOT exceed 50 connections per database instance. [technical — no FR trace]`

### Present for confirmation

<HARD-GATE>
Present the complete TR list for user confirmation. The user may add, remove, reword, or reorder TRs. Ensure all TRs are testable — if a TR is vague ("the system should be fast"), push back and ask for a measurable threshold.
</HARD-GATE>

---

## Phase 4: Validate & Write

### Build traceability matrix

Map FRs and contract elements in scope to the TRs that address them:

| Input | Type | TR coverage |
|-------|------|-------------|
| FR3 | Functional Req | TR1, TR4 |
| ENT2 | Entity | TR2, TR5 |
| API1.3 | API Operation | TR3, TR6, TR7 |
| AC2 | Arch Constraint | TR8 |

Flag gaps: any FR or contract element in scope that has no TR coverage. These are areas where the technical design may be incomplete.

### Present complete TDD for approval

If the user has already reviewed the design (Phase 2) and TRs (Phase 3) individually, present only the traceability matrix as new material — do not re-present approved sections.

Prompt the user: "Would you like to review the traceability matrix before I write to disk, or should I skip ahead and write?"

If the user wants to review, present it and wait for approval. If they choose to skip, write directly.

### Write to disk — split structure

TDDs are **always** written as a split structure: a compact index file plus detail files in a subdirectory.

**Output structure:**

```
docs/tdd.{org,md}                ← compact index (~100-150 lines)
docs/tdd/
  <subject-name>.{org,md}        ← one TDD per subject (named by subject, kebab-case)
```

- Write the detail file to `docs/tdd/<subject-name>.{org,md}`
- Update the index at `docs/tdd.{org,md}` (or create if this is the first TDD)
- If updating an existing index, preserve entries for other TDDs — only add/update the current one

### Sub-TDD continuation

After writing, if sub-TDD candidates were identified:

1. Present the list of sub-TDD candidates
2. Offer to elaborate the first (or user-chosen) candidate immediately
3. If the user accepts, loop back to Phase 1 with the sub-TDD candidate as the subject
4. If the user declines, the candidates remain in the index with status "candidate" — they can be picked up in a later invocation

### Suggest next steps

After writing, tell the user:
- "Your TDD is documented. Next steps you might consider:"
  - Run the beadify skill to decompose features into tasks — tasks will reference TRs alongside FRs and contract elements
  - Run the test-plan skill — TRs become test targets for unit, integration, and e2e tests

---

## Output Document Structure

Output is always split into a compact **index file** and a **detail directory**. Templates below show both org-mode and markdown variants. Only produce the format determined in Phase 0.

### Index file template — org-mode (`docs/tdd.org`):

```org
#+TITLE: Technical Design Documents
#+DATE: YYYY-MM-DD

* Overview

Technical design documents for [project name].

PRD: [[file:prd.org][PRD]]
Floorplan: [[file:floorplan.org][Floorplan]]
Contracts: [[file:contracts.org][Contracts]]
Technology: [[file:technology.org][Technology Choices]] (if exists)

* TDD Inventory

| Subject | Scope | Related COMPs | TRs | Status | Parent |
|---------+-------+---------------+-----+--------+--------|
| [[file:tdd/cache-strategy.org][Cache Strategy]] | Cross-cutting | COMP2, COMP5 | TR1–TR8 | complete | — |
| [[file:tdd/auth-flow.org][Auth Flow]] | Component | COMP4 | TR9–TR15 | complete | — |
| Token Refresh | Sub-component | COMP4 | — | candidate | Auth Flow |

* TR Master List

| TR | Statement | Source TDD | Traces |
|----+-----------+------------+--------|
| TR1 | Cache MUST invalidate within 5s of source update | [[file:tdd/cache-strategy.org][Cache Strategy]] | FR3, ENT2 |
| TR2 | Cache hit ratio MUST exceed 90% under normal load | [[file:tdd/cache-strategy.org][Cache Strategy]] | technical |
| TR9 | Auth tokens MUST be validated in < 10ms | [[file:tdd/auth-flow.org][Auth Flow]] | FR7, API2.1 |

* Traceability Matrix

| Input | Type | TR coverage |
|-------+------+-------------|
| FR3 | Functional Req | TR1, TR4 |
| FR7 | Functional Req | TR9, TR12 |
| ENT2 | Entity | TR1, TR5 |
| API2.1 | API Operation | TR9, TR10 |
| AC2 | Arch Constraint | TR8 |
```

### Index file template — markdown (`docs/tdd.md`):

```markdown
# Technical Design Documents

**PRD:** [PRD](prd.md)
**Floorplan:** [Floorplan](floorplan.md)
**Contracts:** [Contracts](contracts.md)
**Technology:** [Technology Choices](technology.md) (if exists)
**Date:** YYYY-MM-DD

## Overview

Technical design documents for [project name].

## TDD Inventory

| Subject | Scope | Related COMPs | TRs | Status | Parent |
|---------|-------|---------------|-----|--------|--------|
| [Cache Strategy](tdd/cache-strategy.md) | Cross-cutting | COMP2, COMP5 | TR1–TR8 | complete | — |
| [Auth Flow](tdd/auth-flow.md) | Component | COMP4 | TR9–TR15 | complete | — |
| Token Refresh | Sub-component | COMP4 | — | candidate | Auth Flow |

## TR Master List

| TR | Statement | Source TDD | Traces |
|----|-----------|------------|--------|
| TR1 | Cache MUST invalidate within 5s of source update | [Cache Strategy](tdd/cache-strategy.md) | FR3, ENT2 |
| TR2 | Cache hit ratio MUST exceed 90% under normal load | [Cache Strategy](tdd/cache-strategy.md) | technical |
| TR9 | Auth tokens MUST be validated in < 10ms | [Auth Flow](tdd/auth-flow.md) | FR7, API2.1 |

## Traceability Matrix

| Input | Type | TR coverage |
|-------|------|-------------|
| FR3 | Functional Req | TR1, TR4 |
| FR7 | Functional Req | TR9, TR12 |
| ENT2 | Entity | TR1, TR5 |
| API2.1 | API Operation | TR9, TR10 |
| AC2 | Arch Constraint | TR8 |
```

### Detail file template — org-mode (`docs/tdd/<subject>.org`):

```org
#+TITLE: TDD: <Subject Name>
#+DATE: YYYY-MM-DD

* Subject & Scope

<What this TDD covers.>

Related COMPs: COMP2, COMP5 (or "cross-cutting" / "internal")
Parent TDD: <link to parent, or "—" if top-level>

* Context

** Functional Requirements

- FR3: <statement>
- FR7: <statement>

** Architectural Constraints

- AC2: <statement>

** Contract Elements

- ENT2: <name> — <relevance>
- API1.3: <name> — <relevance>
- EVT1: <name> — <relevance>

** Technology Choices

- <relevant technology decisions>

** Codebase Patterns

- <patterns discovered during scanning>

* Technical Approach

** Core Design

<Algorithms, data structures, state management.>

** Behavioral Semantics

<Retry policies, timeouts, backpressure, cache invalidation, ordering, degradation, circuit breaking.>

** Storage & Data-at-Rest

<Encoding, encryption, compression, partitioning, retention, backup/recovery.>
<Omit this section if the subject does not store data.>

** Concurrency Model

<Threading, async, locking, isolation.>
<Omit this section if concurrency is not relevant.>

** Error Recovery

<Failure modes and responses.>

* Technical Requirements

| TR | Statement | Source | Traces |
|----+-----------+--------+--------|
| TR1 | Cache MUST invalidate within 5s of source update | Behavioral Semantics | FR3, ENT2 |
| TR2 | Cache hit ratio MUST exceed 90% under normal load | Core Design | technical |

* Trade-offs & Alternatives Considered

<What was considered, why this approach was chosen, conditions to revisit.>

* Deferred Improvements

| Improvement | Why Deferred | Trigger Condition |
|-------------+--------------+-------------------|
| Write-through caching | Read-through sufficient for current load | Write volume > 10k/sec |

* Sub-TDD Candidates

| Candidate | Rationale | Status |
|-----------+-----------+--------|
| Token Refresh | Complex refresh/rotation logic warrants separate design | candidate |
```

### Detail file template — markdown (`docs/tdd/<subject>.md`):

```markdown
# TDD: <Subject Name>

**Date:** YYYY-MM-DD
**Related COMPs:** COMP2, COMP5 (or "cross-cutting" / "internal")
**Parent TDD:** <link to parent, or "—" if top-level>

## Subject & Scope

<What this TDD covers.>

## Context

### Functional Requirements

- FR3: <statement>
- FR7: <statement>

### Architectural Constraints

- AC2: <statement>

### Contract Elements

- ENT2: <name> — <relevance>
- API1.3: <name> — <relevance>
- EVT1: <name> — <relevance>

### Technology Choices

- <relevant technology decisions>

### Codebase Patterns

- <patterns discovered during scanning>

## Technical Approach

### Core Design

<Algorithms, data structures, state management.>

### Behavioral Semantics

<Retry policies, timeouts, backpressure, cache invalidation, ordering, degradation, circuit breaking.>

### Storage & Data-at-Rest

<Encoding, encryption, compression, partitioning, retention, backup/recovery.>
<Omit this section if the subject does not store data.>

### Concurrency Model

<Threading, async, locking, isolation.>
<Omit this section if concurrency is not relevant.>

### Error Recovery

<Failure modes and responses.>

## Technical Requirements

| TR | Statement | Source | Traces |
|----|-----------|--------|--------|
| TR1 | Cache MUST invalidate within 5s of source update | Behavioral Semantics | FR3, ENT2 |
| TR2 | Cache hit ratio MUST exceed 90% under normal load | Core Design | technical |

## Trade-offs & Alternatives Considered

<What was considered, why this approach was chosen, conditions to revisit.>

## Deferred Improvements

| Improvement | Why Deferred | Trigger Condition |
|-------------|--------------|-------------------|
| Write-through caching | Read-through sufficient for current load | Write volume > 10k/sec |

## Sub-TDD Candidates

| Candidate | Rationale | Status |
|-----------|-----------|--------|
| Token Refresh | Complex refresh/rotation logic warrants separate design | candidate |
```

---

## Identifier Reference Guide

TDD identifiers live alongside (but do not collide with) PRD, floorplan, contract, and feature identifiers. All are globally unique within the project.

| Entity | Prefix | Example | Notes |
|--------|--------|---------|-------|
| Technical Requirement | TR | TR1, TR2 | Testable statements about technical approach |

**PRD identifiers (do not reuse):** CAP, P, UC, UJ, FR, G, NG, AC

**Floorplan identifiers (do not reuse):** COMP, BD, DF, SL

**Contract identifiers (do not reuse):** ENT, API, EVT

**Feature identifiers (do not reuse):** EP, FT

---

## Important Constraints

- Your ONLY output is TDD documents (or interview questions when gathering context)
- Do NOT create implementation code, task hierarchies, test plans, or any artifact other than TDDs
- Do NOT invent requirements — if the PRD doesn't specify something, note the gap in the traceability matrix rather than filling it with assumptions
- TR numbering is global — scan existing TDDs before assigning new numbers
- When updating existing TDDs, preserve TR numbers that are still valid
- When a subject has no floorplan COMPs (cross-cutting concerns), that's expected — document as "cross-cutting" or "internal"
- Conditional sections (Storage & Data-at-Rest, Concurrency Model) should only appear when relevant to the subject — do not include empty sections
- Prefer precision over verbosity in all descriptions
- Use RFC 2119 language (MUST, SHOULD, MAY) in TR statements for testability
