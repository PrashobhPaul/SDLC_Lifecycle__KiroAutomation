# Aurora / RDS Access (Calm Context)

## Connection management — RDS Proxy required

```ts
// adapters/outbound/aurora-leave.repository.ts
import { Pool } from 'pg';
import type { LeaveRepositoryPort } from '../../ports/outbound/leave-repository.port';
import { config } from '../../infrastructure/config';

let _pool: Pool | undefined;

function getPool(): Pool {
  if (!_pool) {
    _pool = new Pool({
      host: config.db.proxyEndpoint,
      port: 5432,
      database: config.db.name,
      user: config.db.user,
      ssl: { rejectUnauthorized: true },
      max: 1,
      idleTimeoutMillis: 30_000,
      connectionTimeoutMillis: 3_000,
      statement_timeout: 5_000,
      query_timeout: 5_000,
    });
  }
  return _pool;
}

export class AuroraLeaveRepository implements LeaveRepositoryPort {
  async findById(id: string): Promise<LeaveCase | null> {
    const { rows } = await getPool().query(
      'SELECT id, crew_id, status, start_date, end_date FROM leave_cases WHERE id = $1',
      [id]
    );
    return rows[0] ? toDomain(rows[0]) : null;
  }
}
```

## Hard rules

| Rule | Severity | Source |
|------|----------|--------|
| Direct Aurora connection without RDS Proxy | MAJOR | BE checklist G |
| Pool max > 1 in Lambda | MAJOR | per-Lambda concurrency model |
| Missing `statement_timeout` / `query_timeout` | MAJOR | BE checklist G |
| `rejectUnauthorized: false` | CRITICAL | BE rule 23 / TLS |
| Dynamic query construction from user input | CRITICAL (SQL injection) | BE rule 23 / H |
| ORM (TypeORM, Prisma) imported into domain or application layer | CRITICAL | BE rule 10 |
| Long-running transactions across Lambda invocations | MAJOR | BE checklist G |
| Cursor / streaming for large result sets not used when needed | MAJOR | BE rule 26 |

## Migrations
- Migrations versioned in `db/migrations/<timestamp>-<name>.sql`.
- Migrations applied via a separate Lambda (or CI step) — not from a request-path Lambda.
- Each migration has both `up` and `down`.

## Read replicas
- Read replicas in the same region for read-heavy use cases.
- Aurora Global secondary in us-west-2 for cross-region failover (per multi-region pattern).

## Connection pool sizing in Lambda
- `max: 1` per Lambda execution context. RDS Proxy multiplexes across executions.
- `idleTimeoutMillis` tuned for the Lambda's expected idle pattern.

## Query patterns
- Parameterised queries only. No string concatenation of user input.
- Bound parameters via `$1`, `$2`, … (pg) — never template-literal string interpolation.
- Use `Promise.allSettled` when fanning out reads where partial failure is recoverable.
