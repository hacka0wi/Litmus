# Saga — Claude Code plugin marketplace

A small marketplace of Claude Code tools for QA / live verification workflows.

## Plugins

| Plugin | What it does |
|---|---|
| **litmus** | `/litmus` — the litmus test for a fix: live-verify a defect/fix **end-to-end in the real app** (browser-MCP reproduce as the reporting user → instrument fetch+XHR / front-end scope / DB relay → prove defect **and** fix → update the tracker). Enforces "verify before you call it done" — backend curl / unit / "it deployed" do **not** count. |

## Install

In any Claude Code session:

```
/plugin marketplace add https://github.com/hacka0wi/Litmus
/plugin install litmus@saga
```

After install, the `/litmus` skill is available in **every project / session** and auto-triggers when
you're about to verify or close a ticket.

## Update

```
/plugin marketplace update saga
```

Bump `version` in `.claude-plugin/marketplace.json` and the plugin's `.claude-plugin/plugin.json` on changes.

## Structure

```
.
├── .claude-plugin/marketplace.json   # marketplace manifest (name: saga)
└── plugins/
    └── litmus/
        ├── .claude-plugin/plugin.json  # plugin manifest (name: litmus)
        └── skills/litmus/SKILL.md      # the /litmus skill (generic, no secrets)
```

> Environment-specific details (hosts, credentials, tracker ids, DB relays) are intentionally **not** in
> this public skill — keep them in your own private config / agent memory.
