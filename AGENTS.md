# AGENTS.md

## Overview

This is a plugin marketplace repository. It hosts plugins that can be installed by coding agents (Claude Code, Codex, Gemini CLI, and others) via their respective plugin/extension systems.

Plugins in this marketplace are designed to be **agent-agnostic** where possible. Skills and agents should avoid hard-coding assumptions about the host agent's capabilities. When agent-specific behavior is necessary, document it clearly and isolate it behind conditionals or separate files.

## Repository Structure

```
.claude-plugin/marketplace.json  # Marketplace configuration (plugin registry)
plugins/                         # All plugins live here
  <plugin-name>/
    .claude-plugin/plugin.json   # Plugin manifest (name, version, paths)
    skills/                      # Skill definitions (SKILL.md files)
    agents/                      # Agent definitions (AGENT.md files)
    docs/                        # Strategy docs, design rationale
    README.md                    # Plugin documentation
```

## Current Plugins

| Plugin | Description |
|--------|-------------|
| `starchitect` | Product architecture toolkit. Pipeline: PRD generation (prd-create) → architectural floorplans (floorplan) → feature decomposition (prd-feature-breakdown) → entity/API/protocol contracts (contracts) → task decomposition (bv-taskify). Also includes technology research (tech-plan). Bridges the gap from product idea to implementation-ready task hierarchies. |

## Adding a New Plugin

1. Create a new directory under `plugins/<plugin-name>/`
2. Add `.claude-plugin/plugin.json` with the plugin manifest
3. Add `skills/` and/or `agents/` directories with SKILL.md/AGENT.md files
4. Add a README.md documenting the plugin
5. Register the plugin in `.claude-plugin/marketplace.json` under the `plugins` array

**Note:** Do not add commands — use skills instead. As of Claude Code 2.1.3+, commands have been merged into skills.

## Skill and Agent Frontmatter

Skills and agents should include frontmatter for proper registration and invocation control.

**Skill frontmatter (`skills/<name>/SKILL.md`):**

```yaml
---
name: skill-name
description: When the agent should use this skill
disable-model-invocation: false  # true = user-only via /command
user-invocable: true             # false = agent-only, hidden from /menu
---
```

**Agent frontmatter (`agents/<name>.md`):**

```yaml
---
name: agent-name
description: When the agent should use this subagent
tools: Read, Grep, Glob          # omit to inherit all tools
model: sonnet                    # sonnet, opus, haiku, or inherit (default)
permissionMode: default          # default, acceptEdits, dontAsk, bypassPermissions, plan
maxTurns: 5                      # max agentic turns before stopping
background: false                # true = runs concurrently
skills:                          # full skill content injected at startup
  - plugin-name:skill-name
color: green                     # UI color for identifying the agent
---
```

