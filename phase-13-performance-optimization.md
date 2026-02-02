# PHASE 13: PERFORMANCE TESTING & OPTIMIZATION

## Overview
Performance isn't an afterthought—it's a feature. Users abandon slow apps (40% leave if page takes >3 seconds). This phase covers load testing, stress testing, profiling, and systematic optimization to ensure your application performs well under real-world conditions.

***

## 13.1 PERFORMANCE REQUIREMENTS

### 13.1.1 Performance Targets

**Define measurable targets:**

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Time to First Byte (TTFB)** | < 200ms | Server response time |
| **Largest Contentful Paint (LCP)** | < 2.5s | Core Web Vital |
| **First Input Delay (FID)** | < 100ms | Core Web Vital |
| **Cumulative Layout Shift (CLS)** | < 0.1 | Core Web Vital |
| **API p50 Latency** | < 100ms | 50th percentile |
| **API p99 Latency** | < 500ms | 99th percentile |
| **Concurrent Users** | 10,000 | Without degradation |
| **Throughput** | 1,000 RPS | Requests per second |

**Performance budget:**
```
JavaScript bundle: < 200KB gzipped
CSS bundle: < 50KB gzipped
Images: < 100KB each (optimized)
Web fonts: < 100KB total
Total page weight: < 1MB
```

**Output:** Performance requirements document

***

### 13.1.2 Performance Baseline

**Establish baseline before optimization:**
```
1. Measure current performance
   - Run load tests at current usage levels
   - Record baseline metrics
   
2. Document current architecture
   - Database query counts per page
   - External API calls per request
   - Cache hit rates
   
3. Identify bottlenecks
   - Slowest endpoints
   - Most expensive queries
   - Memory/CPU intensive operations
```

**Baseline metrics template:**
```
Date: 2024-01-15
Environment: Staging (production-like)

API Performance:
- /api/products: p50=45ms, p99=234ms
- /api/users: p50=23ms, p99=89ms
- /api/orders: p50=120ms, p99=890ms ← Optimization target

Infrastructure:
- CPU utilization: 35% avg
- Memory utilization: 60% avg
- DB connections: 45/100

Frontend:
- LCP: 3.2s ← Above target
- FID: 45ms ✓
- CLS: 0.08 ✓
```

**Output:** Performance baseline report

***

## 13.2 LOAD TESTING

### 13.2.1 Load Testing Strategy

**Load testing types:**

| Type | Purpose | Duration |
|------|---------|----------|
| **Smoke Test** | Verify system works | 1-5 minutes |
| **Load Test** | Normal expected load | 30-60 minutes |
| **Stress Test** | Find breaking point | Until failure |
| **Soak Test** | Extended duration | 4-24 hours |
| **Spike Test** | Sudden traffic surge | 15-30 minutes |

**Load testing tools:**

| Tool | Best For | Learning Curve |
|------|----------|----------------|
| **k6** | Developer-friendly, JavaScript | Low |
| **Locust** | Python, distributed | Medium |
| **Gatling** | JVM, complex scenarios | High |
| **Artillery** | Node.js, simple | Low |
| **JMeter** | GUI, enterprise | Medium |

**Output:** Load testing strategy document

***

### 13.2.2 Load Test Implementation

**k6 load test example:**
```javascript
// load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

// Custom metrics
const errorRate = new Rate('errors');
const orderLatency = new Trend('order_latency');

// Test configuration
export const options = {
  stages: [
    { duration: '2m', target: 100 },   // Ramp up to 100 users
    { duration: '5m', target: 100 },   // Stay at 100 users
    { duration: '2m', target: 200 },   // Ramp up to 200 users
    { duration: '5m', target: 200 },   // Stay at 200 users
    { duration: '2m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% of requests < 500ms
    errors: ['rate<0.01'],              // Error rate < 1%
  },
};

// Test scenario
export default function() {
  // Login
  const loginRes = http.post('https://api.example.com/auth/login', {
    email: 'test@example.com',
    password: 'password123',
  });
  
  check(loginRes, {
    'login successful': (r) => r.status === 200,
  });
  
  const token = loginRes.json('token');
  
  // Authenticated requests
  const headers = { Authorization: `Bearer ${token}` };
  
  // Get products
  const productsRes = http.get('https://api.example.com/products', { headers });
  check(productsRes, {
    'products loaded': (r) => r.status === 200,
  });
  
  sleep(1);
  
  // Create order
  const orderStart = Date.now();
  const orderRes = http.post('https://api.example.com/orders', 
    JSON.stringify({ productId: 'prod_123', quantity: 1 }),
    { headers, headers: { 'Content-Type': 'application/json' } }
  );
  
  orderLatency.add(Date.now() - orderStart);
  errorRate.add(orderRes.status !== 201);
  
  check(orderRes, {
    'order created': (r) => r.status === 201,
  });
  
  sleep(2);
}
```

