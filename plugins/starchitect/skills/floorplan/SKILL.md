---
name: starchitect:floorplan
description: >
  Create an architectural floorplan from a PRD. Produces block diagrams (component relationships),
  data flow diagrams (how data moves through the system), and swim-lane diagrams (multi-component
  conversations and protocols). Includes both high-level system views and second-level zoom-ins.
  Validates the floorplan against the PRD to ensure full coverage.
  Triggers: "floorplan", "architecture diagram", "architectural floorplan", "component diagram",
  "system architecture", "draw the architecture", "how do the components fit together".
user-invocable: true
---

# Floorplan: PRD to Architectural Floorplan

Produce an architectural floorplan from a PRD (and existing code, if present). The floorplan contains three kinds of diagrams at two levels of detail, plus a traceability matrix that validates every PRD item is architecturally accounted for.

Your sole output is the floorplan document. You do not create tasks, write code, generate implementation plans, or perform any action other than producing or refining the floorplan.

<HARD-GATE>
Do NOT skip to writing output. The component inventory must be presented to the user for review and confirmation before generating diagrams. The complete floorplan must be presented for review before writing to disk.
</HARD-GATE>

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Discover inputs** — find PRD, existing docs, scan codebase for architectural facts, determine output format
2. **Extract component inventory** — identify components from PRD and code, present to user for confirmation
3. **Generate diagrams** — produce block diagrams, data flow diagrams, and swim-lane diagrams at both levels
4. **Validate & assess** — build traceability matrix, run validation rules, produce quality assessment
5. **Write output** — present complete floorplan for review, write to disk after approval

---

## Phase 0: Discover Inputs

### Search for existing documents

Search these locations:

| Document | Locations to check |
|----------|--------------------|
| **PRD** | `docs/prd.md`, `docs/prd.org`, `docs/prd/`, `docs/prds/` |
| **Feature PRDs** | `docs/features/` |
| **Existing floorplan** | `docs/floorplan.md`, `docs/floorplan.org` |
| **Architecture** | `docs/architecture.md`, `docs/architecture.org`, `docs/architecture/` |
| **Technology choices** | `docs/technology.md`, `docs/technology.org`, `docs/technology/` |

Use Glob to check all locations. Read any documents found.

### If an existing floorplan is found:

- Load it and summarize its current state to the user
- Ask the user what they want to do: update it, extend it, or start fresh

### If no PRD is found:

- Tell the user: "I couldn't find a PRD. I recommend starting with the prd-create skill to create one first."
- Stop here unless the user provides a PRD or points to one

### Scan the codebase for architectural facts

Existing code constrains the floorplan. Scan for:

**Dependency manifests** — to identify languages and libraries:
- `package.json`, `go.mod`, `Cargo.toml`, `Gemfile`, `requirements.txt`, `pyproject.toml`, `Pipfile`
- `Podfile`, `build.gradle`, `build.gradle.kts`, `pom.xml`, `mix.exs`, `composer.json`
- `pubspec.yaml`, `CMakeLists.txt`, `Makefile`, `Package.swift`, `*.csproj`, `*.sln`

**Service boundaries** — to identify deployed components:
- `docker-compose.yml`, `Dockerfile`, `Procfile`
- `kubernetes/`, `k8s/`, `*.yaml` with kind: Deployment/Service
- Serverless configs: `serverless.yml`, `sam-template.yaml`, `app.yaml`
- `terraform/`, `*.tf`, `pulumi/`

**API surface** — to identify communication patterns:
- Route/controller files (search for router, handler, controller patterns)
- Proto files (`*.proto`) for gRPC
- GraphQL schemas (`*.graphql`, `*.gql`)
- OpenAPI/Swagger specs (`openapi.yaml`, `swagger.json`)

**Data layer** — to identify data stores and flows:
- Migration files, ORM model definitions
- Schema files, database config
- Queue/event config (Redis, Kafka, RabbitMQ, SQS patterns)

**Module structure** — to identify internal boundaries:
- Top-level directory structure under `src/`, `lib/`, `app/`, `pkg/`, `internal/`
- Package/module declarations

Use Glob to find these files. Read the ones that exist and extract architectural facts.

### Present discovered context

