# External Integration Nodes: http-request / tool / knowledge-retrieval

## HTTP Request Node

`type: http-request`; calls external HTTP APIs. Variables can be referenced in both URL and body with `{{#node.field#}}`.

```yaml
data:
  authorization:
    type: no-auth         # no-auth | api-key | bearer | basic
  body:
    data: ''
    type: none            # none | json | form-data | x-www-form-urlencoded | raw-text
  desc: ''
  headers: ''
  method: GET             # GET | POST | PUT | DELETE | PATCH
  params: ''
  retry_config:
    enabled: false
    max_retries: 1
    retry_interval: 1000
    exponential_backoff:
      enabled: false
      multiplier: 2
      max_interval: 10000
  timeout:
    connect: 10
    read: 30
    write: 30
  title: Call API
  type: http-request
  url: 'http://example.com/api/{{#start.user_id#}}'
```

Output variables: `body` (string), `status_code` (number), `headers` (object).

JSON body example:

```yaml
  body:
    type: json
    data: '{"key": "{{#start.value#}}"}'
```

****** example:

```yaml
  authorization:
    type: bearer
    config:
      token: '{{#env.API_TOKEN#}}'
```

---

## Tool Node

`type: tool`; invokes built-in tools or plugin tools. `provider_type: builtin` means built-in tool.

```yaml
data:
  desc: ''
  provider_id: time
  provider_name: time
  provider_type: builtin
  title: Get Current Time
  tool_configurations:
    format:
      type: constant
      value: '%Y-%m-%d %H:%M:%S'
    timezone:
      type: constant
      value: Asia/Shanghai
  tool_description: A tool for getting the current time.
  tool_label: Current Time
  tool_name: current_time
  tool_node_version: '2'
  tool_parameters: {}
  type: tool
```

For plugin tools, use `provider_type: api`, and `provider_id` should be the plugin ID. Each parameter value in `tool_configurations` can be:
- `type: constant`: fixed value
- `type: variable`: reference workflow variable (configure selector in `tool_parameters`)
- `type: mixed`: string containing template variables

---

## Knowledge Retrieval Node

`type: knowledge-retrieval`; retrieves relevant document chunks from a knowledge base. `dataset_ids` are encrypted in exported DSL and automatically decrypted on import.

```yaml
data:
  dataset_ids:
  - <dataset_id>
  multiple_retrieval_config:
    reranking_enable: false
    reranking_mode: weighted_score
    score_threshold: null
    top_k: 4
    weights:
      keyword_setting:
        keyword_weight: 0
      vector_setting:
        embedding_model_name: text-embedding-3-large
        embedding_provider_name: langgenius/openai/openai
        vector_weight: 1
      weight_type: customized
  query_variable_selector:
  - start
  - query
  retrieval_mode: multiple    # multiple | single
  title: Knowledge Retrieval
  type: knowledge-retrieval
```

Output variable: `result` (array[object], `value_selector: [retrieval_node, result]`).
