---
name: starchitect:tech-plan
description: >
  Go from PRD and floorplan to documented technology choices. Scans the repo for existing tech,
  uses the floorplan's component inventory to identify what needs decisions, researches current options,
  and walks through choices interactively. Triggers: "tech plan", "technology choices",
  "what tech should we use", or when a PRD is ready for technology decisions.
user-invocable: true
---

# Tech Plan: PRD and Floorplan to Technology Choices

Go from a PRD and architectural floorplan to a documented set of technology decisions. The floorplan's component inventory (COMP identifiers) defines what needs technology choices. This skill walks through each component or workload, researches current options, and produces a `docs/technology.md` (or `.org`) file recording every choice with rationale.

<HARD-GATE>
Do NOT skip to writing the output document. Every decision must be presented to the user with options and trade-offs, and the user must choose before you record it.
</HARD-GATE>

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Detect resume / assess input** — check for existing documents, determine output format
2. **Discover context** — scan repo for existing tech, load PRD and floorplan
3. **Identify decision categories** — present categories to user for confirmation
4. **Research & decide** — walk through each category interactively, one at a time
5. **Document & commit** — write technology choices file and commit

---

## Phase 0: Detect Resume / Assess Input

### Check for existing documents

Search these locations:

| Document | Locations to check |
|----------|--------------------|
| **Existing tech choices** | `docs/technology.md`, `docs/technology.org`, `docs/technology/` |
| **PRD** | `docs/prd.md`, `docs/prd.org`, `docs/prd/`, `docs/prds/` |
| **Floorplan** | `docs/floorplan.md`, `docs/floorplan.org` |

Use Glob to check all locations. Read any documents found.

### If an existing tech choices doc is found:

- Load it and summarize its current state to the user
- Ask the user what they want to do: update existing decisions, add new ones, or start fresh

### If no PRD is found and the user hasn't described requirements:

- Tell the user: "I couldn't find a PRD. I recommend starting with brainstorming to clarify requirements first."
- Stop here unless the user provides direction

### Determine output format:

- Scan the repo for `.org` files vs `.md` files (Glob for `docs/**/*.org` and `docs/**/*.md`)
- If org-mode files are present, use `docs/technology.org`; otherwise use `docs/technology.md`
- Tell the user which format you'll use

---

## Phase 1: Discover Context

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

Use Glob to find these files, then Read the ones that exist.

### Present detected tech to the user

Summarize what you found: "I see you're using [technologies]. Should we carry these forward as given, or are any of them up for reconsideration?"

Wait for user confirmation before proceeding.

### Identify components from floorplan

- If a floorplan exists, use its component inventory (COMP identifiers) as the definitive list of components needing technology decisions. The floorplan's components have already been validated against the PRD — do not re-derive them.
- If no floorplan exists, tell the user: "I couldn't find a floorplan. I recommend running the floorplan skill first to define the component inventory. I can infer components from the PRD, but the floorplan provides a validated, complete component list." Then ask the user whether to proceed by inferring from the PRD or to stop and create a floorplan first.
- List the components that need technology decisions

---

## Phase 2: Identify Decision Categories

### Fixed checklist

Go through this list and determine which categories are relevant based on the PRD and components:

- Programming language(s)
- Web framework(s)
- Database / data store(s)
- Message queue / event streaming
- API style (REST, GraphQL, gRPC, etc.)
- Frontend framework(s)
- Mobile framework(s)
- Cloud provider / hosting
- CI/CD
- Observability / monitoring
- Authentication / authorization
- Infrastructure as code

### Extend with PRD-specific categories

- Read the PRD for any technology needs outside the fixed checklist (e.g., ML/AI frameworks, payment processing, search engine, file storage, CDN)
- Add those to the list

### Prioritize and present

- Order categories by criticality: decisions that block other decisions come first
- Mark categories where existing tech already covers the need
- Present the full list to the user using AskUserQuestion or similar — ask them to confirm, remove, or add categories before proceeding

---

## Phase 3: Research & Decide (per category)

For **each category**, one at a time:

### 1. Research current options

