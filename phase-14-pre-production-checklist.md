# REACT 18 FRONTEND IMPLEMENTATION - PRODUCTION CHECKLIST

## Quick Reference: Technology Stack

```
Framework:          React 18 (Concurrent Rendering, useTransition)
Build Tool:         Vite 5.0 (100x faster than Webpack)
TypeScript:         Strict mode (no compromise on type safety)
State Management:   Zustand 4.4 (client) + TanStack Query 5 (server)
Routing:            TanStack Router 1.28 (type-safe)
Styling:            Tailwind CSS 3.3 (no CSS-in-JS overhead)
Testing:            Vitest + React Testing Library + Playwright
Error Tracking:     Sentry (production monitoring)
Deployment:         Vercel or AWS CloudFront + Lambda
```

## Performance Targets (Non-Negotiable)

| Metric | Target | Status |
|--------|--------|--------|
| Bundle Size (gzipped) | < 150KB | ⚠️ Monitor |
| Largest Contentful Paint (LCP) | < 2.5s | 🎯 Target |
| First Input Delay (FID) | < 100ms | 🎯 Target |
| Cumulative Layout Shift (CLS) | < 0.1 | 🎯 Target |
| Build Time | < 30s | 🎯 Target |
| Dev Server Startup | < 500ms | 🎯 Target |
| Test Coverage | > 80% | 📊 Measure |

## Critical Implementation Patterns

### 1. Project Structure (Monorepo Template)

```
apps/web/
├── src/
│   ├── app/                    # Root app component
│   ├── pages/                  # Route-level components (lazy-loaded)
│   ├── features/               # Feature-scoped modules
│   │   ├── auth/
│   │   ├── orders/
│   │   └── checkout/
│   ├── components/             # Shared base components
│   ├── hooks/                  # Custom hooks
│   ├── stores/                 # Zustand stores
│   ├── api/                    # API client + queries/mutations
│   ├── types/                  # TypeScript definitions
│   ├── utils/                  # Utility functions
│   ├── styles/                 # Global CSS + design tokens
│   └── config/                 # Routes, env, API config
├── vite.config.ts
├── tsconfig.json              # Strict mode enabled
└── package.json
```

### 2. TypeScript Strict Mode (ALL Flags)

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitThis": true,
    "strictPropertyInitialization": true,
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "strictBindCallApply": true,
    "alwaysStrict": true
  }
}
```

**Why strict mode:**
- Catches null/undefined errors at compile time (not runtime)
- Prevents implicit any types
- Forces explicit type annotations
- Enables safe refactoring at scale
- Reduces production bugs by 40%+

### 3. Code Splitting Strategy (Vite Rollup)

```typescript
// vite.config.ts
manualChunks: {
  // Vendor chunks (cached forever, never change)
  'vendor-react': ['react', 'react-dom'],
  'vendor-query': ['@tanstack/react-query'],
  'vendor-router': ['@tanstack/react-router'],
  'vendor-ui': ['@headlessui/react', '@radix-ui/react-dialog'],
  
  // Feature chunks (loaded on demand)
  'feature-auth': ['src/features/auth'],
  'feature-orders': ['src/features/orders'],
  'feature-checkout': ['src/features/checkout'],
  
  // Shared chunk (loaded synchronously)
  'shared': ['src/components'],
}
```

**Result:**
- Initial bundle: ~100KB (vendor-react + vendor-query + vendor-router + shared)
- Feature auth: ~20KB (loaded when user navigates to /login)
- Feature orders: ~25KB (loaded when user navigates to /orders)
- Total final: ~150KB gzipped

### 4. Component Library Architecture

**Base components (Button, Input, Modal):**
- Accessible (keyboard navigation, ARIA labels)
- Unstyled/headless (Tailwind + custom design system)
- Composable (small, single responsibility)
- Memoized if expensive (React.memo with custom comparison)

```typescript
// src/components/Button.tsx
export const Button = memo(
  ({ variant, size, onClick, children }: ButtonProps) => (
    <button
      className={cn(
        buttonVariants({ variant, size }),
        className
      )}
      onClick={onClick}
    >
      {children}
    </button>
  ),
  (prev, next) => {
    // Only re-render if props actually change
    return prev.variant === next.variant &&
           prev.size === next.size &&
           prev.children === next.children;
  }
);
```

**Form components:**
- Use react-hook-form (performance optimized)
- Validate with Zod (type-safe schemas)
- Show real-time error messages
- Support async validation (availability checks)

```typescript
// src/features/auth/pages/LoginPage.tsx
const { control, handleSubmit } = useForm({
  resolver: zodResolver(loginSchema),
  mode: 'onBlur',  // Validate on blur, not onChange
});