**Running load tests:**
```bash
# Run test
k6 run load-test.js

# Run with more VUs
k6 run --vus 500 --duration 30m load-test.js

# Output to cloud
k6 run --out cloud load-test.js
```

**Output:** Load test scripts

***

### 13.2.3 Load Test Analysis

**Analyze results:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                    LOAD TEST RESULTS                                 │
├─────────────────────────────────────────────────────────────────────┤
│  Test: Normal Load (200 concurrent users, 15 minutes)               │
│  Date: 2024-01-15                                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  OVERALL:                                                            │
│  ✅ Passed - All thresholds met                                     │
│                                                                      │
│  REQUESTS:                                                           │
│  Total: 45,678                                                       │
│  Success: 45,542 (99.7%)                                            │
│  Failed: 136 (0.3%)                                                 │
│                                                                      │
│  LATENCY:                                                            │
│  p50: 89ms ✅                                                        │
│  p95: 234ms ✅                                                       │
│  p99: 456ms ✅                                                       │
│  max: 2,345ms ⚠️ (spike during ramp-up)                             │
│                                                                      │
│  THROUGHPUT:                                                         │
│  Avg: 50.7 req/s                                                    │
│  Peak: 78.2 req/s                                                   │
│                                                                      │
│  RECOMMENDATIONS:                                                    │
│  1. Investigate latency spike during ramp-up                        │
│  2. /api/orders p99 close to threshold, optimize                    │
│  3. Consider connection pooling for DB                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Output:** Load test report template

***

## 13.3 STRESS TESTING

### 13.3.1 Stress Test Design

**Stress testing goals:**
- Find the breaking point
- Identify failure modes
- Verify graceful degradation
- Test recovery after overload

**Stress test configuration:**
```javascript
// stress-test.js
export const options = {
  stages: [
    { duration: '2m', target: 100 },
    { duration: '5m', target: 100 },
    { duration: '2m', target: 200 },
    { duration: '5m', target: 200 },
    { duration: '2m', target: 300 },
    { duration: '5m', target: 300 },
    { duration: '2m', target: 400 },   // Keep increasing
    { duration: '5m', target: 400 },
    { duration: '10m', target: 0 },    // Recovery period
  ],
  thresholds: {
    http_req_duration: ['p(99)<2000'],  // Allow higher latency
    http_req_failed: ['rate<0.1'],       // Allow some failures
  },
};
```

**What to observe:**
```
During stress test, monitor:
- At what load does latency degrade?
- At what load do errors start?
- Do errors cascade or stay isolated?
- Does the system recover after load decreases?
- What resources exhaust first (CPU, memory, connections)?
```

**Output:** Stress test results with breaking point identified

***

### 13.3.2 Soak Testing

**Soak test (endurance test):**
```javascript
// soak-test.js
export const options = {
  stages: [
    { duration: '5m', target: 100 },   // Ramp up
    { duration: '8h', target: 100 },   // Sustained load for 8 hours
    { duration: '5m', target: 0 },     // Ramp down
  ],
};
```

**Soak test detects:**
- Memory leaks
- Connection pool exhaustion
- Log file growth issues
- Database growth problems
- Time-based bugs (midnight, DST transitions)

**Output:** Soak test report

***

## 13.4 PROFILING & BOTTLENECK IDENTIFICATION

### 13.4.1 Backend Profiling

**Profiling areas:**

| Area | Tool | What to Look For |
|------|------|------------------|
| **CPU** | Node.js profiler, py-spy | Hot functions, blocking code |
| **Memory** | heap snapshot, memory profiler | Leaks, large objects |
| **I/O** | async profiler | Blocking I/O, slow queries |
| **Network** | wireshark, tcpdump | Latency, retransmits |

**Node.js profiling:**
```javascript
// Enable profiling
node --prof app.js

// Process the log
node --prof-process isolate-*.log > profile.txt

// Or use clinic.js
npx clinic doctor -- node app.js
npx clinic flame -- node app.js
npx clinic bubbleprof -- node app.js
```

