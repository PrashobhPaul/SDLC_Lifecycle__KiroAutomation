# Suspense & ErrorBoundary

## Hard rules (mapped to UI audit K)

| Rule | Severity | Source |
|------|----------|--------|
| `ErrorBoundary` component absent from codebase | MAJOR architectural | UI rule 13 |
| Any `React.lazy` without BOTH `<Suspense>` AND `<ErrorBoundary>` ancestors | MAJOR per lazy import | UI rule 14 |
| `ErrorBoundary` retry without key-increment | MAJOR | UI checklist K |
| `ErrorBoundary` fallback without actionable message + retry button | MAJOR | UI checklist K |
| `componentDidCatch` only writes to `console.error` (not observability service) | MAJOR | UI checklist K |
| `<Suspense>` fallback is a centered spinner instead of skeleton matching layout | MAJOR (CLS) | UI checklist K |

## Reference ErrorBoundary

```tsx
// src/shared/components/ErrorBoundary.tsx
import { Component, type PropsWithChildren, type ReactNode } from 'react';
import { logger } from '@/lib/logger';

type Props = PropsWithChildren<{
  fallback: (error: Error, reset: () => void) => ReactNode;
  onError?: (error: Error, info: { componentStack: string }) => void;
}>;

type State = { error: Error | null; key: number };

export class ErrorBoundary extends Component<Props, State> {
  state: State = { error: null, key: 0 };

  static getDerivedStateFromError(error: Error): Partial<State> {
    return { error };
  }

  componentDidCatch(error: Error, info: { componentStack: string }): void {
    logger.error('error-boundary', { error, info });
    this.props.onError?.(error, info);
  }

  reset = (): void => this.setState(s => ({ error: null, key: s.key + 1 }));

  render(): ReactNode {
    if (this.state.error) return this.props.fallback(this.state.error, this.reset);
    return <div key={this.state.key}>{this.props.children}</div>;
  }
}
```

Key-increment on reset forces React to remount the children. Without it, the same error throws again immediately.

## Required wrap pattern

```tsx
const CrewRestPanel = lazy(() => import('@/features/crew-rest-violation'));

<ErrorBoundary fallback={(err, retry) => (
  <ErrorState
    title="Couldn't load Crew Rest"
    description={err.message}
    onRetry={retry}
  />
)}>
  <Suspense fallback={<CrewRestSkeleton />}>
    <CrewRestPanel />
  </Suspense>
</ErrorBoundary>
```

## Skeleton over spinner
A spinner forces layout shift on load (CLS regression). A skeleton matches the final layout shape:

```tsx
function CrewRestSkeleton() {
  return (
    <Stack gap={3}>
      <SkeletonRect width="40%" height={24} />
      <SkeletonRect width="100%" height={120} />
      <SkeletonRect width="60%" height={16} />
    </Stack>
  );
}
```
