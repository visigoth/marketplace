# Claude Plugin Marketplace

A collection of plugins for [Claude Code](https://claude.ai/claude-code) that extend its capabilities with specialized skills and agents.

## Available Plugins

### [starchitect](plugins/starchitect/README.md)

A workflow for transforming product ideas into implementation-ready task hierarchies. Guides you through PRD creation, architectural floorplans, technology choices, feature decomposition, contracts, task decomposition, and test planning — each step building on the last.

## Installation

### Quick Install

```bash
curl -fsSL "https://raw.githubusercontent.com/visigoth/marketplace/main/scripts/install.sh?$(date +%s)" | bash
```

### Manual Install

1. Add the marketplace:
   ```
   /plugin marketplace add visigoth/marketplace
   ```

2. Install a plugin:
   ```
   /plugin install starchitect@marketplace
   ```

## License

MIT