**Database query analysis:**
```sql
-- PostgreSQL slow query log
ALTER SYSTEM SET log_min_duration_statement = '100ms';

-- Find slow queries
SELECT query, calls, mean_time, total_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 20;

-- Analyze specific query
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 123;
```

**Output:** Profiling results with bottlenecks identified

***

### 13.4.2 Frontend Profiling

**Core Web Vitals measurement:**
```javascript
// Using web-vitals library
import { getCLS, getFID, getLCP, getFCP, getTTFB } from 'web-vitals';

function sendToAnalytics(metric) {
  const body = JSON.stringify({
    name: metric.name,
    value: metric.value,
    id: metric.id,
  });
  navigator.sendBeacon('/analytics', body);
}

getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getLCP(sendToAnalytics);
getFCP(sendToAnalytics);
getTTFB(sendToAnalytics);
```

**Chrome DevTools profiling:**
```
1. Performance tab
   - Record page load
   - Identify long tasks (> 50ms)
   - Check main thread blocking
   
2. Network tab
   - Identify large resources
   - Check waterfall for blocking
   - Verify caching headers
   
3. Lighthouse
   - Run performance audit
   - Follow recommendations
   - Track score over time
```

**Output:** Frontend performance audit

***

## 13.5 DATABASE OPTIMIZATION

### 13.5.1 Query Optimization

**Common query optimizations:**

```sql
-- ❌ BAD: N+1 query pattern
SELECT * FROM orders WHERE id = 1;
SELECT * FROM order_items WHERE order_id = 1;
SELECT * FROM products WHERE id = 101;
SELECT * FROM products WHERE id = 102;
-- ... repeated for each item

-- ✅ GOOD: JOIN to fetch all at once
SELECT o.*, oi.*, p.*
FROM orders o
JOIN order_items oi ON oi.order_id = o.id
JOIN products p ON p.id = oi.product_id
WHERE o.id = 1;
```

**Index optimization:**
```sql
-- Find missing indexes (PostgreSQL)
SELECT schemaname, tablename, 
       seq_scan, seq_tup_read,
       idx_scan, idx_tup_fetch
FROM pg_stat_user_tables
WHERE seq_scan > idx_scan
ORDER BY seq_tup_read DESC;

-- Create index for common query patterns
CREATE INDEX CONCURRENTLY idx_orders_user_status 
ON orders (user_id, status) 
WHERE status IN ('pending', 'processing');

-- Partial index for specific use case
CREATE INDEX CONCURRENTLY idx_orders_pending 
ON orders (created_at) 
WHERE status = 'pending';
```

**Query plan analysis:**
```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM orders 
WHERE user_id = 123 AND status = 'completed'
ORDER BY created_at DESC
LIMIT 10;

-- Look for:
-- - Sequential scans on large tables
-- - High buffer reads
-- - Sorts on unindexed columns
-- - Hash joins on large datasets
```

**Output:** Query optimization recommendations

***

### 13.5.2 Connection Pooling

**Connection pool configuration:**
```javascript
// PostgreSQL with pg-pool
const pool = new Pool({
  host: 'localhost',
  database: 'myapp',
  max: 20,                    // Max connections in pool
  min: 5,                     // Min connections to maintain
  idleTimeoutMillis: 30000,   // Close idle connections after 30s
  connectionTimeoutMillis: 2000, // Timeout waiting for connection
});

// Monitor pool health
setInterval(() => {
  console.log({
    total: pool.totalCount,
    idle: pool.idleCount,
    waiting: pool.waitingCount
  });
}, 10000);
```

**Pool sizing formula:**
```
Max connections = (core_count * 2) + effective_spindle_count

For most apps:
- Small: 10-20 connections
- Medium: 20-50 connections  
- Large: 50-100 connections

Warning signs:
- Waiting count > 0 frequently
- Connection timeout errors
- Idle connections = Max connections (over-provisioned)
```

**Output:** Connection pool configuration

***

## 13.6 CACHING STRATEGIES

### 13.6.1 Caching Layers

**Cache hierarchy:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                        CACHING LAYERS                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   User → [Browser Cache] → [CDN] → [App Cache] → [DB Query Cache]  │
│              ↓              ↓           ↓              ↓            │
│          Static         Edge        Redis/         Query           │
│          assets       caching     Memcached       results          │
│                                                                      │
│   Cache hit = Fast response, no backend work                        │
│   Cache miss = Full request processing                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Cache types and use cases:**