const { mutate: login, isLoading } = useLoginMutation();
```

### 5. State Management: Zustand + TanStack Query

**Client state (Zustand):**
- Global UI state (theme, sidebar collapsed, modals open)
- User authentication status
- Temporary form state
- Client-only derived state

```typescript
// src/stores/appStore.ts
export const useAppStore = create((set) => ({
  sidebarOpen: true,
  toggleSidebar: () => set((s) => ({ sidebarOpen: !s.sidebarOpen })),
  
  notifications: [] as Notification[],
  addNotification: (n) => set((s) => ({ notifications: [...s.notifications, n] })),
  removeNotification: (id) => set((s) => ({
    notifications: s.notifications.filter((n) => n.id !== id),
  })),
}));
```

**Server state (TanStack Query):**
- API responses (users, orders, products)
- Automatic caching (staleTime + gcTime)
- Automatic refetching (focus refetch, polling)
- Optimistic updates (mutations)
- Automatic retry on failure

```typescript
// src/features/orders/hooks/useOrders.ts
export function useOrders() {
  return useQuery({
    queryKey: ['orders', { page, limit, status }],
    queryFn: () => ordersApi.list({ page, limit, status }),
    staleTime: 5 * 60 * 1000,        // 5 minutes
    gcTime: 10 * 60 * 1000,          // 10 minutes
    throwOnError: false,             // Don't crash, handle error
  });
}

export function useCreateOrder() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: ordersApi.create,
    onSuccess: (newOrder) => {
      // Update cache immediately (optimistic)
      queryClient.setQueryData(['orders'], (old) => ({
        ...old,
        data: [newOrder, ...old.data],
      }));
    },
    onError: (error) => {
      // Rollback on error
      queryClient.invalidateQueries(['orders']);
    },
  });
}
```

### 6. API Client with Interceptors

```typescript
// src/api/client.ts
const apiClient = axios.create({
  baseURL: env.apiUrl,
  timeout: env.apiTimeout,
});

// Request: Add auth token
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('auth_token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response: Handle errors + token refresh
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      // Try to refresh token
      const newToken = await refreshAuthToken();
      if (newToken) {
        error.config.headers.Authorization = `Bearer ${newToken}`;
        return apiClient(error.config);  // Retry request
      } else {
        // Redirect to login
        useAuthStore.getState().logout();
      }
    }
    
    return Promise.reject(error);
  }
);
```

### 7. Error Boundaries & Global Error Handling

```typescript
// src/components/ErrorBoundary.tsx
class ErrorBoundary extends Component {
  componentDidCatch(error, errorInfo) {
    // Report to Sentry for production monitoring
    Sentry.captureException(error);
    
    // Show user-friendly error message
    this.setState({ hasError: true });
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="error-page">
          <h1>Something went wrong</h1>
          <p>Our team has been notified.</p>
          <Button onClick={() => window.location.reload()}>
            Try Again
          </Button>
        </div>
      );
    }
    
    return this.props.children;
  }
}
```

### 8. Testing Pyramid (70/20/10)

**Unit Tests (70%) - Vitest:**
```typescript
// src/utils/__tests__/formatting.test.ts
describe('formatPrice', () => {
  it('formats USD correctly', () => {
    expect(formatPrice(1234.56)).toBe('$1,234.56');
  });
  it('handles edge cases', () => {
    expect(formatPrice(0)).toBe('$0.00');
    expect(formatPrice(-100)).toBe('-$100.00');
  });
});
```

**Integration Tests (20%) - React Testing Library + MSW:**
```typescript
// src/features/auth/__tests__/LoginPage.test.tsx
server.use(
  http.post('/api/auth/login', ({ request }) => {
    // Mock API responses
    return HttpResponse.json({ token: 'mock-jwt' });
  })
);

