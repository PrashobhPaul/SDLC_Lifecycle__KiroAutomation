# OpenTelemetry Tracing

## Trace propagation across agents

A single feature run is one trace. Each agent turn is one or more spans under that trace.

```
trace_id (per feature run)
├── span: feature-builder.intake
├── span: feature-builder.clarify
├── span: feature-builder.handoff   ← envelope written, contains trace_id
│   └── span: developer.intake (parent_span_id = handoff span)
│       └── span: developer.plan
│           └── span: developer.execute.fe
│           └── span: developer.execute.gql
│           └── span: developer.handoff
│               └── span: test-engineer.intake
│                   └── …
```

## Required span attributes
- `kiro.agent` — the agent name
- `kiro.phase` — one of intake/clarify/plan/execute/verify/handoff/review/merge/doc
- `kiro.feature_id`
- `kiro.agent_run_id`
- `kiro.hitl.gate_fired` — when applicable
- `kiro.tokens.in` / `kiro.tokens.out`
- `kiro.guardrail.verdict` — pass/fail/warn

## Backend
Phase 1: traces export to OTel Collector → Jaeger or Tempo.
Phase 2 (post): correlate with downstream AppSync / Lambda traces using AWS X-Ray propagator.

## Propagation across the handoff envelope
The envelope's `observability` block carries `trace_id` and the upstream `span_id`.
Downstream agents set `parent_span_id` to the upstream `span_id` so the trace stitches together.

## Code skeleton (Node-side example)

```ts
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('kiro-calm');

export async function runAgentTurn(input, ctx) {
  return tracer.startActiveSpan('developer.execute', { attributes: { 'kiro.feature_id': ctx.featureId } }, async (span) => {
    try {
      span.setAttribute('kiro.tokens.in', ctx.tokensIn);
      const result = await doWork(input);
      span.setAttribute('kiro.tokens.out', result.tokensOut);
      return result;
    } catch (err) {
      span.recordException(err as Error);
      throw err;
    } finally {
      span.end();
    }
  });
}
```