| Layer | TTL | Use Case |
|-------|-----|----------|
| **Browser** | 1 year (versioned assets) | JS, CSS, images |
| **CDN** | 1 day - 1 week | Static content, API responses |
| **Application** | Seconds - hours | Session data, computed results |
| **Database** | Minutes | Query results, aggregations |

**Output:** Caching architecture document

***

### 13.6.2 Application Caching

**Redis caching patterns:**
```javascript
const Redis = require('ioredis');
const redis = new Redis();

// Cache-aside pattern
async function getUser(userId) {
  const cacheKey = `user:${userId}`;
  
  // Try cache first
  const cached = await redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // Cache miss - fetch from DB
  const user = await db.users.findById(userId);
  
  // Store in cache
  await redis.setex(cacheKey, 3600, JSON.stringify(user)); // 1 hour TTL
  
  return user;
}

// Write-through pattern (invalidate on write)
async function updateUser(userId, data) {
  const user = await db.users.update(userId, data);
  
  // Invalidate cache
  await redis.del(`user:${userId}`);
  
  // Or update cache
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));
  
  return user;
}
```

**Cache invalidation strategies:**
```
1. Time-based (TTL)
   - Simple, consistent
   - May serve stale data until expiry
   
2. Event-based
   - Invalidate on write
   - Complex in distributed systems
   
3. Version-based
   - cache key includes version: user:123:v5
   - Increment version on changes
   
4. Tag-based
   - Group related cache entries
   - Invalidate all by tag
```

**Output:** Caching implementation guide

***

## 13.7 FRONTEND OPTIMIZATION

### 13.7.1 Bundle Optimization

**JavaScript optimization:**
```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
        common: {
          minChunks: 2,
          priority: -10,
          reuseExistingChunk: true,
        },
      },
    },
    minimize: true,
    usedExports: true,  // Tree shaking
  },
};

// Dynamic imports for code splitting
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));
```

**Bundle analysis:**
```bash
# Analyze bundle
npx webpack-bundle-analyzer dist/stats.json

# Check bundle size in CI
npx bundlesize

# bundlesize config
{
  "files": [
    {
      "path": "dist/main.*.js",
      "maxSize": "200 kB"
    }
  ]
}
```

***

### 13.7.2 Image Optimization

**Image optimization checklist:**
```
Format selection:
- Photos: WebP (fallback JPEG)
- Graphics/logos: SVG
- Screenshots: PNG or WebP
- Animated: WebP or optimized GIF

Sizing:
- Never serve larger than display size
- Use srcset for responsive images
- Lazy load below-fold images

Compression:
- Target: ~85% quality for JPEG/WebP
- Use tools: ImageMagick, Sharp, Squoosh
```

**Responsive images:**
```html
<picture>
  <source 
    srcset="image-400.webp 400w, image-800.webp 800w, image-1200.webp 1200w"
    type="image/webp"
    sizes="(max-width: 600px) 100vw, 50vw"
  />
  <img 
    src="image-800.jpg" 
    alt="Description"
    loading="lazy"
    decoding="async"
  />
</picture>
```

**Output:** Image optimization guidelines

***

### 13.7.3 Critical Rendering Path

**Optimize critical path:**
```html
<!-- 1. Preconnect to required origins -->
<link rel="preconnect" href="https://api.example.com" />
<link rel="preconnect" href="https://fonts.googleapis.com" />

<!-- 2. Preload critical resources -->
<link rel="preload" href="/fonts/main.woff2" as="font" type="font/woff2" crossorigin />
<link rel="preload" href="/css/critical.css" as="style" />

<!-- 3. Inline critical CSS -->
<style>
  /* Critical CSS for above-fold content */
</style>

<!-- 4. Defer non-critical CSS -->
<link rel="stylesheet" href="/css/main.css" media="print" onload="this.media='all'" />

<!-- 5. Defer non-critical JavaScript -->
<script src="/js/main.js" defer></script>
```

**Output:** Critical rendering path optimization

***

## 13.8 API OPTIMIZATION

### 13.8.1 Response Optimization

