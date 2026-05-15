# Frontend Testing

## Coverage thresholds (per file)

| Layer | Statements | Branch | Functions |
|-------|-----------|--------|-----------|
| Critical shared hooks/utils | ≥ 95% | ≥ 90% | ≥ 95% |
| Custom hooks | ≥ 90% | ≥ 85% | ≥ 90% |
| Utils / mappers | ≥ 95% | ≥ 90% | ≥ 95% |
| Components | ≥ 70% | ≥ 80% | ≥ 85% |
| Overall | ≥ 90% statements | ≥ 80% branch | ≥ 85% functions |

## Hard rules (mapped to UI audit S)

| Rule | Severity | Source |
|------|----------|--------|
| Coverage below threshold | MAJOR per file | UI checklist S [T2] |
| Failing suite | MAJOR | UI checklist S [T2] |
| `transformIgnorePatterns` missing a needed pkg | CRITICAL | UI rule 17 [T6+T7] |
| `collectCoverageFrom` blanket-excluding `services/**` | MAJOR | UI checklist S [T16] |
| Critical shared hook with 0% coverage | MAJOR | UI checklist S |
| Module-level singleton in production code | MAJOR | UI rule 12 [T15] |
| Test asserts implementation detail (private state, internal call) | MAJOR | UI checklist S |
| Mock data type mismatch with production interface | MAJOR | UI checklist S |
| `await` missing on async test op | MAJOR | UI checklist S |
| Shared mutable state between tests | MAJOR | UI checklist S |
| Snapshot test instead of behavioral assertion | MAJOR | UI checklist S |
| `getByTestId` instead of `getByRole` / `getByLabelText` | MAJOR per instance | UI rule 26 |
| `\|\| true` on test command in CI | MAJOR | UI rule 10 [T8] |

## Query priority

```
getByRole > getByLabelText > getByPlaceholderText > getByText > getByDisplayValue > getByAltText > getByTitle > getByTestId
```

`getByTestId` is the last resort. Couples tests to implementation. Flag every instance.

## RTL patterns

```tsx
// Render with providers
function renderWithProviders(ui: ReactElement, options?: RenderOptions) {
  return render(
    <ThemeProvider>
      <MockedProvider mocks={mocks} addTypename={false}>
        <MemoryRouter>{ui}</MemoryRouter>
      </MockedProvider>
    </ThemeProvider>,
    options
  );
}

// Behavior, not implementation
test('opens the rest details panel on row click', async () => {
  renderWithProviders(<CrewRestPanelContainer />);

  const row = await screen.findByRole('row', { name: /pilot p123/i });
  await userEvent.click(row);

  expect(await screen.findByRole('dialog', { name: /rest details/i })).toBeInTheDocument();
});
```

## Test data factories

```ts
// tests/factories/pilot.ts
import { faker } from '@faker-js/faker';

faker.seed(42);  // deterministic

export function makePilot(overrides: Partial<Pilot> = {}): Pilot {
  return {
    pilotId:   `P${faker.number.int({ min: 100000, max: 999999 })}`,
    name:      faker.person.fullName(),
    base:      faker.helpers.arrayElement(['DAL', 'HOU', 'PHX', 'OAK']),
    updatedAt: faker.date.recent().toISOString(),
    ...overrides,
  };
}
```

Match production interfaces exactly. Use `Partial<T>` overrides only.

## MSW for HTTP / GraphQL

```ts
// tests/mocks/handlers.ts
import { graphql, HttpResponse } from 'msw';
import { GetPilotByIdDocument } from '@/__generated__/graphql';

export const handlers = [
  graphql.query(GetPilotByIdDocument, ({ variables }) => {
    return HttpResponse.json({
      data: { pilotById: makePilot({ pilotId: variables.pilotId }) },
    });
  }),
];
```

## Accessibility tests

```tsx
import { axe, toHaveNoViolations } from 'jest-axe';
expect.extend(toHaveNoViolations);

test('has no a11y violations', async () => {
  const { container } = renderWithProviders(<CrewRestPanelView vm={vm} />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

## E2E (UI audit T)

| Rule | Severity | Source |
|------|----------|--------|
| E2E framework absent | MAJOR architectural | UI rule 19 |
| Critical user flow without E2E | MAJOR | UI checklist T |
| E2E suite not in CI on every PR | MAJOR | UI checklist T |

Phase-1 commitment: Pilot 1 (Crew Rest Violation Alert) has Playwright E2E covering ticket-arrives → handoff → MR-merged. Pilot 2 same.
