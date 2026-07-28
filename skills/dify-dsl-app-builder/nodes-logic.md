# Logic Control Nodes: if-else / variable-aggregator / iteration

## IF/ELSE Node

`type: if-else`; conditional branching with `true` and `false` outputs (`sourceHandle`).

```yaml
data:
  cases:
  - case_id: 'true'
    conditions:
    - comparison_operator: contains   # = | != | > | < | contains | not contains | starts with | ends with | empty | not empty
      id: <uuid>
      value: hello
      varType: string
      variable_selector:
      - start
      - query
    id: 'true'
    logical_operator: and
  desc: ''
  title: Condition Check
  type: if-else
```

Edge output example:

```yaml
- source: 'if_node'
  sourceHandle: 'true'
  target: 'end_true'
- source: 'if_node'
  sourceHandle: 'false'
  target: 'end_false'
```

---

## Variable Aggregator Node

`type: variable-aggregator`; aggregates variables from multiple branches into a unified output.

```yaml
data:
  desc: ''
  output_type: string
  title: Variable Aggregation
  type: variable-aggregator
  variables:
  - - branch_true_node
    - output
  - - branch_false_node
    - output
```

Output variable: `output` (`value_selector: [aggregator_node, output]`).

If grouped aggregation is needed, add `advanced_settings`:

```yaml
  advanced_settings:
    group_enabled: true
    groups:
    - groupId: <uuid>
      group_name: Group1
      output_type: string
      variables:
      - - branch_a
        - output
      - - branch_b
        - output
```

---

## Iteration Node

`type: iteration`; iterates over an array item by item, containing an internal `iteration-start` sub-node.

**Container node:**

```yaml
data:
  error_handle_mode: terminated   # terminated | continue | remove-abnormal-output
  is_parallel: false
  iterator_input_type: array[string]
  iterator_selector:
  - code_node
  - result
  output_selector:
  - inner_template_node
  - output
  output_type: array[string]
  parallel_nums: 10
  start_node_id: iter_node_start
  title: Iteration
  type: iteration
height: 178
id: iter_node
width: 388
zIndex: 1
```

**Internal iteration-start sub-node** (fixed on the left side of container, not draggable):

```yaml
data:
  title: ''
  type: iteration-start
draggable: false
height: 48
id: iter_node_start
parentId: iter_node
position:
  x: 24
  y: 68
positionAbsolute:
  x: 708   # iter_node.position.x + 24
  y: 350   # iter_node.position.y + 68
selectable: false
sourcePosition: right
targetPosition: left
type: custom-iteration-start
width: 44
zIndex: 1002
```

**Internal edge rules:**

```yaml
data:
  isInIteration: true
  isInLoop: false
  iteration_id: iter_node
  sourceType: iteration-start
  targetType: template-transform
zIndex: 1002
```

**Internal node rules**: add `parentId: iter_node` and `zIndex: 1002`.

Reference the current iterated item via `value_selector: [iter_node, item]`; its type matches the element type of `iterator_input_type`.
