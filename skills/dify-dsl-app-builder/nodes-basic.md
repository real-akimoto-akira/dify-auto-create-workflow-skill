# Basic Nodes: start / end / answer

## Start Node

`type: start`; declares input parameters in workflow mode. advanced-chat does not require `variables`.

```yaml
data:
  desc: ''
  title: Input Parameters
  type: start
  variables:
  - default: ''
    hint: ''
    label: User ID
    max_length: 256
    options: []
    required: true
    type: text-input      # text-input | paragraph | number | select | file | file-list
    variable: user_id
```

---

## End Node

`type: end`; used only for final output in workflow mode.

```yaml
data:
  desc: ''
  outputs:
  - value_selector:
    - code_node
    - content
    value_type: string    # string | number | object | array[string] | array[number] | array[object]
    variable: content
  title: Output
  type: end
```

---

## Answer Node

`type: answer`; used only for streaming output in advanced-chat mode. Supports variable template syntax `{{#node_id.field#}}`.

```yaml
data:
  answer: '{{#llm_node.text#}}'
  desc: ''
  title: Reply
  type: answer
  variables: []
```