Summarize what you found to the user:
- "From the PRD, I see these capabilities and requirements: [summary]"
- "From the codebase, I see these existing components and patterns: [summary]"
- "These existing structures will constrain the floorplan."

### Determine output format

- Scan `docs/` for `.org` vs `.md` files (Glob for `docs/**/*.org` and `docs/**/*.md`)
- If org-mode files are present, output `docs/floorplan.org`; otherwise output `docs/floorplan.md`
- Tell the user which format you'll use

---

## Phase 1: Extract Component Inventory

### Analyze the PRD

Read through the PRD and catalog:

- **Capabilities** (CAP items) — what the product can do
- **Use Cases** (UC items) — how users interact with it
- **User Journeys** (UJ items) — end-to-end user flows
- **Functional Requirements** (FR items) — specific behaviors
- **Architectural Constraints** (AC items) — technical boundaries
- **Non-Goals** (NG items) — explicit scope boundaries

### Identify components

From the PRD requirements and codebase analysis, identify the architectural components needed. For each component:

- **Identifier**: COMP1, COMP2, etc.
- **Name**: short, descriptive (e.g., "API Gateway", "Auth Service", "User Database")
- **Type**: service, library, database, message queue, external system, UI, CDN, cache, or other
- **Responsibility**: 1-2 sentences on what this component does
- **PRD provenance**: which CAP, FR, UC, UJ, or AC items justify this component's existence
- **Code status**: "existing" (found in codebase), "new" (required by PRD but not yet implemented), or "external" (third-party service)

**Rules for component identification:**

