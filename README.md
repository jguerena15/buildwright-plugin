# Buildwright Plugin

A Claude Code plugin for building Next.js + Tailwind CSS applications backed by Figma designs, with iterative UI development powered by Playwright.

## Features

### Skills & Commands

| Command | Description |
|---|---|
| `/buildwright-plugin:build-ui` | Build UI components and pages from Figma designs. Pulls design specs via Figma MCP, generates components following your project's design system, and verifies output with Playwright. |
| `/buildwright-plugin:prd` | Generate a structured Product Requirements Document. Asks clarifying questions, then outputs a detailed PRD to `tasks/prd-[feature-name].md`. |
| `/buildwright-plugin:ralph` | Convert a PRD into Ralph's `prd.json` format for autonomous execution. |

All three are also available as **agent skills** — Claude will automatically invoke them when the task context matches (e.g., "build this page from Figma" or "plan this feature").

### Ralph Loop (Autonomous Execution)

Ralph is an autonomous agent loop that works through a PRD's user stories one at a time. Each iteration spawns a fresh Claude instance, implements one story, runs quality checks, commits, and moves on.

**Workflow:**

1. `/buildwright-plugin:prd` — plan a feature → `tasks/prd-feature-name.md`
2. `/buildwright-plugin:ralph` — convert to JSON → `scripts/ralph/prd.json`
3. Run the loop:
   ```bash
   ./scripts/ralph/ralph.sh        # default: 10 iterations
   ./scripts/ralph/ralph.sh 20     # or specify max iterations
   ```

See `scripts/ralph/prd.json.example` for the expected JSON format.

### Quality Check Hook

Runs automatically after every file Write/Edit. Checks TypeScript compilation, ESLint, and Prettier against the user's project — with autofix where possible.

Configure behavior via `hooks/hook-config.json`.

### TypeScript LSP

Bundles a TypeScript Language Server configuration so Claude gets real-time diagnostics, go-to-definition, and type information out of the box.

## Prerequisites

- [Claude Code](https://claude.com/claude-code) v1.0.33+
- Node.js and npm/pnpm/bun
- `typescript` and `typescript-language-server` installed globally:
  ```bash
  npm install -g typescript-language-server typescript
  ```
- `jq` (for Ralph loop) — `brew install jq` on macOS

### Optional

- **Figma MCP** — required for `/buildwright-plugin:build-ui` to pull designs directly from Figma. [Setup guide](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server). If not configured, the skill will prompt you with setup instructions and offer to work from screenshots instead.
- **Playwright MCP** — used for visual verification of UI builds across breakpoints.

## Installation

### From the marketplace

```bash
/plugin marketplace add jguerena15/buildwright-plugin
/plugin install buildwright-plugin@buildwright
```

### Local development

```bash
claude --plugin-dir /path/to/buildwright-plugin
```

## Plugin Structure

```
buildwright-plugin/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── commands/
│   ├── build-ui.md              # /buildwright-plugin:build-ui
│   ├── prd.md                   # /buildwright-plugin:prd
│   └── ralph.md                 # /buildwright-plugin:ralph
├── skills/
│   ├── build-ui/SKILL.md        # Figma → code agent skill
│   ├── prd/SKILL.md             # PRD generator agent skill
│   └── ralph/SKILL.md           # PRD → JSON converter agent skill
├── scripts/
│   └── ralph/
│       ├── ralph.sh             # Autonomous loop script
│       ├── CLAUDE.md            # Prompt for each Ralph iteration
│       └── prd.json.example     # Reference format
├── hooks/
│   ├── hooks.json               # PostToolUse hook config
│   ├── quality-check.js         # TS/ESLint/Prettier checker
│   └── hook-config.json         # Quality check settings
├── .lsp.json                    # TypeScript LSP config
├── .mcp.json                    # MCP server config
└── settings.json                # Plugin settings
```

## License

MIT
