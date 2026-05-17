# Step Functions

**TL;DR** — Visual workflows / state machines. Orchestrate Lambdas, ECS tasks, Glue jobs, etc. Built-in retries, parallel branches, error handling. Two types: Standard and Express.

## What it is

A state machine engine. You define a workflow in **ASL (Amazon States Language, JSON)**; Step Functions runs it, calling AWS services at each step, handling failures, branches, parallelism. Audit-friendly — every state transition is recorded.

## Why it exists

When you have multi-step workflows (place order → charge card → reserve inventory → notify warehouse → email user), you could chain Lambdas with SQS — but error handling, retries, parallel branches, observability become a nightmare. Step Functions handles them declaratively.

## Two flavors

### Standard
- Long-running (up to 1 year), exactly-once execution.
- $0.025 per 1,000 state transitions.
- Audit history, visual exec view.
- Use for: business workflows, long batch jobs.

### Express
- High-throughput (100k/sec+), short (5 min max), at-least-once.
- Pay per duration + memory.
- No visual history per execution (CloudWatch Logs).
- Use for: API back-of, event-driven micro-flows.

## Key concepts

- **State machine** — the workflow definition.
- **State** — a node in the graph. Types: Task, Choice, Wait, Parallel, Map, Pass, Succeed, Fail.
- **Execution** — one run of the state machine.
- **Task state** — calls an AWS service (Lambda, ECS, SNS, DynamoDB, SQS, Bedrock, etc.) via the **AWS SDK integration** or **Optimized integration**.
- **Workflow Studio** — visual designer in the console.
- **Map state** — iterate over an array (sequential or distributed for big lists).
- **Callback / Wait for Task Token** — pause until external event resumes the workflow.
- **Retry / Catch** — per-state error handling.

## Real-world example

> ShareDeal `OrderFulfillment` state machine:
>
> ```
> Start → ChargePayment (Lambda)
>   ↓
> ReserveInventory (DynamoDB)
>   ↓
> Parallel:
>   ├── SendEmail (SNS)
>   ├── NotifyWarehouse (SQS)
>   └── UpdateAnalytics (Firehose)
>   ↓
> Choice: PaymentDeclined?
>   yes → Refund → End (Fail)
>   no  → End (Success)
> ```
>
> Retries on transient failures, full execution history in console, alerts on failures via EventBridge.

## Usage

### Define a state machine (ASL JSON)

```json
{
  "Comment": "Order fulfillment",
  "StartAt": "ChargePayment",
  "States": {
    "ChargePayment": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:..:function:charge",
      "Retry": [{ "ErrorEquals": ["States.ALL"], "MaxAttempts": 3, "IntervalSeconds": 2, "BackoffRate": 2 }],
      "Catch":  [{ "ErrorEquals": ["PaymentDeclined"], "Next": "Refund" }],
      "Next": "Notify"
    },
    "Notify": {
      "Type": "Parallel",
      "Branches": [
        { "StartAt": "Email", "States": { "Email": { "Type": "Task", "Resource": "arn:aws:states:::sns:publish", "Parameters": { "TopicArn": "...", "Message.$": "$.orderId" }, "End": true } } },
        { "StartAt": "Warehouse", "States": { "Warehouse": { "Type": "Task", "Resource": "arn:aws:states:::sqs:sendMessage", "Parameters": { "QueueUrl": "...", "MessageBody.$": "$" }, "End": true } } }
      ],
      "End": true
    },
    "Refund": { "Type": "Task", "Resource": "arn:aws:lambda:..:function:refund", "End": true }
  }
}
```

### Start an execution

```bash
aws stepfunctions start-execution \
  --state-machine-arn arn:aws:states:..:stateMachine:OrderFulfillment \
  --input '{"orderId":"ord_42","total":12500}'
```

### CDK example (using L2)

```ts
const charge = new tasks.LambdaInvoke(this, "Charge", { lambdaFunction: chargeFn });
const email  = new tasks.SnsPublish(this, "Email", { topic: emailTopic, message: sfn.TaskInput.fromJsonPathAt("$") });

const definition = charge.next(
  new sfn.Parallel(this, "Notify").branch(email).branch(/* ... */),
);

new sfn.StateMachine(this, "OrderFulfillment", { definitionBody: sfn.DefinitionBody.fromChainable(definition) });
```

## Pricing

- **Standard:** $0.025 per 1,000 state transitions. Every state entry/exit is a transition.
- **Express:** $1.00/M requests + $0.0001/GB-second of duration.

For Standard: a 10-state workflow run 10,000 times = 100k transitions = $2.50.

## Built-in service integrations

- Lambda invoke (sync/async, .waitForTaskToken).
- ECS RunTask.
- SQS SendMessage.
- SNS Publish.
- DynamoDB GetItem/PutItem/UpdateItem.
- S3.
- Glue StartJobRun.
- EventBridge PutEvents.
- Bedrock InvokeModel.
- Plus generic `aws-sdk:*` for almost everything.

Direct integration → no Lambda needed for "just call this API."

## Gotchas

- **Cost balloons on Standard** if you have huge fan-out (Map state with thousands of items). Use Distributed Map.
- **Express has at-least-once semantics** — make tasks idempotent.
- **State machine size limit** ~1 MB ASL — split into nested machines.
- **Input/output size limit** 256 KB per state — pass S3 references for big payloads.
- **Error handling is per-state** — define `Retry` + `Catch` everywhere meaningful.
- **`States.ALL`** catches everything — be selective.

## Related

- [Lambda](../01-compute/lambda.md)
- [EventBridge](./eventbridge.md)
- [SQS](./sqs.md)
- [Batch](../01-compute/batch.md)