- Use WebSearch to find current comparisons, benchmarks, and ecosystem health for the relevant options
- Focus on options that align with existing tech in the repo (e.g., if using Go, prefer Go-native libraries)
- Consider the specific requirements from the PRD

### 2. Present 2-4 options

For each option, cover:

- **What it is** (one sentence)
- **Strengths** for this project's needs
- **Weaknesses** or trade-offs
- **Ecosystem fit** with existing/chosen tech
- **Community health** (maintenance status, adoption trend)

Note which option aligns best with existing tech.

### 3. Make a recommendation

- State which option you recommend and why, grounded in the PRD's requirements
- Be direct — "I recommend X because..." not "You might consider..."

### 4. Ask user to choose

- Use AskUserQuestion with the options
- Include your recommendation as the first option with "(Recommended)" label

### 5. Record the decision

- Note the choice, rationale, and alternatives considered
- Move to the next category

### Multiple technologies per category

When the PRD has multiple components that need different tech in the same category (e.g., Swift for iOS + Kotlin for Android), present and decide per component.

### Group by component

When there are multiple distinct components/workloads, group the decisions so related choices are made together (e.g., all backend decisions, then all frontend decisions).

---

## Phase 4: Document & Commit

### Write the technology choices document

Use the appropriate template below based on the output format determined in Phase 0.

**Markdown template (`docs/technology.md`):**

```markdown
# Technology Choices

**PRD:** [link to PRD if exists]
**Date:** YYYY-MM-DD

## Context

Brief summary of what's being built.

## Existing Technology

Technologies detected in the repo that are carried forward.

| Technology | Category | Notes |
|-----------|----------|-------|
| Go 1.22 | Language | Detected from go.mod |

## Decisions

### [Component/Workload Name]

| Category | Choice | Rationale | Alternatives Considered |
|----------|--------|-----------|------------------------|
| Language | Go | [why] | Rust, Java |
| Database | PostgreSQL | [why] | MySQL, CockroachDB |

### [Another Component]

| Category | Choice | Rationale | Alternatives Considered |
|----------|--------|-----------|------------------------|
| Language | Swift | [why] | Flutter, React Native |

## Deferred Decisions

Decisions identified but not yet made, with notes on when they should be revisited.

| Category | Component | Notes |
|----------|-----------|-------|
| CI/CD | All | Decide after initial implementation |
```

**Org-mode template (`docs/technology.org`):**

```org
#+TITLE: Technology Choices
#+DATE: YYYY-MM-DD

* Context

Brief summary of what's being built.

PRD: [[file:prd.org][PRD]]

* Existing Technology

Technologies detected in the repo that are carried forward.

| Technology | Category | Notes              |
|------------+----------+--------------------|
| Go 1.22    | Language | Detected from go.mod |

* Decisions

** [Component/Workload Name]

| Category | Choice     | Rationale | Alternatives Considered |
|----------+------------+-----------+------------------------|
| Language | Go         | [why]     | Rust, Java              |
| Database | PostgreSQL | [why]     | MySQL, CockroachDB      |

** [Another Component]

| Category | Choice | Rationale | Alternatives Considered |
|----------+--------+-----------+------------------------|
| Language | Swift  | [why]     | Flutter, React Native   |

* Deferred Decisions

Decisions identified but not yet made, with notes on when they should be revisited.

| Category | Component | Notes                          |
|----------+-----------+--------------------------------|
| CI/CD    | All       | Decide after initial implementation |
```

### Fill in the template

- Replace all placeholder text with actual decisions from Phase 3
- Use today's date
- Link to the PRD file if it exists
- Include all decided categories under the appropriate component
- List any categories that were identified but deferred

### Commit the document

- Stage the new file
- Commit with message: "docs: add technology choices document"

### Suggest next steps

After committing, suggest the user's next steps:
- "Your technology choices are documented. Next steps you might consider:"
  - Review with your team
  - If no floorplan exists, run the floorplan skill to define the architectural structure
  - Run the prd-feature-breakdown skill to decompose the PRD into feature-level PRDs
  - Create an implementation plan based on these choices
