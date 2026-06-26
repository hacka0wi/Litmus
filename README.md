# Saga — Claude Code plugin marketplace

Team marketplace for the NEB e-Budgeting (NEB-E2E) project. Named after **Saga**, the agent persona
that signs the Plane comments.

## Plugins

| Plugin | What it does |
|---|---|
| **litmus** | `/litmus` — the litmus test for a fix: live-verify a NEB defect/fix end-to-end (Chrome MCP reproduce as the ticket's user → instrument fetch+XHR / Angular scope / Oracle relay → prove defect & fix → update Plane). Enforces the verify-before-Ready discipline. |

## Install

In any Claude Code session:

```
/plugin marketplace add ~/Works/saga
/plugin install litmus@saga
```

(Or, if this repo is pushed to git: `/plugin marketplace add <git-url>`.)

After install, the `/litmus` skill is available in **every project / session** and auto-triggers
when you're about to verify or close a NEB-E2E ticket.

## Update

```
/plugin marketplace update saga
```

Bump `version` in `.claude-plugin/marketplace.json` and the plugin's `.claude-plugin/plugin.json` when you change a plugin.

## Structure

```
saga/
├── .claude-plugin/marketplace.json   # marketplace manifest (name: saga)
└── plugins/
    └── litmus/
        ├── .claude-plugin/plugin.json  # plugin manifest (name: litmus)
        └── skills/litmus/SKILL.md      # the /litmus skill
```
