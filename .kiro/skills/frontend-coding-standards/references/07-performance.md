# Performance

## Single-spa MFE vs standard SPA

| Architecture | webpack `splitChunks` | webpack `output.filename` | Note |
|--------------|----------------------|--------------------------|------|
| single-spa MFE (`libraryTarget: "system"`) | intentionally absent | no `[contenthash:8]` | UI rule 9 — do NOT flag |
| Standard SPA | `{ chunks: 'all' }` required | `[contenthash:8]` required | UI rules in H |

Detection: T17 (`grep "libraryTarget\|system\|single-spa" webpack.config.js`).

## Hard rules (mapped to UI audit H)

| Rule | Severity | Source |
|------|----------|--------|
| Standard SPA missing `splitChunks: { chunks: 'all' }` | MAJOR | UI checklist H |
| Standard SPA missing `[contenthash:8]` in output filename | MAJOR | UI checklist H |
| Heavy lib (pdfmake, xlsx, chart libs) not dynamically imported | MAJOR | UI checklist H |
| Expensive array op (sort/filter/map on >20 items) outside `useMemo` | MAJOR per instance | UI checklist H |
| Inline object/function/array literals as JSX props | MAJOR per instance | UI checklist H |
| List with >20 items without `react-window` / `react-virtual` | MAJOR | UI checklist H |
| Image missing `loading="lazy"` / wrong format / missing dimensions | MAJOR | UI checklist H |
| Fonts not preloaded / missing `font-display: swap` | MAJOR | UI checklist H |
| Unused dep in bundle | MAJOR | UI checklist H [T5] |
| `JSON.stringify(variables)` in render or hook body for change detection | MAJOR | UI checklist H |
| High-frequency event handler (scroll/resize/input) without debounce/throttle | MAJOR | UI checklist H |
| Interval/subscription/listener without cleanup in `useEffect` return | MAJOR (memory leak) | UI checklist H |
| Web Vitals risk (LCP >2.5s, CLS >0.1, INP >200ms) | MAJOR with cause | UI checklist H |

## Patterns

```tsx
// Stable callbacks
const handleClick = useCallback((id: string) => {
  navigate(`/pilot/${id}`);
}, [navigate]);

// Stable list item to prevent re-renders
const PilotRow = memo(({ pilot, onClick }: { pilot: Pilot; onClick: (id: string) => void }) => (
  <Row onClick={() => onClick(pilot.id)}>{pilot.name}</Row>
));

// Memoised expensive op
const sortedItems = useMemo(
  () => items.toSorted((a, b) => a.priority - b.priority),
  [items]
);
```

```tsx
// Debounce input
const debounced = useMemo(
  () => debounce((q: string) => onSearch(q), 250),
  [onSearch]
);
useEffect(() => () => debounced.cancel(), [debounced]);
```

```tsx
// Virtualised list
import { FixedSizeList } from 'react-window';

<FixedSizeList height={600} itemCount={items.length} itemSize={48} width="100%">
  {({ index, style }) => <Row style={style} pilot={items[index]!} />}
</FixedSizeList>
```

```tsx
// Cleanup
useEffect(() => {
  const onResize = () => setWidth(window.innerWidth);
  window.addEventListener('resize', onResize);
  return () => window.removeEventListener('resize', onResize);
}, []);
```

## Heavy libs — dynamic import

```tsx
const exportToXlsx = async (rows: Row[]) => {
  const { utils, writeFile } = await import('xlsx');
  const wb = utils.book_new();
  utils.book_append_sheet(wb, utils.json_to_sheet(rows), 'Sheet1');
  writeFile(wb, 'export.xlsx');
};
```
