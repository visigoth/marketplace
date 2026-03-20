# Starchitect

A workflow for transforming product ideas into implementation-ready task hierarchies. Rather than jumping from PRD to code, Starchitect guides you through architecture, technology choices, feature decomposition, contracts, and test planning — each step building on the last.

## Workflow Progression

```
┌─────────────┐     ┌───────────┐     ┌───────────┐     ┌───────────────────────┐
│  prd-create │────▶│ floorplan │────▶│ tech-plan │────▶│ prd-feature-breakdown │
└─────────────┘     └───────────┘     └───────────┘     └───────────────────────┘
                                                                    │
       ┌────────────────────────────────────────────────────────────┘
       ▼
┌───────────┐     ┌─────────┐     ┌───────────┐
│ contracts │────▶│ beadify │────▶│ test-plan │
└───────────┘     └─────────┘     └───────────┘
                                        │
                                        ▼
                                  ┌───────────┐
                                  │ tech-plan │  (revisit)
                                  └───────────┘
```

### 1. prd-create

Start with an idea, end with a structured PRD. The skill interviews you to gather requirements, constraints, and context, then produces an org-mode PRD ready for architectural analysis.

### 2. floorplan

Transform the PRD into an architectural floorplan: block diagrams showing component relationships, data flow diagrams showing how data moves through the system, and swim-lane diagrams showing multi-component protocols. Validates coverage against every PRD item.

### 3. tech-plan

Walk through each component in the floorplan and make technology decisions. The skill scans for existing tech in the repo, researches current options, presents trade-offs, and records your choices with rationale in `docs/technology.md`.

### 4. prd-feature-breakdown

Break the PRD into epics and features. Identifies epic boundaries from capabilities and floorplan clusters, decomposes into features, analyzes dependencies, and generates feature-level PRDs with typed dependency graphs.

### 5. contracts

Define the interfaces between components: entity schemas, API operations, wire protocols, and async events. Uses the floorplan's edges and data flows as the starting point — each edge becomes an API or event contract. The feature index helps prioritize which API boundaries need the cleanest interfaces.

### 6. beadify

Convert features into implementation task hierarchies in beads (`bd`). Each task is scoped to a single component so agents can work in parallel without file conflicts. Tasks reference contracts, floorplan elements, and FRs for full traceability.

### 7. test-plan

Produce test specifications from PRDs, contracts, and task hierarchies. Adds unit test specs to implementation tasks and creates separate tasks for integration, e2e, and UX tests.

### 8. tech-plan (revisit)

Test planning often surfaces new technology decisions — test frameworks, mocking tools, e2e infrastructure, CI integration. Run tech-plan again after test-plan to capture these choices.

## Usage

Each skill can be invoked by name or trigger phrase:

| Skill | Triggers |
|-------|----------|
| prd-create | "write a PRD", "product requirements", "I have an idea for..." |
| floorplan | "floorplan", "architecture diagram", "how do the components fit together" |
| tech-plan | "tech plan", "technology choices", "what tech should we use" |
| contracts | "contracts", "define the APIs", "entity model", "data schema" |
| prd-feature-breakdown | "feature breakdown", "break down the PRD", "split into features" |
| beadify | "beadify", "taskify", "create tasks", "break into tasks", "implementation tasks" |
| test-plan | "test plan", "test strategy", "test specs", "add tests" |

## Output Artifacts

| Skill | Output Location |
|-------|-----------------|
| prd-create | `docs/prd.org` or `docs/prd.md` |
| floorplan | `docs/floorplan.org` or `docs/floorplan.md` |
| tech-plan | `docs/technology.org` or `docs/technology.md` |
| contracts | `docs/contracts.org` or `docs/contracts.md` + `docs/contracts/` |
| prd-feature-breakdown | `docs/features/index.org` + `docs/features/*.org` |
| beadify | `.beads/` (via `bd` CLI) |
| test-plan | `.beads/` (via `bd` CLI) + `docs/testing/` |

## Philosophy

Starchitect is about doing the thinking before the typing. Each skill is a checkpoint that forces you to make decisions explicitly rather than discovering them mid-implementation. The artifacts form a chain of traceability: every task traces back to FRs, every FR traces back to capabilities, every capability traces back to the original product vision.

## Superpowers

Starchitect dovetails with @obra's [Superpowers](https://github.com/obra/superpowers). For example, **superpowers:brainstorm** can be used to flesh out an idea prior to entering the Starchitect workflow. **starchitect:prd-create** can be used to formalize the output of **brainstorm**. While you could continue to use **superpowers:writing-plans**, Starchitect breaks down the various aspects of writing plans into the various pieces: a floorplan, a technical plan, the features, the contracts within and around those features, and finally, tasks. Every item gets an identifier that can be cross-referenced, and files are kept to reasonable sizes to avoid context overload. Starchitect ensures there is an audit trail from task all the way back to product vision. It's still possible to use **superpowers**' development skills, though they aren't tuned to Starchitect's output.
