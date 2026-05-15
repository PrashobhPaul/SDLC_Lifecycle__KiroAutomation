# Test Strategy — Test Engineer Template

## Strategy <strategy_id>

**Feature:** <feature-id>
**Layers under test:** <list>
**G5 owner:** test-engineer

## Test plan (per layer)

### react-mfe
- Unit: <component list, target ≥ 70% lines>
- Hook tests: <list, target ≥ 90%>
- Component a11y (jest-axe): <list>
- E2E (playwright): <one or two happy-path flows>

### graphql-schema
- graphql-inspector diff against base: <yes/no>
- Schema codegen `--check`: <yes>

### node-resolver
- aws-sdk-client-mock unit tests: <list>
- Integration tests with local DynamoDB / Aurora (testcontainers): <list>
- Coverage targets: domain ≥ 95%, application ≥ 90%, adapters ≥ 80%, handlers ≥ 70%

### vtl-resolver
- amplify-appsync-simulator goldens: <list, 6 standard cases per template>

### terraform-iac
- `terraform validate` and `terraform plan` non-destructive: required
- tfsec / tflint / checkov: required, no high or critical

## Mock strategy

- Network: MSW (Node mode)
- AWS SDK v3: `aws-sdk-client-mock`
- Kafka: in-memory test consumer

## Approval

> Type the literal word `approve` to proceed to generation.