**API response best practices:**
```javascript
// Pagination (don't return 10,000 items)
app.get('/api/products', async (req, res) => {
  const { page = 1, limit = 20 } = req.query;
  const offset = (page - 1) * limit;
  
  const [products, total] = await Promise.all([
    db.products.find().skip(offset).limit(limit),
    db.products.countDocuments()
  ]);
  
  res.json({
    data: products,
    pagination: {
      page: parseInt(page),
      limit: parseInt(limit),
      total,
      pages: Math.ceil(total / limit)
    }
  });
});

// Field selection (return only needed fields)
app.get('/api/users/:id', async (req, res) => {
  const fields = req.query.fields?.split(',') || ['id', 'name', 'email'];
  const user = await db.users.findById(req.params.id).select(fields);
  res.json(user);
});

// Compression
app.use(compression({
  filter: (req, res) => {
    if (req.headers['x-no-compression']) return false;
    return compression.filter(req, res);
  },
  level: 6  // Balance between speed and compression ratio
}));
```

***

### 13.8.2 HTTP/2 and Connection Optimization

**HTTP/2 benefits:**
```
- Multiplexing (multiple requests over single connection)
- Header compression
- Server push
- Stream prioritization

Enable HTTP/2:
- Most CDNs support automatically
- Configure in nginx/load balancer
- No application code changes needed
```

**Connection optimization:**
```javascript
// Keep-alive connections to external services
const agent = new https.Agent({
  keepAlive: true,
  keepAliveMsecs: 10000,
  maxSockets: 100,
});

const response = await fetch('https://api.external.com', { agent });
```

**Output:** API optimization checklist

***

## 13.9 PERFORMANCE MONITORING

### 13.9.1 Real User Monitoring (RUM)

**RUM implementation:**
```javascript
// Track real user performance
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    analytics.track('performance', {
      name: entry.name,
      value: entry.value || entry.duration,
      startTime: entry.startTime,
    });
  }
});

observer.observe({ 
  entryTypes: ['largest-contentful-paint', 'first-input', 'layout-shift'] 
});
```

**What to track:**
```
User-centric metrics:
- Page load time by route
- Time to interactive
- Core Web Vitals (LCP, FID, CLS)

Segmentation:
- By device type (mobile, desktop)
- By connection speed (4G, 3G, WiFi)
- By geography
- By browser
```

***

### 13.9.2 Synthetic Monitoring

**Synthetic performance tests:**
```yaml
# Run Lighthouse in CI
lighthouse-ci:
  runs-on: ubuntu-latest
  steps:
    - name: Lighthouse CI
      uses: treosh/lighthouse-ci-action@v10
      with:
        urls: |
          https://example.com/
          https://example.com/products
        budgetPath: ./budget.json
        uploadArtifacts: true
```

**Performance budget:**
```json
{
  "performance": {
    "lcp": 2500,
    "fid": 100,
    "cls": 0.1,
    "ttfb": 600
  },
  "resourceSizes": [
    {
      "resourceType": "script",
      "budget": 300
    },
    {
      "resourceType": "total",
      "budget": 1000
    }
  ]
}
```

**Output:** Performance monitoring setup

***

## 13.10 DELIVERABLES & GATES

### Deliverable 1: Performance Test Results
**Contains:**
- Load test results
- Stress test results (with breaking point)
- Soak test results

### Deliverable 2: Optimization Report
**Contains:**
- Bottlenecks identified
- Optimizations implemented
- Before/after metrics

### Deliverable 3: Performance Monitoring
**Contains:**
- RUM dashboard
- Synthetic monitoring
- Alerting on performance regression

***

## 13.11 PHASE 13 CHECKLIST

- [ ] Performance targets defined
- [ ] Baseline measurements recorded
- [ ] Load tests passing at expected traffic
- [ ] Stress tests completed (breaking point known)
- [ ] Database queries optimized
- [ ] Caching implemented
- [ ] Frontend bundle optimized
- [ ] Images optimized
- [ ] Core Web Vitals passing
- [ ] Performance monitoring in place
- [ ] Performance budget enforced in CI

***

## 13.12 ESTIMATED DURATION

**Timeline: 1-2 weeks**

- Load testing setup and execution: 2-3 days
- Bottleneck identification: 2-3 days
- Optimization implementation: 3-5 days
- Verification and documentation: 1-2 days

***

**END OF PHASE 13**

Fast software is not an accident—it's the result of intentional design, measurement, and optimization. Users notice performance, even when they can't articulate it.
