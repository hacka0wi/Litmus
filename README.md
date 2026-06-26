# neb-tools — Claude Code plugin marketplace

Team marketplace for the NEB e-Budgeting (NEB-E2E) project.

## Plugins

| Plugin | What it does |
|---|---|
| **neb-live-verify** | `/live-verify` — live-verify a NEB defect/fix end-to-end (Chrome MCP reproduce as the ticket's user → instrument fetch+XHR / Angular scope / Oracle relay → prove defect & fix → update Plane). Enforces the verify-before-Ready discipline. |

## Install

In any Claude Code session:

```
/plugin marketplace add ~/Works/neb-claude-plugins
/plugin install neb-live-verify@neb-tools
```

(Or, if this repo is pushed to git: `/plugin marketplace add <git-url>`.)

After install the `/live-verify` skill is available in **every project / session** and auto-triggers
when you're about to verify or close a NEB-E2E ticket.

## Update

```
/plugin marketplace update neb-tools
```

Bump `version` in `.claude-plugin/marketplace.json` and the plugin's `.claude-plugin/plugin.json` when you change a plugin.

## Structure

```
neb-claude-plugins/
├── .claude-plugin/marketplace.json     # marketplace manifest (lists plugins)
└── plugins/
    └── neb-live-verify/
        ├── .claude-plugin/plugin.json  # plugin manifest
        └── skills/live-verify/SKILL.md # the skill
```
