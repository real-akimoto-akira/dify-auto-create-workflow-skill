# Processing Nodes: code / template-transform / parameter-extractor

## Code Node

`type: code`; executes Python3 or JavaScript, declaring `variables` and `outputs`.

```yaml
data:
  code: "import json\n\ndef main(user_id: str, raw: str) -> dict:\n    return {\"content\": f\"id:{user_id}\"}\n"
  code_language: python3    # python3 | javascript
  desc: ''
  outputs:
    content:
      children: null
      type: string          # string | number | object | array[string] | array[number] | array[object]
  title: Code Processing
  type: code
  variables:
  - value_selector:
    - start
    - user_id
    value_type: string
    variable: user_id
  - value_selector:
    - http_node
    - body
    value_type: string
    variable: raw
```

Output variables are referenced via `[code_node, <output_key>]`.

---

## Template Transform Node

`type: template-transform`; Jinja2 template rendering, suitable for simple string composition without writing Python.

```yaml
data:
  desc: ''
  template: 'User: {{ user_id }}, Result: {{ result }}'
  title: Template Rendering
  type: template-transform
  variables:
  - value_selector:
    - start
    - user_id
    value_type: string
    variable: user_id
  - value_selector:
    - code_node
    - content
    value_type: string
    variable: result
```

Output variable: `output` (`value_selector: [template_node, output]`).

---

## Parameter Extractor Node

`type: parameter-extractor`; uses LLM to extract structured parameters from natural-language text. Output fields are defined by `parameters`.

```yaml
data:
  description: Extract parameters from user input
  model:
    mode: chat
    name: gpt-4o
    provider: langgenius/openai/openai
  parameters:
  - description: City name
    name: city
    required: true
    type: string    # string | number | bool | array[string] | array[number] | array[object] | object
  - description: Date range
    name: date_range
    required: false
    type: string
  query:
  - start
  - query
  reasoning_mode: function_call   # function_call | prompt
  title: Parameter Extraction
  type: parameter-extractor
```

Output variables are referenced via `[extractor_node, <parameter_name>]`, for example `[extractor_node, city]`.