1. Components already present in the codebase are carried forward — do not reinvent them. Use the names and boundaries that exist.
2. New components are identified only where the PRD requires capabilities not yet present in code.
3. External systems referenced by the PRD (payment processors, email services, third-party APIs) are listed as components with type "external."
4. Infrastructure components (load balancers, message brokers, caches) are included only when justified by a PRD requirement (FR, AC, or implied by a UC/UJ's performance or reliability needs).
5. Every component must trace to at least one PRD item. No phantom components.

### Present for confirmation

Show the component inventory table to the user. Ask them to confirm, add, remove, or adjust components before proceeding.

<HARD-GATE>
Do NOT proceed to Phase 2 until the user has confirmed the component inventory.
</HARD-GATE>

---

## Phase 2: Generate Diagrams

Produce three kinds of diagrams, each at two levels of detail. Use Mermaid syntax for all diagrams — it is text-based, version-controllable, and renders in most documentation tooling.

### Diagram Kind 1: Block Diagrams (BD)

Block diagrams show components as blocks and directed arrows for communication channels between them.

#### Level 1: System Block Diagram (BD1)

A single diagram showing all components (COMP references) and their connections.

- Each node is a component from the inventory, labeled with its COMP identifier and name
- Each directed edge is labeled with the communication mechanism (HTTP, gRPC, events, WebSocket, shared DB, SDK call, etc.)
- Group related components visually using Mermaid subgraphs where natural clusters exist
- Use Mermaid `graph` or `flowchart` syntax

**Example structure:**

```
#+BEGIN_SRC mermaid
graph LR
    subgraph Frontend
        COMP1[COMP1: Web App]
        COMP2[COMP2: Mobile App]
    end
    subgraph Backend
        COMP3[COMP3: API Gateway]
        COMP4[COMP4: Auth Service]
    end
    COMP5[(COMP5: User DB)]

    COMP1 -->|HTTPS| COMP3
    COMP2 -->|HTTPS| COMP3
    COMP3 -->|gRPC| COMP4
    COMP4 -->|SQL| COMP5
#+END_SRC
```

#### Level 2: Subsystem Block Diagrams (BD1.1, BD1.2, ...)

One block diagram per logical cluster that is complex enough to warrant decomposition. Each Level-2 diagram zooms into the internal structure of a cluster from Level 1.

- Only produce Level-2 diagrams for clusters with 3+ internal components or non-trivial internal communication
- Each Level-2 diagram must be a strict refinement of its cluster in BD1 — no components or connections that contradict the Level-1 view
- Show internal subcomponents, internal communication paths, and the external interfaces (edges entering/leaving the cluster)

### Diagram Kind 2: Data Flow Diagrams (DF)

Data flow diagrams show how data moves through the system — sources, transformations, stores, and sinks.

#### Level 1: System Data Flow (DF1)

A single diagram showing major data paths across the full system.

- Nodes are components (COMP references), data stores (databases, caches, queues), and external data sources/sinks
- Edges are labeled with what data moves along them (e.g., "user credentials", "order events", "session tokens")
- Show the direction of data flow
- Use Mermaid `flowchart` syntax with data annotations on edges

#### Level 2: Data Flow Details (DF1.1, DF1.2, ...)

Per-flow diagrams that trace a specific data lifecycle through the system.

- One diagram per significant data path (e.g., "user registration flow", "order fulfillment pipeline", "notification delivery")
- Show transformations, validation points, and data shape changes at each step
- Include error paths where data gets rejected or rerouted
- Only produce Level-2 diagrams for data paths that involve 3+ components or non-trivial transformations

### Diagram Kind 3: Swim-Lane Diagrams (SL)

Swim-lane diagrams show multi-component conversations — the sequence of messages, requests, and responses that implement a specific interaction.

- Use Mermaid `sequenceDiagram` syntax
- Each participant is a component (COMP reference) from the inventory
- Show request/response pairs, async messages, protocol steps, and error responses
- Include timing-relevant details (e.g., "async", "fire-and-forget", "blocking")

**Coverage rules:**

- Every user journey (UJ) that involves 3+ components MUST have its own swim-lane diagram
- Every use case (UC) that involves a multi-step protocol MUST have its own swim-lane diagram
- Additional swim-lane diagrams for critical system interactions not directly tied to a single UJ/UC (e.g., health checks, cache invalidation, event propagation) should be included when they are architecturally significant

**Identifiers:** SL1, SL2, etc. Each diagram gets a title describing the interaction (e.g., "SL1: User Authentication Flow", "SL2: Order Placement and Fulfillment").

### Determine which Level-2 diagrams to produce

Not every Level-1 diagram needs a Level-2 breakdown. Produce Level-2 diagrams only when:

1. A cluster has 3+ internal components with non-trivial communication
2. A data path involves 3+ components or non-trivial transformations
3. The Level-1 diagram abstracts away detail that is important for understanding the architecture

Present the diagrams to the user as you generate them. Adjust based on feedback before moving to Phase 3.

---

## Phase 3: Validate & Assess

### Build the PRD Traceability Matrix

Create a table that maps every PRD item to the architectural elements that realize it. The matrix answers: **"What does it take to make this requirement real?"**

Each row is a PRD item. Each row must map to a sufficient set of components, diagrams, and flows:

| PRD Item | Type | What it takes to realize it |
|----------|------|-----------------------------|
| CAP1 | Capability | COMP1, COMP3, COMP5; DF1.2; SL1 |
| FR3 | Functional Req | COMP2, COMP4; DF2; SL3 |
| UC2 | Use Case | SL2 (full conversation); COMP1, COMP3, COMP5 |
| AC1 | Arch Constraint | Honored by: COMP1-COMP4 are stateless; state in COMP5 |

**Every PRD item must have at least one entry in the "What it takes" column.** Empty rows are validation failures.

### Run validation rules

Evaluate the floorplan against each rule below. For each rule, state pass or fail with a brief note.

#### Completeness rules (nothing missing)

1. **Capability coverage** — Every CAP (and its sub-items) in the PRD maps to at least one COMP and appears in at least one diagram.
2. **Functional requirement coverage** — Every FR maps to at least one COMP and appears in at least one diagram (BD, DF, or SL).
3. **User journey representation** — Every UJ that involves 3+ components has its own swim-lane diagram (SL).
4. **Use case representation** — Every UC traces to at least one data flow (DF) or swim-lane (SL).
5. **Data entity coverage** — Every data entity implied by the PRD's functional requirements appears in at least one data flow diagram.

#### Consistency rules (nothing contradictory)

6. **Constraint compliance** — Every architectural constraint (AC) is visibly honored. For each AC, state *how* the floorplan satisfies it.
7. **No phantom components** — Every COMP traces back to at least one PRD item (CAP, FR, UC, UJ, or AC). Components that exist for architectural reasons (load balancers, message brokers) cite the AC or FR that justifies them.
8. **Non-goal boundary enforcement** — No component, flow, or diagram implements functionality listed as a non-goal (NG). If a component incidentally touches a non-goal area, note and scope it.
9. **Arrow justification** — Every directed edge in a block diagram is justifiable from a FR, UC, or UJ.

#### Coherence rules (diagrams agree with each other)

10. **Cross-diagram consistency** — Components in block diagrams, data flows, and swim lanes use the same COMP identifiers. A component in a swim lane must also appear in the block diagram.
11. **Level consistency** — Level-2 diagrams are strict refinements of their Level-1 parent. No component or connection in a Level-2 diagram contradicts the Level-1 view.
12. **Identifier namespace integrity** — All identifiers (COMP, BD, DF, SL) are unique, follow the prefix convention, and use dot notation only for sub-items. No collisions with PRD identifiers (CAP, FR, UC, UJ, AC, G, NG, P).

### Produce the Quality Assessment

Write the assessment as the final section of the floorplan document. List each rule with its pass/fail status and a brief note. If any rule fails, note the gap and what would need to change to close it.

---

## Phase 4: Write Output

### Present the complete floorplan

Show the user the complete floorplan document — all sections, all diagrams, the traceability matrix, and the quality assessment.

<HARD-GATE>
Do NOT write to disk until the user has reviewed and approved the floorplan.
</HARD-GATE>

### Write to disk

Write to `docs/floorplan.org` (or `docs/floorplan.md` based on Phase 0).

### Suggest next steps

After writing, tell the user:
- "Your architectural floorplan is documented. Next steps you might consider:"
  - Run the tech-plan skill to make technology decisions informed by this architecture
  - Run the prd-feature-breakdown skill to decompose the PRD into feature-level PRDs aligned with these components
  - Review with your team — the traceability matrix makes it easy to verify coverage
  - Use the swim-lane diagrams as the basis for API contracts and interface definitions

---

## Output Document Structure

The floorplan document follows this structure. Use org-mode or markdown based on the format determined in Phase 0.

### Org-mode template (`docs/floorplan.org`):

```org
#+TITLE: Architectural Floorplan
#+DATE: YYYY-MM-DD

* Overview

Brief summary of the system being modeled.

PRD: [[file:prd.org][PRD]]
Technology: [[file:technology.org][Technology Choices]] (if exists)

* Component Inventory

| ID | Name | Type | Responsibility | PRD Provenance | Status |
|----+------+------+----------------+----------------+--------|
| COMP1 | Web App | UI | User-facing web interface | CAP1, UC1 | existing |
| COMP2 | API Gateway | service | Routes and authenticates requests | FR1, FR2, AC1 | new |

* Block Diagrams

** System block diagram (BD1)

#+BEGIN_SRC mermaid
graph LR
    ...
#+END_SRC

** [Subsystem name] (BD1.1)

#+BEGIN_SRC mermaid
graph LR
    ...
#+END_SRC

* Data Flow Diagrams

** System data flow (DF1)

#+BEGIN_SRC mermaid
flowchart LR
    ...
#+END_SRC

** [Flow name] (DF1.1)

#+BEGIN_SRC mermaid
flowchart LR
    ...
#+END_SRC

* Swim-Lane Diagrams

** [Interaction name] (SL1)

#+BEGIN_SRC mermaid
sequenceDiagram
    ...
#+END_SRC

** [Interaction name] (SL2)

#+BEGIN_SRC mermaid
sequenceDiagram
    ...
#+END_SRC

* PRD Traceability Matrix

| PRD Item | Type | What it takes to realize it |
|----------+------+-----------------------------|
| CAP1 | Capability | COMP1, COMP3; DF1.1; SL1 |
| FR1 | Functional Req | COMP2, COMP4; SL2 |

* Floorplan Quality Assessment

| # | Rule | Status | Notes |
|---+------+--------+-------|
| 1 | Capability coverage | Pass | All CAP items mapped |
| 2 | Functional requirement coverage | Pass | All FR items mapped |
| 3 | User journey representation | Pass | UJ1-UJ3 each have SL diagrams |
| 4 | Use case representation | Pass | All UC items traced |
| 5 | Data entity coverage | Pass | All data entities in DF diagrams |
| 6 | Constraint compliance | Pass | AC1: stateless services confirmed |
| 7 | No phantom components | Pass | All COMP items traced to PRD |
| 8 | Non-goal boundary enforcement | Pass | No NG items implemented |
| 9 | Arrow justification | Pass | All edges traced to FR/UC/UJ |
| 10 | Cross-diagram consistency | Pass | COMP IDs consistent across diagrams |
| 11 | Level consistency | Pass | L2 diagrams refine L1 |
| 12 | Identifier namespace integrity | Pass | No collisions |
```

### Markdown template (`docs/floorplan.md`):

```markdown
# Architectural Floorplan

**PRD:** [PRD](prd.md)
**Technology:** [Technology Choices](technology.md) (if exists)
**Date:** YYYY-MM-DD

## Overview

Brief summary of the system being modeled.

## Component Inventory

| ID | Name | Type | Responsibility | PRD Provenance | Status |
|----|------|------|----------------|----------------|--------|
| COMP1 | Web App | UI | User-facing web interface | CAP1, UC1 | existing |
| COMP2 | API Gateway | service | Routes and authenticates requests | FR1, FR2, AC1 | new |

## Block Diagrams

### System block diagram (BD1)

```mermaid
graph LR
    ...
```

### [Subsystem name] (BD1.1)

```mermaid
graph LR
    ...
```

## Data Flow Diagrams

### System data flow (DF1)

```mermaid
flowchart LR
    ...
```

### [Flow name] (DF1.1)

```mermaid
flowchart LR
    ...
```

## Swim-Lane Diagrams

### [Interaction name] (SL1)

```mermaid
sequenceDiagram
    ...
```

### [Interaction name] (SL2)

```mermaid
sequenceDiagram
    ...
```

## PRD Traceability Matrix

| PRD Item | Type | What it takes to realize it |
|----------|------|-----------------------------|
| CAP1 | Capability | COMP1, COMP3; DF1.1; SL1 |
| FR1 | Functional Req | COMP2, COMP4; SL2 |

## Floorplan Quality Assessment

| # | Rule | Status | Notes |
|---|------|--------|-------|
| 1 | Capability coverage | Pass | All CAP items mapped |
| 2 | Functional requirement coverage | Pass | All FR items mapped |
| 3 | User journey representation | Pass | UJ1-UJ3 each have SL diagrams |
| 4 | Use case representation | Pass | All UC items traced |
| 5 | Data entity coverage | Pass | All data entities in DF diagrams |
| 6 | Constraint compliance | Pass | AC1: stateless services confirmed |
| 7 | No phantom components | Pass | All COMP items traced to PRD |
| 8 | Non-goal boundary enforcement | Pass | No NG items implemented |
| 9 | Arrow justification | Pass | All edges traced to FR/UC/UJ |
| 10 | Cross-diagram consistency | Pass | COMP IDs consistent across diagrams |
| 11 | Level consistency | Pass | L2 diagrams refine L1 |
| 12 | Identifier namespace integrity | Pass | No collisions |
```

---

## Identifier Reference Guide

Floorplan identifiers live alongside (but do not collide with) PRD identifiers. All are globally unique within the project.

| Entity | Prefix | Example | Notes |
|--------|--------|---------|-------|
| Component | COMP | COMP1, COMP2 | Flat list — no dot notation |
| Block Diagram | BD | BD1 | Level-1 system diagram |
| Block Diagram (L2) | BD | BD1.1, BD1.2 | Zoom-in under BD1 |
| Data Flow | DF | DF1 | Level-1 system flow |
| Data Flow (L2) | DF | DF1.1, DF1.2 | Zoom-in under DF1 |
| Swim Lane | SL | SL1, SL2 | One per significant interaction |

**PRD identifiers (do not reuse):** CAP, P, UC, UJ, FR, G, NG, AC

---

## Important Constraints

- Your ONLY output is the floorplan document (or interview questions when gathering context)
- Do NOT create implementation plans, code, task lists, or anything outside the floorplan
- Do NOT invent requirements — if the PRD doesn't specify something, note the gap in the quality assessment rather than filling it with assumptions
- When updating an existing floorplan, preserve components and identifiers that are still valid
- Prefer precision over verbosity in all descriptions
- Diagrams must be syntactically valid Mermaid — verify node names are properly quoted if they contain special characters
