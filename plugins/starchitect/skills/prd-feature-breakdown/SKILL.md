---
name: starchitect:prd-feature-breakdown
description: >
  Break a high-level PRD into feature-level PRDs, each scoped for direct conversion into tasks.
  Analyzes capabilities, use cases, user journeys, and functional requirements to identify coherent
  feature boundaries. Supports recursive decomposition — run again on a feature PRD if it's still too large.
  Triggers: "break down the PRD", "feature breakdown", "split PRD into features",
  "decompose PRD", "feature PRDs", "break this into features".
user-invocable: true
---

# PRD Feature Breakdown: High-Level PRD to Feature-Level PRDs

Break a high-level PRD into independently implementable feature-level PRDs. Each feature PRD is self-contained and specific enough to convert directly into a task hierarchy.

<HARD-GATE>
Do NOT skip to writing output. Every proposed feature list and every feature PRD must be presented to the user for review and confirmation before writing to disk.
</HARD-GATE>

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Discover inputs** — find parent PRD, architecture docs, existing tech, existing feature PRDs, determine output format
2. **Identify features** — analyze PRD items, group into coherent features, present to user for confirmation
3. **Generate feature PRDs** — produce each feature PRD one at a time, presenting each for review before writing
4. **Summary & next steps** — write index file, suggest next actions

---

## Phase 0: Discover Inputs

### Search for the parent PRD

Search these locations:

| Document | Locations to check |
|----------|--------------------|
| **PRD** | `docs/prd.md`, `docs/prd.org`, `docs/prd/`, `docs/prds/` |
| **Architecture** | `docs/architecture.md`, `docs/architecture.org`, `docs/architecture/` |
| **Technology choices** | `docs/technology.md`, `docs/technology.org`, `docs/technology/` |
| **Existing feature PRDs** | `docs/features/` |

Use Glob to check all locations. Read any documents found.

### If no PRD is found:

- Tell the user: "I couldn't find a PRD. I recommend starting with the prd-create skill to create one first."
- Stop here unless the user provides a PRD or points to one

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

## Phase 1: Identify Features

### Analyze the PRD

Read through the parent PRD and catalog:

- **Capabilities** (CAP items) — what the product can do
- **Use Cases** (UC items) — how users interact with it
- **User Journeys** (UJ items) — end-to-end user flows
- **Functional Requirements** (FR items) — specific behaviors

### Group into coherent features

Cluster related PRD items into features based on:

- **Bounded scope**: each feature should be independently implementable
- **Cohesion**: items that share data, UI, or domain concepts belong together
- **Minimal coupling**: features should have clear interface boundaries, not deep entanglement
- **Size**: each feature should be small enough to plan and implement as a unit — if it feels like it needs its own breakdown, flag it as "may need recursive decomposition"

### Present the proposed feature list

For each proposed feature, show:

- **Feature name**: short, descriptive name (e.g., "Authentication", "Billing", "Team Management")
- **Parent PRD coverage**: which CAP, UC, UJ, FR items this feature covers
- **Brief description**: one sentence explaining the feature's scope
- **Dependencies**: other features this one depends on or is depended on by
- **Size assessment**: normal or "large — may need recursive decomposition"

Use AskUserQuestion or present the list and ask the user to confirm, adjust, add, or remove features before proceeding.

<HARD-GATE>
Do NOT proceed to Phase 2 until the user has confirmed the feature list.
</HARD-GATE>

---

## Phase 2: Generate Feature PRDs (One at a Time)

For **each confirmed feature**, one at a time:

### 1. Produce the feature PRD

Follow this structure. The feature PRD uses the project's flat identifier namespace — items from the parent PRD keep their original identifiers, and any new items introduced in the feature PRD must use identifiers that don't collide with the parent or any other feature PRD.

#### Provenance

State which parent PRD items this feature covers:

> Covers: CAP1, CAP1.2, UC1, UC3, FR1, FR4, FR4.1–FR4.3

#### Overview

- What this feature is
- Scoped description — what it does and what it doesn't do
- Why it exists (the user problem it solves)

#### User Personas

- Relevant subset from the parent PRD
- Include only personas that interact with this feature

#### User Stories

- Organized into Use Cases (UC) and User Journeys (UJ), same convention as parent PRD
- Items from the parent PRD keep their original identifiers (e.g., if this feature covers UC3 and UC5, use UC3 and UC5 — do not renumber to UC1, UC2)
- New items not in the parent PRD continue from the highest existing number across the project (e.g., if UC5 is the highest in the parent, new use cases start at UC6)
- Each story has: identifier, title, description, acceptance criteria
- Sub-items extend the parent identifier (UC3.1, UC3.2) and are naturally unique

#### Functional Requirements

