---
name: "step-functions-jsonata"
displayName: "AWS Step Functions with JSONata"
description: "Build AWS Step Functions state machines using JSONata query language. Covers ASL structure, all state types, variables, data transformation, error handling, and service integrations in JSONata mode."
keywords: ["step functions", "state machine", "serverless", "jsonata", "asl", "amazon states language", "workflow orchestration"]
author: "Jeff Palmer https://linkedin.com/in/jeffrey-palmer/"
---

# Step Functions JSONata

Build AWS Step Functions state machines using the JSONata query language instead of legacy JSONPath. JSONata simplifies data transformation, reduces boilerplate, and reduces external dependencies.

## Overview

AWS Step Functions uses Amazon States Language (ASL) to define state machines as JSON. With JSONata mode, you replace the five JSONPath I/O fields (InputPath, Parameters, ResultSelector, ResultPath, OutputPath) with just two fields: `Arguments` and `Output`. You also gain workflow variables via `Assign`, and powerful `Condition` expressions in Choice states.

This power provides comprehensive guidance for writing state machines in JSONata mode, covering:
- ASL structure and all eight state types in JSONata mode
- The `$states` reserved variable and JSONata expression syntax
- Workflow variables with `Assign` for cross-state data sharing
- Data transformation patterns with `Arguments` and `Output`
- Error handling with `Retry` and `Catch`
- Service integration patterns (Lambda, DynamoDB, SNS, SQS, etc.)

## When to Load Steering Files

Load the appropriate steering file based on what the user is working on:

- **ASL structure**, **state types**, **Task**, **Pass**, **Choice**, **Wait**, **Succeed**, **Fail**, **Parallel**, **Map** → see `asl-state-types.md`
- **Variables**, **Assign**, **data passing**, **scope**, **$states**, **input**, **output**, **Arguments**, **Output**, **data transformation** → see `variables-and-data.md`
- **Error handling**, **Retry**, **Catch**, **fallback**, **error codes**, **States.Timeout**, **States.ALL** → see `error-handling.md`
- **Service integrations**, **Lambda invoke**, **DynamoDB**, **SNS**, **SQS**, **SDK integrations**, **Resource ARN**, **sync**, **async** → see `service-integrations.md`

## Quick Reference

### Enabling JSONata

Set `QueryLanguage` at the top level to apply to all states:

```json
{
  "QueryLanguage": "JSONata",
  "StartAt": "MyState",
  "States": { ... }
}
```

### JSONata Expression Syntax

Wrap expressions in `{% %}`:

```json
"Output": "{% $states.input.customer.name %}"
"TimeoutSeconds": "{% $timeout %}"
"Condition": "{% $states.input.age >= 18 %}"
```

### The `$states` Reserved Variable

```
$states.input        → Original state input
$states.result       → Task/Parallel/Map result (on success)
$states.errorOutput  → Error output (only in Catch)
$states.context      → Execution context object
```

### Key Fields in JSONata Mode

| Field | Purpose | Available In |
|-------|---------|-------------|
| `Arguments` | Input to task/branches | Task, Parallel |
| `Output` | Transform state output | All except Fail |
| `Assign` | Store workflow variables | All except Succeed, Fail |
| `Condition` | Boolean branching | Choice rules |
| `Items` | Array for iteration | Map |

### JSONata Functions Provided by Step Functions

| Function | Purpose |
|----------|---------|
| `$partition(array, size)` | Partition array into chunks |
| `$range(start, end, step)` | Generate array of values |
| `$hash(data, algorithm)` | Calculate hash (MD5, SHA-1, SHA-256, SHA-384, SHA-512) |
| `$random([seed])` | Random number 0 ≤ n < 1, optional seed |
| `$uuid()` | Generate v4 UUID |
| `$parse(jsonString)` | Deserialize JSON string |

Plus all [built-in JSONata functions](https://github.com/jsonata-js/jsonata/tree/master/docs)

### Minimal Complete Example

```json
{
  "Comment": "Order processing workflow",
  "QueryLanguage": "JSONata",
  "StartAt": "ValidateOrder",
  "States": {
    "ValidateOrder": {
      "Type": "Task",
      "Resource": "arn:aws:states:::dynamodb:getItem",
      "Arguments": {
        "TableName": "OrdersTable",
        "Key": {
          "orderId": {
            "S": "{% $states.input.orderId %}"
          }
        }
      },
      "Assign": {
        "orderId": "{% $states.input.orderId %}"
      },
      "Output": "{% $states.result.Item %}",
      "Next": "CheckStock"
    },
    "CheckStock": {
      "Type": "Choice",
      "Choices": [
        {
          "Condition": "{% $states.input.inStock = true %}",
          "Next": "ProcessPayment"
        }
      ],
      "Default": "OutOfStock"
    },
    "ProcessPayment": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sqs:sendMessage",
      "Arguments": {
        "QueueUrl": "https://sqs.us-east-1.amazonaws.com/123456789012/PaymentQueue",
        "MessageBody": "{% $string({'orderId': $orderId, 'amount': $states.input.total.N}) %}"
      },
      "Output": {
        "orderId": "{% $orderId %}",
        "messageId": "{% $states.result.MessageId %}"
      },
      "Retry": [
        {
          "ErrorEquals": ["States.TaskFailed"],
          "IntervalSeconds": 2,
          "MaxAttempts": 3,
          "BackoffRate": 2.0
        }
      ],
      "End": true
    },
    "OutOfStock": {
      "Type": "Fail",
      "Error": "OutOfStockError",
      "Cause": "Requested item is out of stock"
    }
  }
}
```

## Best Practices

- Always set `"QueryLanguage": "JSONata"` at the top level for new state machines
- Use `Assign` to store data needed in later states instead of threading it through Output
- Keep `Output` minimal — only provide what the next state actually needs
- Use `$states.input` to reference original state input, not `$` (which is restricted at top level in JSONata)
- Remember: `Assign` and `Output` are evaluated in parallel — variable assignments in `Assign` are NOT available in `Output` of the same state
- All JSONata expressions must produce a defined value — `$data.nonExistentField` throws `States.QueryEvaluationError`
- Use `$states.context.Execution.Input` to access the original workflow input from any state
- Save state machine definitions with `.asl.json` extension when working outside the console
- Prefer the optimized Lambda integration (`arn:aws:states:::lambda:invoke`) over the SDK integration

## Troubleshooting

### Common Errors

- `States.QueryEvaluationError` — JSONata expression failed. Check for type errors, undefined fields, or out-of-range values.
- Mixing JSONPath fields (`Parameters`, `InputPath`, `ResultPath`, etc.) with JSONata `QueryLanguage` — these are mutually exclusive.
- Using `$` or `$$` at the top level of a JSONata expression — use `$states.input` instead.
- Forgetting `{% %}` delimiters around JSONata expressions — the string will be treated as a literal.
- Assigning variables in `Assign` and expecting them in `Output` of the same state — new values only take effect in the next state.

## Resources

- [ASL Specification](https://states-language.net/spec.html)
- [Transforming data with JSONata in Step Functions](https://docs.aws.amazon.com/step-functions/latest/dg/transforming-data.html)
- [Passing data between states with variables](https://docs.aws.amazon.com/step-functions/latest/dg/workflow-variables.html)
- [JSONata documentation](https://docs.jsonata.org/overview.html)
- [Step Functions Developer Guide](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html)
