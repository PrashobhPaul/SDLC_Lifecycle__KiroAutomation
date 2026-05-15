# Architecture — Hexagonal

```
src/
├── domain/                       # entities, value objects, domain services — pure
│   ├── pilot.ts
│   ├── leave-request.ts
│   └── rest-violation.ts
├── application/                  # use cases — orchestration only
│   ├── process-fmla-event.usecase.ts
│   └── compute-rest-violation.usecase.ts
├── ports/                        # interfaces (inbound + outbound)
│   ├── inbound/
│   │   └── fmla-event.port.ts
│   └── outbound/
│       ├── pilot-repository.port.ts
│       └── workday-publisher.port.ts
├── adapters/
│   ├── inbound/                  # SQS, AppSync, EventBridge handlers
│   │   ├── sqs-fmla.adapter.ts
│   │   └── appsync-pilot-resolver.adapter.ts
│   └── outbound/                 # DynamoDB, Aurora, HTTP, Kafka clients
│       ├── ddb-pilot.repository.ts
│       └── kafka-workday.publisher.ts
├── infrastructure/               # cross-cutting: DI container, AWS clients
│   ├── di-container.ts
│   ├── aws-clients.ts
│   └── config.ts
└── shared/                       # pure utilities, no AWS SDK (BE rule 44 — CRITICAL if violated)
    ├── result.ts
    ├── time.ts
    └── ids.ts
```

## Hard rules (mapped to BE audit)

| Rule | Severity | Source |
|------|----------|--------|
| `domain/` imports `@aws-sdk` / `axios` / ORMs | CRITICAL | BE rule 10 |
| `application/` imports `@aws-sdk` / `axios` / ORMs | CRITICAL | BE rule 10 |
| `shared/` imports `@aws-sdk` | CRITICAL | BE rule 44 |
| `domain/` entity contains logging / SDK / DB call | MAJOR | BE rule 70 |
| Business logic outside `domain/` (in adapters/handlers) | MAJOR | BE rule 68 |
| Same domain concept modelled inconsistently across files | MAJOR | BE rule 69 |
| Local interface duplicating domain model type | MAJOR | BE rule 53 |
| Adapter instantiated in domain/use-case (instead of port) | CRITICAL | BE rule 10 |
| Stub/no-op adapter in production | MAJOR | BE rule 58 |

## Direction of dependency
Adapters depend on ports (interfaces). Ports live next to domain.
Use cases depend on ports.
Domain depends on nothing else.
Infrastructure (DI container) wires concrete adapters to ports.

## Inbound vs outbound
- Inbound adapters handle external triggers (SQS, EventBridge, AppSync resolver, HTTP).
- Outbound adapters handle external dependencies (DynamoDB, Aurora, Kafka, HTTP downstream).
- Both implement port interfaces.

## DI container
Single source of truth at `infrastructure/di-container.ts`. Singletons + lazy-init for expensive resources (AWS clients, DB connections). The Developer agent never instantiates SDK clients inside a handler body or catch block (BE rule 29 — MAJOR).