- Detailed, implementable requirements — inherit FR identifiers from parent PRD, continue numbering for new items
- Verifiable sub-items extend the parent (FR3.1, FR3.2, etc.)
- Specific enough to convert directly into tasks

#### Interface Boundaries

- What this feature exposes to other features (APIs, events, shared state)
- What this feature consumes from other features
- How this feature connects to the rest of the system

#### Dependencies

- Other features this one depends on (must be built first or concurrently)
- Other features that depend on this one
- External dependencies (third-party services, APIs)

#### Architectural Constraints

- Inherited from the parent PRD's architectural constraints (AC items) — keep original identifiers
- Feature-specific constraints continue numbering from the parent (e.g., if parent has AC1–AC4, new constraints start at AC5)

#### Non-Goals

- What is explicitly out of scope for this feature
- Inherit parent NG identifiers where applicable; new non-goals continue numbering from the parent

### 2. Present for review

Show the complete feature PRD to the user. Ask for confirmation before writing to disk.

<HARD-GATE>
Do NOT write the feature PRD to disk until the user has reviewed and approved it.
</HARD-GATE>

### 3. Write to disk

- Write to `docs/features/<feature-name>.org` (or `.md` based on Phase 0)
- Use kebab-case for the filename (e.g., `team-management.org`)

### 4. Move to the next feature

Repeat steps 1-3 for each remaining feature.

---

## Phase 3: Summary & Next Steps

### Write the index file

Create `docs/features/index.org` (or `.md`) listing all feature PRDs with their parent PRD coverage.

**Org-mode template (`docs/features/index.org`):**

```org
#+TITLE: Feature PRDs Index
#+DATE: YYYY-MM-DD

* Overview

Feature-level PRDs derived from [[file:../prd.org][parent PRD]].

* Features

| Feature | File | Parent PRD Coverage | Dependencies |
|---------+------+---------------------+--------------|
| Authentication | [[file:authentication.org][authentication.org]] | CAP1, UC1, FR1–FR3 | None |
| Team Management | [[file:team-management.org][team-management.org]] | CAP2, UC2, UC4, FR4–FR6 | Authentication |
| Billing | [[file:billing.org][billing.org]] | CAP3, UC3, FR7–FR9 | Authentication, Team Management |

* Dependency Graph

List features in implementation order based on dependencies.

1. Authentication (no dependencies)
2. Team Management (depends on: Authentication)
3. Billing (depends on: Authentication, Team Management)
```

**Markdown template (`docs/features/index.md`):**

```markdown
# Feature PRDs Index

**Date:** YYYY-MM-DD
**Parent PRD:** [PRD](../prd.md)

## Features

| Feature | File | Parent PRD Coverage | Dependencies |
|---------|------|---------------------|--------------|
| Authentication | [authentication.md](authentication.md) | CAP1, UC1, FR1–FR3 | None |
| Team Management | [team-management.md](team-management.md) | CAP2, UC2, UC4, FR4–FR6 | Authentication |
| Billing | [billing.md](billing.md) | CAP3, UC3, FR7–FR9 | Authentication, Team Management |

## Dependency Graph

Features in implementation order based on dependencies:

1. Authentication (no dependencies)
2. Team Management (depends on: Authentication)
3. Billing (depends on: Authentication, Team Management)
```

### Fill in the template

- Replace placeholder data with actual features from Phase 2
- Use today's date
- Link to the parent PRD
- Order the dependency graph for implementation sequencing

### Suggest next steps

After writing the index, tell the user:

- "Your PRD has been broken into feature-level PRDs. Next steps you might consider:"
  - For large features flagged during Phase 1: run this skill again on that feature PRD for recursive decomposition
  - Run the tech-plan skill to make technology decisions
  - Begin implementation planning for features with no dependencies first
  - Review the dependency graph to identify parallelizable work

## Identifier Reference Guide

All identifiers live in a **flat, project-wide namespace**. Every identifier is globally unique — "FR3" means the same thing whether referenced from the parent PRD, a feature PRD, or a task.

**Rules:**

1. **Inherited items** keep their original identifiers from the parent PRD
2. **New items** in a feature PRD continue numbering from the highest existing number across the entire project (parent + all feature PRDs)
3. **Sub-items** extend the parent identifier (FR3.1, FR3.2) and are naturally unique
4. Before assigning a new identifier, check the parent PRD and all existing feature PRDs to avoid collisions

| Entity | Prefix | Example |
|---|---|---|
| Capability | CAP | CAP1 |
| Persona | P | P1 |
| Use Case | UC | UC1 |
| User Journey | UJ | UJ1 |
| User Story | parent.N | UC1.1, UJ2.3 |
| Functional Req | FR | FR1 |
| Goal | G | G1 |
| Non-Goal | NG | NG1 |
| Arch Constraint | AC | AC1 |
| Verifiable sub-item | parent.N | FR1.1, CAP1.2, G1.1 |
