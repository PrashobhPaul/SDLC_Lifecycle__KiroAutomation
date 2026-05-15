# Routing & Navigation

## Hard rules (mapped to UI audit G)

| Rule | Severity | Source |
|------|----------|--------|
| Page component without `React.lazy` | MAJOR per page | UI checklist G |
| Auth-protected route relying only on UI hide | CRITICAL — needs server-side guard | UI checklist G |
| RBAC `PermissionGate` without server-side counterpart | MAJOR | UI checklist G |
| 404 / catch-all fallback route absent | MAJOR | UI checklist G |
| URL/query param state duplicated in `useState` | MAJOR | UI checklist G |

## Lazy-load every page

```tsx
const CrewRestPanel    = lazy(() => import('@/features/crew-rest-violation'));
const LeaveManagement  = lazy(() => import('@/features/leave-management'));
const NotFound         = lazy(() => import('@/routes/NotFound'));

export const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    children: [
      { path: 'crew-rest',       element: <RequireRole role="crew-supervisor"><CrewRestPanel /></RequireRole> },
      { path: 'leave-management', element: <RequireRole role="crew-admin"><LeaveManagement /></RequireRole> },
      { path: '*',               element: <NotFound /> },
    ],
  },
]);
```

Every `lazy(...)` is wrapped by `<Suspense>` and `<ErrorBoundary>` upstream — see `10-suspense-error-boundaries.md`.

## RBAC: UI guard ≠ security

```tsx
// UI hides the menu item — convenience only
function RequireRole({ role, children }: PropsWithChildren<{ role: Role }>) {
  const { roles } = useAuth();
  if (!roles.includes(role)) return <Navigate to="/forbidden" replace />;
  return <>{children}</>;
}
```

Server-side guard MUST exist (Lambda authorizer + resolver-level role check). UI hiding is not security.

## URL is the source of truth

```tsx
// ❌
const [filter, setFilter] = useState('all');
useEffect(() => {
  const sp = new URLSearchParams(location.search);
  setFilter(sp.get('filter') ?? 'all');
}, [location.search]);

// ✅
const [searchParams, setSearchParams] = useSearchParams();
const filter = searchParams.get('filter') ?? 'all';
```

## 404 / catch-all

Defined explicitly. Renders an actionable page (back to home, contact support).