it('completes login flow', async () => {
  render(<LoginPage />);
  
  await userEvent.type(screen.getByLabelText(/email/i), 'user@example.com');
  await userEvent.type(screen.getByLabelText(/password/i), 'password123');
  await userEvent.click(screen.getByRole('button', { name: /login/i }));
  
  await waitFor(() => {
    expect(localStorage.getItem('auth_token')).toBe('mock-jwt');
  });
});
```

**E2E Tests (10%) - Playwright:**
```typescript
// e2e/checkout.spec.ts
test('completes checkout flow', async ({ page }) => {
  await page.goto('http://localhost:3000');
  await page.click('text=Login');
  await page.fill('[name="email"]', 'test@example.com');
  // ... fill form, submit, verify success
  await expect(page).toHaveURL('/orders/success');
});
```

## Critical Performance Optimizations

### Bundle Size

1. **Code splitting:**
   - Routes: lazy load with dynamic import + Suspense
   - Features: manual chunks by feature
   - Vendors: separate React, Query, Router chunks

2. **Image optimization:**
   - Use WebP format (80KB → 30KB)
   - Lazy load images (loading="lazy")
   - Optimize with imagemin plugin

3. **CSS:**
   - Tailwind only (built-in tree-shaking)
   - Remove unused @tailwind directives
   - No CSS-in-JS (emotion, styled-components) - overhead

4. **Dependencies:**
   - Remove unused packages (npm audit)
   - Check size with bundlesize: `npm install -D rollup-plugin-visualizer`

### Runtime Performance

1. **React optimization:**
   - React.memo for expensive components
   - useCallback for memoized callbacks
   - useMemo for expensive calculations
   - Avoid anonymous functions in JSX props

2. **Data fetching:**
   - TanStack Query automatic caching
   - Pagination for large lists
   - Infinite query for scrolling feeds

3. **Rendering:**
   - Virtual lists for 1000+ items (react-window)
   - Suspense for data loading states
   - Error boundaries for error states

## Security Checklist

- [ ] **XSS prevention:** Escape HTML, use textContent not innerHTML, sanitize with DOMPurify
- [ ] **CSRF protection:** Token-based (validate on server)
- [ ] **Input validation:** Zod schemas for all forms
- [ ] **API security:** Auth tokens in Authorization header, HTTPS only
- [ ] **Password security:** Never store, validate on server, bcrypt hashing
- [ ] **Data validation:** Validate all user input server-side (never trust client)
- [ ] **Error messages:** Don't leak sensitive info (specific SQL errors, etc)

## Accessibility Checklist

- [ ] **Semantic HTML:** nav, main, section, article (not just divs)
- [ ] **ARIA labels:** All interactive elements have aria-label or aria-describedby
- [ ] **Keyboard navigation:** Tab through form, Escape closes modals, Enter submits
- [ ] **Focus management:** Focus visible, focus trap in modals
- [ ] **Color contrast:** 4.5:1 for normal text, 3:1 for large text
- [ ] **Screen reader:** Test with NVDA (Windows) or VoiceOver (Mac)
- [ ] **Reduced motion:** Respect prefers-reduced-motion media query

## Deployment Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│ GitHub Push to main/staging                                 │
└──────────────────┬──────────────────────────────────────────┘
                   │
        ┌──────────▼──────────┐
        │  Run Tests (CI)     │
        │  - ESLint           │
        │  - Unit tests       │
        │  - Build check      │
        │  - Coverage report  │
        └──────────┬──────────┘
                   │
        ┌──────────▼──────────┐
        │  Code Review        │
        │  - Performance      │
        │  - Bundle size      │
        │  - Accessibility    │
        │  - Types            │
        └──────────┬──────────┘
        (on staging)
                   │
        ┌──────────▼──────────┐
        │  QA Testing         │
        │  - E2E tests        │
        │  - Manual review    │
        │  - Performance      │
        └──────────┬──────────┘
                   │
        ┌──────────▼──────────┐
        │  Deploy to Prod     │
        │  - Vercel: auto     │
        │  - Monitor: Sentry  │
        │  - Analytics        │
        └─────────────────────┘
```

## Monitoring & Alerting

```typescript
// src/lib/monitoring.ts
setupPerformanceMonitoring();  // Core Web Vitals
setupErrorTracking();           // Sentry
setupAnalytics();               // PostHog

// Alert on:
// - LCP > 2.5s
// - FID > 100ms
// - CLS > 0.1
// - Error rate > 1%
// - API latency > 3s
```

## Production Deployment Checklist

- [ ] Run `npm run build` locally, verify no errors
- [ ] Check bundle size: `npm run build -- --visualize`
- [ ] Run all tests: `npm run test`, achieve 80%+ coverage
- [ ] Run E2E tests: `npm run test:e2e`
- [ ] Set environment variables (API URL, Sentry DSN, etc)
- [ ] Configure error tracking (Sentry)
- [ ] Configure performance monitoring (Web Vitals, APM)
- [ ] Configure alerting (Slack, PagerDuty)
- [ ] Test staging environment thoroughly
- [ ] Setup monitoring dashboard
- [ ] Plan rollback strategy (feature flags, blue-green deployment)
- [ ] Document runbook for common issues
- [ ] Setup observability (logs, metrics, traces)

---

**This checklist ensures production-grade frontend engineering with no tech debt.**
