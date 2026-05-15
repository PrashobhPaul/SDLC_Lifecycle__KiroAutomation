# Lambda Defaults

## Required configuration

```hcl
# === Module: lambda-fn ===

resource "aws_lambda_function" "this" {
  function_name = "${var.project}-${var.bounded_context}-${var.name}"
  role          = aws_iam_role.this.arn

  # Runtime & architecture
  runtime       = var.runtime          # default "nodejs22.x"
  architectures = [var.architecture]   # default ["arm64"] — Graviton2

  # Bundle
  filename         = var.bundle_path
  source_code_hash = filebase64sha256(var.bundle_path)
  handler          = var.handler

  # Sizing
  memory_size = var.memory_mb          # right-sized per fn — never 128 default for DB/HTTP
  timeout     = var.timeout_seconds    # aligned with downstream

  # Concurrency
  reserved_concurrent_executions = var.reserved_concurrency

  # Tracing
  tracing_config {
    mode = "Active"                    # X-Ray on (BE rule 17)
  }

  # Env vars
  environment {
    variables = merge(
      {
        NODE_OPTIONS                          = "--enable-source-maps"
        AWS_NODEJS_CONNECTION_REUSE_ENABLED   = "1"
        LOG_LEVEL                             = var.log_level
        POWERTOOLS_SERVICE_NAME               = var.name
      },
      var.environment_variables
    )
  }

  # Networking (when required)
  vpc_config {
    subnet_ids         = var.subnet_ids
    security_group_ids = var.security_group_ids
  }

  # DLQ for async invocations
  dead_letter_config {
    target_arn = aws_sqs_queue.dlq.arn
  }

  tags = local.tags
}

# === Always-on log group with retention ===

resource "aws_cloudwatch_log_group" "this" {
  name              = "/aws/lambda/${aws_lambda_function.this.function_name}"
  retention_in_days = var.log_retention_days   # default 30
  kms_key_id        = aws_kms_key.logs.arn
  tags              = local.tags
}

# === DLQ ===

resource "aws_sqs_queue" "dlq" {
  name                              = "${aws_lambda_function.this.function_name}-dlq"
  kms_master_key_id                 = aws_kms_key.queues.id
  message_retention_seconds         = 1209600   # 14 days
  tags                              = local.tags
}

# === CloudWatch alarms ===

resource "aws_cloudwatch_metric_alarm" "errors" {
  alarm_name          = "${aws_lambda_function.this.function_name}-errors"
  namespace           = "AWS/Lambda"
  metric_name         = "Errors"
  statistic           = "Sum"
  period              = 300
  evaluation_periods  = 2
  threshold           = 1
  comparison_operator = "GreaterThanOrEqualToThreshold"
  dimensions          = { FunctionName = aws_lambda_function.this.function_name }
  alarm_actions       = [aws_sns_topic.alarms.arn]
  treat_missing_data  = "notBreaching"
}

resource "aws_cloudwatch_metric_alarm" "throttles"      { /* throttles >= 1 */ }
resource "aws_cloudwatch_metric_alarm" "duration_p99"   { /* duration p99 > timeout * 0.9 */ }
resource "aws_cloudwatch_metric_alarm" "dlq_depth"      { /* DLQ ApproximateNumberOfMessagesVisible > 0 */ }
```

## Defaults summary

| Setting | Default | Override conditions |
|---------|---------|---------------------|
| runtime | nodejs22.x | Older runtime requires justification |
| architecture | arm64 | x86_64 only with documented reason (BE rule 63) |
| memory_size | 512 MB | Right-size; never 128 for DB/HTTP |
| timeout | 30s | Aligned with longest downstream call + buffer |
| log retention | 30 days | Compliance may require longer |
| tracing | Active (X-Ray) | Required (BE rule 17) |
| DLQ | required | Always present (BE rule 16) |
| KMS encryption (logs, DLQ) | required | Always present |
| `NODE_OPTIONS=--enable-source-maps` | required | BE rule 65 |
| `AWS_NODEJS_CONNECTION_REUSE_ENABLED=1` | required | Always present |

## Provisioned concurrency
Documented per resolver when latency budget demands it. The Developer agent surfaces the trade-off in
the IaC plan and fires G4.

## Layers
- OTEL SDK as a Layer when bundle size pressure exists.
- AWS Powertools for Lambda as a Layer (logger, tracer, metrics).
- Layer publishing is centralised under `terraform/shared/layers/`.

## Bundle size
- Unzipped < 250 MB (AWS hard limit), zipped < 50 MB (BE rule 64).
- Tree-shake; mark dev-only deps as `devDependencies`.
- Any artifact > 50 MB zipped fails CI.
