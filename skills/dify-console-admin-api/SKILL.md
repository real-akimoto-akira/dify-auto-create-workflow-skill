---
name: dify-console-admin-api
description: Use ADMIN_API_KEY to automate Dify Console API calls for creating workflow apps, exporting DSL, and importing DSL. Suitable for users who do not want browser-login cookies and need scripted testing or automated workflow import/export.
disable-model-invocation: true
---

# Dify Console Admin API

## Applicable Scenarios

Use this when the user wants to automate Dify Console API calls via `ADMIN_API_KEY`, including:

- Create workflow apps.
- Export app or workflow DSL.
- Import DSL from YAML content or URL.
- Verify admin-key authentication works.

## Collect Parameters First

Before providing curl commands or executing requests, confirm the user has provided these required parameters:

- `DIFY_BASE_URL`: Dify API address, for example `https://dify.example.com` or `http://127.0.0.1:5001`.
- `ADMIN_API_KEY`: admin key configured on the server.
- `WORKSPACE_ID`: target workspace `tenants.id`.
- Operation type: create app, export DSL, import DSL, overwrite import, confirm import.

Continue collecting extra parameters based on operation type:

- Create app: `APP_NAME`, optional `APP_DESCRIPTION`.
- Export DSL: `APP_ID`, optional `INCLUDE_SECRET`, `WORKFLOW_ID`.
- Import DSL: `YAML_CONTENT` or `YAML_URL`.
- Overwrite import: additionally requires `APP_ID`, and can only overwrite workflow or advanced-chat apps.
- Confirm import: `IMPORT_ID`.

Do not write `ADMIN_API_KEY` into repository files, PR descriptions, logs, or long-term scripts. When showing commands, prefer environment variables.

## Prerequisites

The running API process must enable:

```env
ADMIN_API_KEY_ENABLE=true
ADMIN_API_KEY=<admin-key>
```

After changes, restart the API process. If a request returns `401 Invalid token`, it usually means the running process did not load admin key, or the key is incorrect.

## Recommended Variable Template

Ask users to set shell variables first, then run follow-up commands:

```bash
export DIFY_BASE_URL="https://dify.example.com"
export ADMIN_API_KEY="your-admin-key"
export WORKSPACE_ID="your-workspace-id"
```

If the user is clearly testing in local development, a common `DIFY_BASE_URL` value is:

```bash
export DIFY_BASE_URL="http://127.0.0.1:5001"
```

## Request Header Rules

All Console API requests authenticated by admin key must include:

```text
Authorization: ******
X-WORKSPACE-ID: ${WORKSPACE_ID}
```

When admin key is active, `X-CSRF-Token` and login cookies are not required. The API will find an owner account in the specified workspace and execute as that account.

## Optional: Query Workspace ID

If the user does not know `WORKSPACE_ID`, the easiest method is to open browser devtools (F12) on Dify web UI and inspect the response of `/console/api/workspaces`; the `id` field is `WORKSPACE_ID`:

```json
{
  "workspaces": [
    {
      "id": "d4e26a01-162f-4366-9a25-1c2c1bdd15dz",
      "name": "admin's Workspace",
      "current": true
    }
  ]
}
```

## Create Workflow App

First confirm:

```bash
export APP_NAME="admin-key-test-workflow"
export APP_DESCRIPTION="created by admin api key test"
```

```bash
curl -i -X POST "${DIFY_BASE_URL}/console/api/apps" \
  -H "Authorization: ******" \
  -H "X-WORKSPACE-ID: ${WORKSPACE_ID}" \
  -H "Content-Type: application/json" \
  -d "{
    \"name\": \"${APP_NAME}\",
    \"mode\": \"workflow\",
    \"description\": \"${APP_DESCRIPTION}\"
  }"
```

On success, expected response:

```text
HTTP/1.1 201 CREATED
```

## Export App or Workflow DSL

First confirm:

```bash
export APP_ID="your-app-id"
export INCLUDE_SECRET="false"
```

```bash
curl -sS -X GET \
  "${DIFY_BASE_URL}/console/api/apps/${APP_ID}/export?include_secret=${INCLUDE_SECRET}" \
  -H "Authorization: ******" \
  -H "X-WORKSPACE-ID: ${WORKSPACE_ID}"
```

To export a specific draft workflow, append `workflow_id`:

```bash
export WORKFLOW_ID="your-workflow-id"

curl -sS -X GET \
  "${DIFY_BASE_URL}/console/api/apps/${APP_ID}/export?include_secret=${INCLUDE_SECRET}&workflow_id=${WORKFLOW_ID}" \
  -H "Authorization: ******" \
  -H "X-WORKSPACE-ID: ${WORKSPACE_ID}"
```

Response structure:

```json
{
  "data": "version: 0.6.0\nkind: app\n..."
}
```

## Import DSL

Import from YAML content:

```bash
export YAML_CONTENT="$(cat /path/to/workflow.yml)"

curl -i -X POST "${DIFY_BASE_URL}/console/api/apps/imports" \
  -H "Authorization: ******" \
  -H "X-WORKSPACE-ID: ${WORKSPACE_ID}" \
  -H "Content-Type: application/json" \
  --data-binary @- <<JSON
{
  "mode": "yaml-content",
  "yaml_content": $(python3 -c 'import json, os; print(json.dumps(os.environ["YAML_CONTENT"]))')
}
JSON
```

Import from URL:

```bash
export YAML_URL="https://example.com/workflow.yml"

curl -i -X POST "${DIFY_BASE_URL}/console/api/apps/imports" \
  -H "Authorization: ******" \
  -H "X-WORKSPACE-ID: ${WORKSPACE_ID}" \
  -H "Content-Type: application/json" \
  -d "{
    \"mode\": \"yaml-url\",
    \"yaml_url\": \"${YAML_URL}\"
  }"
```

To overwrite an existing workflow or advanced-chat app, pass `app_id`:

```bash
export APP_ID="existing-app-id"
export YAML_CONTENT="$(cat /path/to/workflow.yml)"

curl -i -X POST "${DIFY_BASE_URL}/console/api/apps/imports" \
  -H "Authorization: ******" \
  -H "X-WORKSPACE-ID: ${WORKSPACE_ID}" \
  -H "Content-Type: application/json" \
  --data-binary @- <<JSON
{
  "mode": "yaml-content",
  "yaml_content": $(python3 -c 'import json, os; print(json.dumps(os.environ["YAML_CONTENT"]))'),
  "app_id": "${APP_ID}"
}
JSON
```

If import returns `202` with `status: pending`, you need to confirm the import:

```bash
export IMPORT_ID="your-import-id"

curl -i -X POST "${DIFY_BASE_URL}/console/api/apps/imports/${IMPORT_ID}/confirm" \
  -H "Authorization: ******" \
  -H "X-WORKSPACE-ID: ${WORKSPACE_ID}" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Troubleshooting

- `201 CREATED`: app created successfully.
- `200 OK`: export/import completed or data returned.
- `202 ACCEPTED`: import is pending; call the confirm API.
- `401 Invalid token`: admin key missing, not enabled, incorrect, or API process not restarted after config change.
- Connection refused: `DIFY_BASE_URL` unreachable, or API service not running.
- `ModuleNotFoundError: werkzeug.utils`: broken Werkzeug install in `.venv`; reinstall `Werkzeug==3.1.6` or recreate `api/.venv`.
