# BE Lambda Code Review Checklist

Code Reviewer agent runs this. Source = full BE-Lambda audit. Every finding cites file:line + tool ref + concrete fix.

## Tooling pass (T1 – T31)

```bash
mkdir -p /tmp/be-audit
(npx tsc --noEmit > /tmp/be-audit/t1.txt 2>&1) &
(npm test -- --coverage --passWithNoTests --watchAll=false > /tmp/be-audit/t2.txt 2>&1) &
(npm audit --json > /tmp/be-audit/t3.txt 2>/dev/null) &
(npx eslint src --ext .ts > /tmp/be-audit/t4.txt 2>&1) &
(node tools/find-unused-deps.js > /tmp/be-audit/t5.txt) &
(grep -rn "password\|secret\|api_key\|token\|credential\|private_key\|access_key" src/ --include="*.ts" -i \
  | grep -v "//\|test\|spec\|mock\|interface\|type\|param\|SSM\|SecretManager" > /tmp/be-audit/t6.txt) &
(grep -rn "console\.\(log\|warn\|error\|info\|debug\)" src/ --include="*.ts" \
  | grep -v "__tests__\|test\|spec\|logger\|setupTests\|powertools\|Logger" > /tmp/be-audit/t7.txt) &
(grep -rn ": any\|as any\|<any>\|any\[\]" src/ --include="*.ts" \
  | grep -v "__tests__\|test\|spec\|\.d\.ts" > /tmp/be-audit/t8.txt) &
(find src/ -name "*.ts" | grep -v "__tests__\|test\|spec\|\.d\.ts" \
  | xargs wc -l 2>/dev/null | sort -rn | head -20 > /tmp/be-audit/t9.txt) &
(grep -rn "TODO\|FIXME\|HACK\|@ts-ignore\|@ts-nocheck" src/ --include="*.ts" > /tmp/be-audit/t10.txt) &
(grep -rn "from 'aws-sdk'\|require('aws-sdk')" src/ --include="*.ts" > /tmp/be-audit/t11.txt) &
(grep -rn "from '@aws-sdk\|from 'axios\|from 'pg\|from 'mysql\|from 'mongoose\|from 'typeorm" \
  src/domain/ src/application/ 2>/dev/null --include="*.ts" > /tmp/be-audit/t12.txt) &
(grep -rn "\"arn:aws\|\"us-east\|\"us-west\|\"eu-west\|\"ap-" \
  terraform/ infra/ iac/ 2>/dev/null --include="*.tf" \
  | grep -v "variable\|output\|#" > /tmp/be-audit/t13.txt) &
(grep -rn '"[*]"' terraform/ infra/ iac/ 2>/dev/null --include="*.tf" > /tmp/be-audit/t14.txt) &
(grep -rn "environment\|variables" terraform/ 2>/dev/null --include="*.tf" -A 10 \
  | grep -i "password\|secret\|key\|token\|credential" \
  | grep -v "ssm\|secretsmanager\|kms\|#" > /tmp/be-audit/t15.txt) &
(grep -rn "async " src/ --include="*.ts" \
  | grep -v "test\|spec\|__tests__" > /tmp/be-audit/t16.txt) &
(npx madge --circular src/ > /tmp/be-audit/t17.txt 2>&1) &
(npx ts-prune | grep -v "test\|spec\|__tests__\|index" | head -30 > /tmp/be-audit/t18.txt) &
(grep -rn "batchItemFailures\|BatchItemFailures\|SQSBatchResponse" src/ --include="*.ts" > /tmp/be-audit/t19.txt) &
(grep -rn "xray\|AWSXRay\|captureAWS\|opentelemetry" src/ --include="*.ts" -i > /tmp/be-audit/t20.txt) &
(grep -rn "for.*{" src/ --include="*.ts" -A 5 | grep "await" > /tmp/be-audit/t21.txt) &
(grep -rn "\.scan(" src/ --include="*.ts" > /tmp/be-audit/t22.txt) &
(grep -rn "Promise\.all(" src/ --include="*.ts" \
  | grep -v "test\|spec\|__tests__" > /tmp/be-audit/t23.txt) &
(grep -rn "new \(LambdaClient\|DynamoDBClient\|SQSClient\|SNSClient\|S3Client\|SecretsManagerClient\)" \
  src/ --include="*.ts" \
  | grep -v "__tests__\|test\|spec\|Container\|container\|config\|\.config\." > /tmp/be-audit/t24.txt) &
(grep -rn "|| 'local'\||| 'localhost'\||| 'us-east-1'\|| '.*password'\|| '.*secret'" \
  src/ --include="*.ts" \
  | grep -v "__tests__\|test\|spec" > /tmp/be-audit/t25.txt) &
(grep -rn "eslint-disable" src/ --include="*.ts" \
  | grep -v "eslint-disable-next-line\|eslint-disable-line\|__tests__\|test\|spec" > /tmp/be-audit/t26.txt) &
(grep -rn "\.slice(" src/ --include="*.ts" \
  | grep -v "__tests__\|test\|spec" > /tmp/be-audit/t27.txt) &
(grep -rn "^[[:space:]]*//" src/ --include="*.ts" \
  | grep -v "__tests__\|test\|spec\|TODO\|FIXME\|eslint\|@\|http\|https\|import\|export" \
  | head -30 > /tmp/be-audit/t28.txt) &
(find src/ -name "*.ts" -o -name "*.json" \
  | xargs ls -la 2>/dev/null | awk '$5 > 102400 {print $5, $9}' | sort -rn > /tmp/be-audit/t29.txt) &
(grep -rn "from 'dotenv'\|require('dotenv')" src/ --include="*.ts" \
  | grep -v "__tests__\|test\|spec" > /tmp/be-audit/t30.txt) &
(grep -rn "new Date().toISOString\|uuidv4()\|uuid()\|nanoid()" \
  src/adapters/inbound/ --include="*.ts" 2>/dev/null > /tmp/be-audit/t31.txt) &
wait
```