See the [subagents documentation](https://code.claude.com/docs/en/sub-agents#supported-frontmatter-fields) for the complete field reference (including `disallowedTools`, `mcpServers`, `hooks`, `memory`, `isolation`).

Always include a clear `description` that explains when the skill or agent should be used; the host agent uses the description to decide when to delegate.

See the [skills documentation](https://code.claude.com/docs/en/skills) for more details.

## Plugin Manifest Format

```json
{
  "name": "plugin-name",
  "version": "0.1.0",
  "description": "Description of the plugin",
  "author": { "name": "Author Name" },
  "skills": "./skills/",
  "agents": "./agents/"
}
```

## Agent-Agnostic Design Principles

Plugins should work across multiple coding agents. Follow these principles:

1. **Use standard tool names.** Reference tools like `Read`, `Write`, `Edit`, `Grep`, `Glob`, `Bash` — these map to equivalent capabilities in most agents.
2. **Don't assume system prompt structure.** Your skill/agent markdown is the entire context the subagent receives. Be self-contained.
3. **Prefer file-based I/O.** When passing data between steps, write to files rather than relying on agent-specific memory or context mechanisms.
4. **Document agent-specific features.** If a skill leverages a capability unique to one agent (e.g., Claude Code's `Task` tool for parallel subagents), note it in the skill's description and provide a fallback path.
5. **Keep skills declarative.** Describe *what* to do, not *how* a specific agent should do it. Let the agent map instructions to its own tool set.

## Releasing a New Version

When asked to "cut a release" or "release a new version":

1. **Determine bump type** from changes since last tag (`git log --oneline $(git describe --tags --abbrev=0 2>/dev/null || echo HEAD~10)..HEAD`):
   - `major` — breaking changes, major reorganization
   - `minor` — new skills, agents, or significant behavior changes
   - `patch` — bug fixes, doc updates, minor improvements

2. **Write the changelog entry** in `CHANGELOG.md` (repo root). Keep it scannable. Link each line to its PR with inline markdown links.

3. **Run the release script** (if available) — bumps version in both the plugin's `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`:
   ```bash
   ./scripts/release.sh <plugin-name> <major|minor|patch>
   ```

4. **Commit and open a PR:**
   ```bash
   git add -A && git commit -m "chore(release): <plugin-name> <version>"
   ```
   Push the branch and open a PR. The `<plugin-name>/v<version>` tag is created automatically when the PR merges.

## Commit and PR Conventions

Use [Conventional Commits](https://www.conventionalcommits.org/) for commit messages and PR titles:

- `feat:` — new skills, agents, or capabilities
- `fix:` — bug fixes, behavior corrections
- `chore:` — maintenance, config, releases (`chore(release): 1.3.5`)
- `docs:` — documentation-only changes
- `refactor:` — restructuring without behavior change

PR titles follow the same format. Keep them under 70 characters.

## Plugin Development Learnings

Patterns and pitfalls discovered while building skills, agents, and workflows.

### Plugin skill namespacing in agent frontmatter

When an agent's `skills:` field references a skill from the same plugin, use the full `plugin-name:skill-name` namespace. The bare skill name silently fails to resolve.

```yaml
# Wrong — skill won't load, no error
skills:
  - my-skill

# Correct
skills:
  - starchitect:my-skill
```

### Agent invocation control

Agents have no `disable-model-invocation` or `user-invocable` fields (those only exist for skills). The `description` is the only mechanism to prevent the host agent from auto-invoking a subagent. For internal agents that should only be spawned by a specific skill, lead with the constraint and explain why standalone invocation won't work:

```yaml
# Vague — agent may match this to general user requests
description: Researches technology options for architecture decisions.

# Clear — agent understands this can't be used standalone
description: >
  Internal implementation detail of the starchitect skill.
  Do not invoke directly — requires structured input
  that only the starchitect orchestrator provides.
```

### Skills as preloaded agent knowledge

Non-user-invocable skills can carry invariant rules (file formats, schemas, validation checklists) that get injected into a subagent's context at startup via the `skills:` frontmatter field. This separates what never changes (the craft) from what varies per invocation (the task).

Use `user-invocable: false` to hide the skill from the `/` menu. Keep `disable-model-invocation: false` (or omit it) so the agent preloading system can discover and inject the skill content.

```yaml
---
name: internal-knowledge
user-invocable: false
disable-model-invocation: false
---
```

**Warning:** Setting `disable-model-invocation: true` makes the skill invisible to the agent entirely — including agent `skills:` preloading. The skill silently fails to inject with no error. Only use `disable-model-invocation: true` for skills that are exclusively user-invoked via `/command`.

### Custom agents save context

Custom agents receive only their markdown body as the system prompt, not the full host agent system prompt. This saves significant context for single-turn agents where every token matters. Combine with `maxTurns: 1` to structurally prevent multi-turn exploration.

### Keep docs in sync with skill changes

Strategy docs (`plugins/<name>/docs/`) describe the rationale and mechanics behind each skill. When a skill's behavior changes, the corresponding strategy doc must be updated in the same PR. The README (`plugins/<name>/README.md`) must also reflect new skills, agents, or workflow changes.

Docs that commonly drift:

- **External CLI invocations** — when command syntax changes, update strategy docs
- **Input handling** — when the mechanism for passing data changes, update relevant sections
- **Agent/skill additions or renames** — update the README's skills and agents tables
- **Workflow changes** — update the README's workflow diagrams and stage boundaries

### Agent + skill + orchestrator abstraction

When a skill orchestrates parallel agents, split responsibilities cleanly:

- **Agent definition** (the `.md` file): who it is and how it behaves. Keep minimal.
- **Preloaded skill**: the craft. Invariant rules, file format, schema, validation checklist. Loaded once, shared across all invocations.
- **Orchestrator prompt**: what to build. Per-invocation specifics (the brief, data, parameters). Don't repeat invariant content here.

This keeps orchestrator prompts lean (the rules live in the skill) and agent definitions focused (the knowledge lives in the skill).

## Installation (for users)

```bash
# Install via curl
curl -fsSL "https://raw.githubusercontent.com/visigoth/marketplace/main/scripts/install.sh?$(date +%s)" | bash
```

Or install manually:

```
/plugin marketplace add visigoth/marketplace
/plugin install <plugin-name>@marketplace
```
