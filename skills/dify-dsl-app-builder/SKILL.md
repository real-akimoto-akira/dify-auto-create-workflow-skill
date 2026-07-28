---
name: dify-dsl-app-builder
description: Design and generate Dify app DSL based on user requirements, and create or update workflow/advanced-chat apps. Use when the user asks to build a Dify workflow from requirements, generate DSL, create an app from DSL, update an existing app DSL, or automate Dify app creation/update through Admin API.
---

# Dify DSL App Builder

## Applicable Scenarios

Use this skill when the user wants to "automatically generate Dify DSL from requirements and create or update apps", including:

- Generate workflow DSL from natural-language requirements.
- Modify existing DSL and update an app.
- Create a new workflow app and import DSL.
- Overwrite-import an existing workflow or advanced-chat app.
- Re-export after import and verify remote DSL.

When requests involve the Console Admin API, you must first read and follow `skills/dify-console-admin-api/SKILL.md`.

## Basic Principles

- First provide a DSL design proposal and ask for user confirmation. Only modify files or call APIs after the user agrees.
- Prefer editing an existing DSL file. Create a new local DSL file only if none exists.
- Do not write `ADMIN_API_KEY` into repository files, PR descriptions, long-lived scripts, or final responses.
- Default to `include_secret=false` unless the user explicitly asks to export secrets.
- Before overwrite updates, export the old DSL as a backup to avoid losing existing configuration.
- If Admin API calls hang or fail, report errors and troubleshooting suggestions first. Only write directly to the database when explicitly authorized by the user.

## Parameter Collection

Confirm these parameters before execution:

- `DIFY_BASE_URL`: for example `http://127.0.0.1:5001`.
- `ADMIN_API_KEY`: admin key configured on the server.
- `WORKSPACE_ID`: target workspace `tenants.id`.
- Operation type: create app, update app, or generate DSL only.

Then continue by operation type:

- Create app: `APP_NAME`, optional `APP_DESCRIPTION`.
- Update app: `APP_ID`, optional existing DSL file path.
- DSL only: target file path or app name.

## DSL Design Process

Form a proposal based on user requirements first:

1. Clarify app mode; use `workflow` by default.
2. Clarify input parameters, including variable name, type, required flag, and default value.
3. Clarify node chain, such as `start -> http-request -> code -> end`.
4. Clarify external dependencies, such as HTTP APIs, plugins, knowledge bases, and models.
5. Clarify output fields and output format.
6. Clarify error-handling strategy, such as returning "unknown" or an error message when external APIs fail.

Generate DSL only after proposal confirmation. Node structures should reference exported DSL or test fixtures in this repository, and keep field names and node types aligned with the current Dify version.

## App Creation Flow

1. Generate a local DSL file.
2. Use the create-app API from `skills/dify-console-admin-api/SKILL.md` to create a workflow app.
3. Use the returned `APP_ID` to overwrite-import DSL.
4. If import returns `202 pending`, call the confirm API.
5. Re-export the app DSL and verify that remote content includes key nodes, variables, and outputs.

## App Update Flow

1. Export current DSL with `APP_ID` and save it as a backup file.
2. Modify or generate a new local DSL.
3. Use `apps/imports` with `app_id` for overwrite import.
4. If import returns `202 pending`, call the confirm API.
5. Re-export the app DSL and verify that remote content includes key nodes, variables, and outputs.

## Validation Requirements

After updates, verify at least:

- Remote DSL export succeeds.
- Remote DSL includes expected input variables.
- Remote DSL includes expected node types and key configuration.
- Remote DSL includes expected output fields or output text.

In the final response, only provide result, file path, app ID, and validation conclusion. Do not reveal `ADMIN_API_KEY`.

## Edge Structure Notes

Each edge must include:

```yaml
- data:
    isInIteration: false
    isInLoop: false
    sourceType: <source node type>
    targetType: <target node type>
  id: <source>-source-<target>-target
  source: '<source_node_id>'
  sourceHandle: source        # if-else branches use 'true'/'false'; iteration output uses source
  target: '<target_node_id>'
  targetHandle: target
  type: custom
  zIndex: 0
```

For edges inside an iteration container, also add: `isInIteration: true`, `iteration_id`.

## Node Positioning Recommendations

Arrange nodes from left to right at x=80,384,688,992... (spacing 304), with unified y=282. Branch nodes should be offset vertically by about ±130.

Common outer fields for each node: `id`, `height` (54/90/110/126/178), `position`/`positionAbsolute` (x/y), `selected: false`, `sourcePosition: right`, `targetPosition: left`, `type: custom`, `width: 242`.

## Node Reference

When you need a node category, read the corresponding file for complete templates:

| File | Included Nodes |
|---|---|
| [nodes-basic.md](nodes-basic.md) | `start`, `end`, `answer` |
| [nodes-logic.md](nodes-logic.md) | `if-else`, `variable-aggregator`, `iteration` |
| [nodes-processing.md](nodes-processing.md) | `code`, `template-transform`, `parameter-extractor` |
| [nodes-ai.md](nodes-ai.md) | `llm`, `agent` (Function Calling / MCP SSE), expose Dify tools as MCP services (`mcp_compat_dify_tools`) |
| [nodes-external.md](nodes-external.md) | `http-request`, `tool`, `knowledge-retrieval` |
