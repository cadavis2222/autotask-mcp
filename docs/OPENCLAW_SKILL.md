# OpenClaw Skill Conversion

This repository now includes an OpenClaw skill at:
- skills/autotask-mcp/SKILL.md

## What was converted

The MCP server tool surface was converted into an AgentSkills-compatible OpenClaw skill that:
- Describes when and how to use Autotask MCP tools.
- Adds OpenClaw metadata for runtime eligibility checks.
- Provides a full tool inventory in skills/autotask-mcp/TOOL_CATALOG.md.

## Install options

OpenClaw auto-loads workspace skills from:
- <workspace>/skills

Because this repo now has skills/autotask-mcp/SKILL.md, OpenClaw can detect it when this folder is used as an OpenClaw workspace.

## Verify in OpenClaw

1. Start a new OpenClaw session.
2. Run openclaw skills list.
3. Confirm autotask-mcp appears.
4. Test with a prompt such as:
- "Check Autotask connection and find open tickets for company Acme"

## Required environment

Set these environment variables where OpenClaw runs:
- AUTOTASK_USERNAME
- AUTOTASK_SECRET
- AUTOTASK_INTEGRATION_CODE

Optional:
- AUTOTASK_API_URL

## Notes

- The skill itself does not create tools; it teaches the agent how to use tools exposed by this MCP server.
- The MCP server must still be configured in OpenClaw or the underlying agent runtime.
