# React Performance Optimization Guide

This guide covers low-level (code-focused) and high-level (architectural) performance improvements for React apps. It’s structured for both beginners and experienced developers.

---

## Low-Level Optimizations (Component Level)

### 1. Memoization with `React.memo`, `useMemo`, and `useCallback`
While these hooks help prevent unnecessary re-renders in React 18 and below, React 19 introduces an automatic memoization system that reduces the need for manual usage in many scenarios. However, for performance-critical or complex components, explicit memoization may still provide benefits.

```js
const ExpensiveComponent = React.memo(({ data }) => <div>{data.value}</div>);

const computedValue = useMemo(() => heavyCalc(input), [input]);
const handleClick = useCallback(() => doSomething(), []);
```

### 2. Avoid Anonymous Functions in JSX
```js
// Not ideal
<button onClick={() => doSomething()} />

// Recommended
const handleClick = useCallback(() => doSomething(), []);
<button onClick={handleClick} />
```

### 3. Proper Key Usage in Lists
```js
{items.map(item => <li key={item.id}>{item.name}</li>)}
```
Avoid using array index as a key.

### 4. Component Decomposition
Break large components into smaller ones to isolate renders.

### 5. Avoid Unnecessary State
Use refs for values that don’t trigger re-renders:
```js
const interval = useRef();
```

### 6. Throttle and Debounce Events
```js
const debounced = useCallback(debounce((val) => search(val), 300), []);
```

### 7. Virtualize Large Lists
```jsx
import { FixedSizeList as List } from 'react-window';

<List height={400} itemCount={1000} itemSize={35} width={300}>
  {({ index, style }) => <div style={style}>{items[index]}</div>}
</List>
```

### 8. Lazy Load Components
```js
const LazyComponent = React.lazy(() => import('./HeavyComponent'));

<Suspense fallback={<Loading />}>
  <LazyComponent />
</Suspense>
```

---

## High-Level Optimizations (App Level)

### 1. Production Builds
```bash
npm run build
```
Use minified, tree-shaken builds.

### 2. Code Splitting
```js
const AdminPanel = React.lazy(() => import('./AdminPanel'));
```
Split on route or component level.

### 3. Image Optimization
- Use WebP, AVIF formats
- `loading="lazy"` for non-visible images

### 4. Bundle Size Analysis
```bash
npm install --save-dev source-map-explorer
npx source-map-explorer 'build/static/js/*.js'
```

### 5. Optimize Library Imports
```js
// Not recommended
import _ from 'lodash';

// Recommended
import debounce from 'lodash/debounce';
```

### 6. SSR & SSG
Use Next.js for server-side rendering or static export.

### 7. Data Fetching & Caching
Use React Query or SWR:
```js
const { data } = useQuery('posts', fetchPosts);
```

### 8. Service Workers & PWA
Enable offline mode with:
```js
serviceWorker.register();
```

### 9. CDN for Assets
Deliver static files via CDN for reduced latency.

### 10. Tree Shaking
Use ES module versions like `lodash-es`, `date-fns` for better dead-code elimination.

---

## Benchmarking Tools

| Tool | Purpose |
|------|---------|
| React DevTools Profiler | Analyze component re-renders |
| Chrome DevTools | JS profiling, memory leaks |
| Lighthouse | Audit PWA, performance, accessibility |
| Webpack Bundle Analyzer | Visualize bundle size |
| Why Did You Render | Warn about unnecessary re-renders |

---

Happy optimizing!
