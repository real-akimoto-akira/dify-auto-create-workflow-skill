# AI Nodes: llm / agent

## LLM Node

`type: llm`; calls a large language model to generate text.

```yaml
data:
  context:
    enabled: false
    variable_selector: []
  desc: ''
  memory:
    enabled: false
    window:
      enabled: false
      size: 50
  model:
    mode: chat
    name: gpt-4o
    provider: langgenius/openai/openai
  prompt_template:
  - role: system
    text: You are an assistant.
  - role: user
    text: '{{#start.query#}}'
  retry_config:
    enabled: false
    max_retries: 1
    retry_interval: 1000
    exponential_backoff:
      enabled: false
      multiplier: 2
      max_interval: 10000
  structured_output:
    enabled: false
  title: LLM
  type: llm
  vision:
    enabled: false
    configs:
      variable_selector: []
```

Output variable: `text` (`value_selector: [llm_node, text]`); template reference: `{{#llm_node.text#}}`.

When `memory.enabled: true`, the node reads conversation history (advanced-chat mode only).

---

## Agent Node

`type: agent`; invokes an Agent strategy provided by plugins to execute multi-turn tool-use tasks.

`agent_strategy_provider_name` format is `<author>/<plugin>/<provider>`, and `agent_strategy_name` is the strategy name.

### Function Calling Strategy (Built-in)

```yaml
data:
  agent_parameters:
    model:
      type: constant
      value:
        completion_params: {}
        mode: chat
        model: gpt-4o            # keep consistent with name
        model_type: llm
        name: gpt-4o
        provider: langgenius/openai/openai
        type: model-selector
    tools:
      type: constant
      value:
        - enabled: true          # enabled: true is required
          provider_id: <provider_id>
          provider_name: <provider_name>
          provider_type: builtin
          tool_label: <label>
          tool_name: <tool_name>
          tool_parameters: {}
    query:
      type: constant
      value: '{{#start.query#}}'  # use template syntax in agent node; do not use type: variable
    max_iterations:
      type: constant
      value: 5
    instruction:
      type: constant
      value: 'You are an assistant.'
  agent_strategy_label: Function Calling
  agent_strategy_name: function_calling
  agent_strategy_provider_name: langgenius/agent/agent   # note: not agent_strategies
  output_schema: {}
  title: Agent
  tool_node_version: '2'
  type: agent
```

**Notes:**
- `model.value` must contain `completion_params`, `model` (same as `name`), `model_type: llm`, and `type: model-selector`; otherwise UI shows "Model not selected"
- `query` must use `type: constant` + template syntax `{{#node_id.field#}}`; do not use the `type: variable` selector list
- Every tool must include `enabled: true`
- `agent_strategy_provider_name` must be `langgenius/agent/agent` (official Dify Agent strategy plugin)
- Output variable: `text` (`value_selector: [agent_node, text]`); template reference: `{{#agent_node.text#}}`

### MCP SSE Agent Strategy (requires `junjiem/mcp_see_agent` plugin)

Supports external tool services through the MCP protocol.

```yaml
data:
  agent_parameters:
    model:
      type: constant
      value:
        completion_params: {}
        mode: chat
        model: gpt-4o
        model_type: llm
        name: gpt-4o
        provider: langgenius/openai/openai
        type: model-selector
    tools:
      type: constant
      value:
        - enabled: true
          provider_id: <provider_id>
          provider_name: <provider_name>
          provider_type: builtin
          tool_label: <label>
          tool_name: <tool_name>
          tool_parameters: {}
    query:
      type: constant
      value: '{{#start.query#}}'
    max_iterations:
      type: constant
      value: 5
    instruction:
      type: constant
      value: 'You are an assistant.'
    mcp_servers:
      type: constant
      value: |
        {
          "mcpServers": {
            "my-server": {
              "transport": "sse",
              "url": "http://your-mcp-server/sse"
            }
          }
        }
  agent_strategy_label: Function Calling (MCP)
  agent_strategy_name: function_calling
  agent_strategy_provider_name: junjiem/mcp_see_agent/mcp_see_agent
  output_schema: {}
  title: MCP Agent
  tool_node_version: '2'
  type: agent
```

`mcp_servers` supports two transport modes:
- SSE: `"transport": "sse"`, `"url": "http://host/sse"`
- Streamable HTTP: `"transport": "streamable_http"`, `"url": "http://host/mcp"`

You can configure multiple MCP services in `mcpServers`.

Agent node output variable: `text` (`value_selector: [agent_node, text]`); template reference: `{{#agent_node.text#}}`. If `output_schema` defines concrete fields, reference by field name.

---

## Expose Dify Tools as MCP Services

The [junjiem/mcp_compat_dify_tools](https://marketplace.dify.ai/plugin/junjiem/mcp_compat_dify_tools) plugin (requires Dify 1.2.0+) can convert Dify platform tool APIs into MCP-compatible interfaces. After installation, add an Endpoint in the plugin page and select tools to expose, then you can get the MCP service URL.

After changing the tool list of an existing Endpoint, you must disable and then re-enable it for changes to take effect.

After obtaining the URL, fill it into the MCP SSE Agent `mcp_servers` configuration:

**Streamable HTTP transport (recommended):**

```json
{
  "mcpServers": {
    "dify-tools": {
      "transport": "streamable_http",
      "url": "https://your-dify.com/e/<endpoint-id>/mcp"
    }
  }
}
```

**SSE transport (legacy):**

```json
{
  "mcpServers": {
    "dify-tools": {
      "transport": "sse",
      "url": "https://your-dify.com/e/<endpoint-id>/sse"
    }
  }
}
```

This allows MCP Agent to call tools configured on the Dify platform as MCP tools, enabling closed-loop integration between Dify tools and the Agent node.
