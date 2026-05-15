# IAM Least-Privilege + Action↔SDK Cross-Reference

## Hard rules

1. `Action: "*"` AND `Resource: "*"` together → CRITICAL refuse.
2. Either `Action` or `Resource` is specific. Justification comment required when one side is wildcarded.
3. Every IAM action in a Terraform policy must have a corresponding SDK call in the source (BE rule 61).
4. Cross-account assume-role uses ExternalId where the trust account is outside the SWA org.
5. No inline policies on production roles; all policies are managed and named.

## Cross-reference workflow

The Code Reviewer agent runs this check on every Terraform diff:

```bash
node tools/iam-sdk-crossref.ts \
  --terraform terraform/envs/<env>/ \
  --source src/ \
  --report .kiro/handoff/<feature-id>/iam-crossref.md
```

The script:
1. Parses every `aws_iam_policy_document` and extracts (Action, Resource) tuples.
2. Greps source for `@aws-sdk/client-*` imports and `Send` invocations.
3. Maps SDK calls to the IAM actions they require.
4. Flags any IAM action with no corresponding SDK call.
5. Flags any SDK call with no corresponding IAM action (potential runtime authz failure).

## Example

```hcl
data "aws_iam_policy_document" "pilot_lambda" {
  statement {
    sid     = "DynamoDBReadPilots"
    actions = [
      "dynamodb:GetItem",
      "dynamodb:Query"
    ]
    resources = [
      aws_dynamodb_table.pilots.arn,
      "${aws_dynamodb_table.pilots.arn}/index/*"
    ]
  }

  statement {
    sid     = "KMSDecryptForDDBSSE"
    actions = ["kms:Decrypt"]
    resources = [aws_kms_key.ddb.arn]
  }

  statement {
    sid     = "WriteOwnLogs"
    actions = [
      "logs:CreateLogStream",
      "logs:PutLogEvents"
    ]
    resources = ["${aws_cloudwatch_log_group.lambda.arn}:*"]
  }

  statement {
    sid     = "XRayWrite"
    actions = [
      "xray:PutTraceSegments",
      "xray:PutTelemetryRecords"
    ]
    resources = ["*"]   # X-Ray API does not support resource-level permissions; documented.
  }
}
```

## Wildcards
- `Resource: "*"` is allowed for services that do not support resource-level permissions
  (CloudWatch Logs CreateLogGroup, X-Ray Put*). The exception is documented in code.
- `Action: "*"` is never allowed in any production policy.

## Boundary policies
Production accounts apply IAM permission boundaries that prevent privilege escalation.
The Developer agent never proposes a policy that violates the boundary.

## Rotation
- Secrets accessed via SecretsManager have rotation enabled (BE checklist; MAJOR if absent).
- Long-lived API keys are not used; use IAM roles or assumed roles.