## Output files
- `REVIEW-metrics.md` — header, test results, coverage, tsc errors, npm audit
- `REVIEW-arch.md` — cross-file findings (ARCH-1, ARCH-2, …)
- `REVIEW-files.md` — per-file table `# | Line(s) | Severity | Category | Finding | Fix`
- `REVIEW-summary.md` — Section U (config), severity roll-up, refactor candidates, infra gaps, E2E gap

Write each as soon as ready. Never accumulate findings before writing.

## Severity reference

| Level | Triggers |
|-------|---------|
| **CRITICAL** | Hardcoded secret/credential · IAM `*:*` wildcard · security hole · data loss · crash on prod input · missing auth · SQL injection |
| **MAJOR** | `any` · tsc error · test failure · <80% coverage · layer violation · SDK v2 · no error handling · `console.log` in prod · DRY violation · missing DLQ · no X-Ray · no structured logging · magic string/number · dead export · missing idempotency · N+1 · unhandled promise · missing input validation · file >200 LOC · function >25 LOC · `eslint-disable` w/o reason+ticket |

## Enforcement rules (1–70 from BE audit)

| # | Rule | Severity |
|---|------|----------|
| 1 | Every finding → exact line(s) + tool ref + concrete fix | — |
| 2 | Cross-file issues → reference every affected file | — |
| 3 | Only report what tools confirm or direct inspection proves | — |
| 4 | Write 4 output files; write each as soon as ready | — |
| 5 | File >200 LOC → MAJOR. Function >25 LOC → MAJOR | MAJOR |
| 7 | Pattern in >2 files → REVIEW-arch.md | — |
| 8 | Every CRITICAL → ready-to-use code fix | — |
| 9 | Coverage from T2 only. TypeScript errors from T1 only | — |
| 10 | Domain/application importing infrastructure | CRITICAL |
| 11 | `any` in prod | MAJOR per instance |
| 12 | Hardcoded secret/credential | CRITICAL |
| 13 | IAM `"*"` without justification | CRITICAL |
| 14 | AWS SDK v2 | MAJOR per file |
| 15 | `console.log` in prod | MAJOR per file |
| 16 | Missing DLQ on async trigger | MAJOR arch |
| 17 | No X-Ray/OTEL tracing | MAJOR arch |
| 18 | SQS without `batchItemFailures` | MAJOR |
| 19 | Empty catch block | MAJOR per instance |
| 20 | No integration test suite | MAJOR arch |
| 21 | `await` inside loop | MAJOR per instance |
| 22 | Mapper with side effects | MAJOR per instance |
| 23 | Error detail in API response | CRITICAL |
| 24 | No retry on transient AWS errors | MAJOR per adapter |
| 25 | Missing idempotency on SQS/SNS/EventBridge | MAJOR |
| 26 | Unbounded in-memory accumulation | MAJOR per instance |
| 27 | DynamoDB `Scan` without justification | MAJOR |
| 28 | `Promise.all` where `Promise.allSettled` needed | MAJOR |
| 29 | Infrastructure client/service in handler/catch body | MAJOR |
| 30 | Hardcoded fallback in config | MAJOR |
| 31 | `eslint-disable` w/o reason + ticket | MAJOR per instance |
| 32 | In-memory pagination | MAJOR |
| 33 | N+1 DB pattern | MAJOR per occurrence |
| 34 | Commented-out business logic w/o ticket | MAJOR |
| 36 | Coverage-padding test | MAJOR per file |
| 37 | `(obj as any)` to access private method in test | MAJOR |
| 38 | `jest.doMock` inside test body | MAJOR |
| 39 | Within-lambda duplication (>2 files) | MAJOR |
| 40 | Static file >100KB in `src/` | MAJOR |
| 41 | Business constant hardcoded; ESLint complexity >15 | MAJOR |
| 42 | `noUnusedParameters: false` in tsconfig | MAJOR |
| 43 | Unnecessary indirection layer | MAJOR |
| 44 | `shared/` with AWS SDK imports | CRITICAL |
| 45 | Build artifacts/debug files committed | MAJOR per file |
| 46 | CloudWatch log group without retention | MAJOR |
| 47 | Lambda event typed as `any` | MAJOR per handler |
| 48 | `catch (error: any)` or unnarrowed catch | MAJOR |
| 49 | No schema validation library for complex input | MAJOR |
| 50 | `jest.clearAllMocks()` absent from `beforeEach` | MAJOR |
| 51 | Input validated in handler AND use case | MAJOR |
| 52 | Value generated in handler but ignored downstream | MAJOR |
| 53 | Local interface duplicating domain model type | MAJOR |
| 54 | Dead factory/utility method | MAJOR |
| 55 | `dotenv` in production Lambda source | MAJOR |
| 56 | Env var default inconsistent with Terraform value | MAJOR |
| 57 | Typed error caught and returned as success | MAJOR |
| 58 | Stub/no-op adapter method in production | MAJOR |
| 59 | Function with >5 positional parameters | MAJOR |
| 60 | `strictPropertyInitialization: false` in tsconfig | MAJOR |
| 61 | IAM action with no corresponding SDK call in source | MAJOR |
| 62 | Deprecated Lambda runtime | MAJOR |
| 63 | `x86_64` architecture without justification | MAJOR |
| 64 | Bundle >50MB zipped | MAJOR |
| 65 | `NODE_OPTIONS=--enable-source-maps` absent | MAJOR |
| 66 | Terraform state not separated per environment | MAJOR |
| 67 | No Renovate/Dependabot | MAJOR |
| 68 | Business logic outside domain layer | MAJOR per instance |
| 69 | Same domain concept modelled inconsistently | MAJOR |
| 70 | Domain entity method containing logging/SDK/DB | MAJOR |
