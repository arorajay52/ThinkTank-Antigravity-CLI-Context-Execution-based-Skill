# React Performance & Optimization Reference

This document outlines guidelines for profiling React components and applying targeted optimizations to prevent rendering bottlenecks.

---

## 1. Profiling & Measurement (DevTools First)

Never perform premature optimizations. Always verify bottlenecks using measurements before editing code.

### Verification Steps
1.  Open the browser developer tools and go to the **React DevTools → Profiler** tab.
2.  Click **Record**, interact with the slow UI component, and click **Stop**.
3.  Inspect the **Flamegraph**:
    *   Any component rendering longer than **16ms** (1 frame at 60fps) is a candidate for optimization.
    *   Look for yellow/red blocks representing long-running execution times.
4.  Use the **Ranked Chart** to sort components by self-render time and identify top offenders.
5.  Check render counts. If a component re-renders 50 times during a simple keypress, find the state churn.

---

## 2. Optimization Patterns

### Pattern A: Isolate Churning (Ticking) State
If a single section of a page updates frequently (e.g. scroll listener, typing inputs, timers, animation loops), isolate that state inside a small, dedicated leaf component so it does not trigger re-renders across the parent page structure.

```tsx
// ❌ BEFORE: Entire page re-renders every second, including the heavy list
function Page({ heavyData }) {
  const [seconds, setSeconds] = useState(0);
  useEffect(() => {
    const interval = setInterval(() => setSeconds(s => s + 1), 1000);
    return () => clearInterval(interval);
  }, []);

  return (
    <div>
      <h1>Seconds Elapsed: {seconds}</h1>
      <HeavyDashboard data={heavyData} />
    </div>
  );
}

// ✅ AFTER: Only the Timer component re-renders; HeavyDashboard remains untouched
function Timer() {
  const [seconds, setSeconds] = useState(0);
  useEffect(() => {
    const interval = setInterval(() => setSeconds(s => s + 1), 1000);
    return () => clearInterval(interval);
  }, []);
  return <h1>Seconds Elapsed: {seconds}</h1>;
}

function Page({ heavyData }) {
  return (
    <div>
      <Timer />
      <HeavyDashboard data={heavyData} />
    </div>
  );
}
```

---

### Pattern B: Stabilizing Props & Handlers (`useCallback` + `memo`)
Passing raw inline objects or functions down as props causes child components to re-render on every parent render (because object references change). Use `useCallback` and `useMemo` combined with `React.memo` to prevent unnecessary sub-tree renders.

```tsx
// ❌ BEFORE: Inline function creates a new reference on every render, bypassing Row memo
function List({ items }) {
  const handleSelect = (id) => console.log("Selected:", id);
  return items.map(item => <Row key={item.id} item={item} onSelect={handleSelect} />);
}

// ✅ AFTER: Stable callback reference keeps child Row from re-rendering unless its item changes
const Row = React.memo(({ item, onSelect }) => (
  <div onClick={() => onSelect(item.id)}>{item.name}</div>
));

function List({ items }) {
  const handleSelect = useCallback((id) => console.log("Selected:", id), []);
  return items.map(item => <Row key={item.id} item={item} onSelect={handleSelect} />);
}
```

---

### Pattern C: Derived Computations (`useMemo`)
Avoid doing expensive calculations (sorting, filtering, mapping large datasets) directly inside the render block. Memoize the result to compute only when inputs change.

```tsx
// ✅ CORRECT: Calculates only when the raw data or filter term changes
const filteredData = useMemo(() => {
  return largeDataset.filter(item => item.category === activeCategory);
}, [largeDataset, activeCategory]);
```
