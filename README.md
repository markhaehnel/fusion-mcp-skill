# Fusion MCP Skill

[![skills.sh](https://skills.sh/b/markhaehnel/fusion-mcp-skill)](https://skills.sh/markhaehnel/fusion-mcp-skill)

A vendor-neutral [Agent Skill](https://agentskills.io/) for working safely in Autodesk Fusion through its MCP server. It guides an AI agent through document selection, mechanical modeling, Fusion API discovery, viewport verification, undo/redo, and read-only Electronics queries while preserving the user's design intent and save state.

## What it provides

- Safe preflight checks for the active document, modified state, product type, and interactive commands.
- Parametric mechanical-modeling guidance for components, sketches, parameters, features, assemblies, and units.
- Live Fusion API documentation discovery before uncertain or consequential API use.
- Verification through read-only model inspection and, when useful, viewport screenshots.
- Read-only Fusion Electronics queries with schema discovery and complete pagination.
- Vendor-neutral MCP capability matching, plus optional OpenAI metadata.

## Requirements

- Autodesk Fusion running on the same machine as the MCP server.
- The [Autodesk Fusion MCP Server enabled and connected](https://help.autodesk.com/view/ADSKMCP/ENU/?guid=ADSKMCP_FusionDesktopMcp_connecting_to_the_fusion_mcp_server_html).
- An Agent Skills-compatible AI client that can access the server's Fusion read, execute, update, and Electronics capabilities as needed.

This repository contains the agent skill only. It does not install Autodesk Fusion or configure the MCP connection.

## Install

List the skills available from this repository:

```bash
npx skills add markhaehnel/fusion-mcp-skill --list
```

Install `fusion-mcp` into the current project:

```bash
npx skills add markhaehnel/fusion-mcp-skill --skill fusion-mcp
```

Install it for the current user instead:

```bash
npx skills add markhaehnel/fusion-mcp-skill --skill fusion-mcp --global
```

The repository must be public on GitHub before other users can install it with these commands. Explicit skill-invocation syntax varies by client; a portable request is:

> Use the fusion-mcp skill to add a parametrically controlled 12 mm mounting hole to the active Fusion design and verify it without saving.

## Safety

The Fusion execution capability can run Python against a live design and can therefore make consequential changes. Review the skill before installation and keep important designs backed up or versioned. The workflow checks the active document, avoids conflicting interactive commands, verifies mutations, and leaves documents unsaved unless the user explicitly requests a save.

The skill does not bundle executable scripts, credentials, telemetry, or network clients.

## Repository layout

```text
skills/fusion-mcp/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── electronics.md
    └── mechanical-modeling.md
```

`SKILL.md` and `references/` are portable across compatible agents. `agents/openai.yaml` is an optional adapter and is not required by the open Agent Skills format.

## Development

Edit files under `skills/fusion-mcp/`. Keep the entrypoint concise and put domain-specific details in the routed reference files. Check local discovery before publishing:

```bash
npx skills add . --list
```

skills.sh does not require a separate package manifest or publish command. Push the repository publicly, then install it through the `skills` CLI; eligible installs allow the skill to appear on skills.sh through install telemetry.

## License

[MIT](LICENSE)

This project is not affiliated with or endorsed by Autodesk. Autodesk and Fusion are trademarks of Autodesk, Inc.
