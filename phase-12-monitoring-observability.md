# PHASE 12: MONITORING & OBSERVABILITY

## Overview
You can't fix what you can't see. Monitoring and observability transform your application from a black box into a transparent system where issues are detected before users notice them. This phase covers the three pillars of observability: metrics, logs, and traces—plus alerting that pages humans when the robots need help.

***

## 12.1 OBSERVABILITY FUNDAMENTALS

### 12.1.1 Three Pillars of Observability

**The pillars:**

```
┌─────────────────────────────────────────────────────────────────────┐
│                    THREE PILLARS OF OBSERVABILITY                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────┐      ┌─────────────┐      ┌─────────────┐         │
│   │   METRICS   │      │    LOGS     │      │   TRACES    │         │
│   │             │      │             │      │             │         │
│   │ • Counters  │      │ • Events    │      │ • Request   │         │
│   │ • Gauges    │      │ • Context   │      │   journey   │         │
│   │ • Histograms│      │ • Errors    │      │ • Latency   │         │
│   │             │      │             │      │   breakdown │         │
│   │ "What's     │      │ "Why did    │      │ "Where is   │         │
│   │  happening?"│      │  it fail?"  │      │  time spent?"│        │
│   └─────────────┘      └─────────────┘      └─────────────┘         │
│                                                                      │
│   Combined: Full understanding of system behavior                    │
└─────────────────────────────────────────────────────────────────────┘
```

**When to use each:**

| Pillar | Use Case | Example Question |
|--------|----------|------------------|
| **Metrics** | Aggregate health, trends | "What's our error rate over the last hour?" |
| **Logs** | Specific event details | "Why did user X's payment fail?" |
| **Traces** | Request flow across services | "Which service is slowing down this request?" |

**Output:** Observability strategy document

***

### 12.1.2 Observability Maturity Model

**Levels of maturity:**

| Level | Description | Capabilities |
|-------|-------------|--------------|
| **0: Reactive** | Fix problems after user complaints | Basic uptime check |
| **1: Proactive** | Basic monitoring, manual investigation | Metrics + logs |
| **2: Predictive** | Alerting on anomalies, fast debugging | + Traces, dashboards |
| **3: Autonomous** | Self-healing, ML-driven insights | + AIOps, auto-remediation |

**Minimum viable observability:**
```
Level 1 (Day 1):
- Health endpoints
- Error rate monitoring
- Basic application logging
- Uptime alerts

Level 2 (Month 1):
- Custom metrics
- Structured logging
- Distributed tracing
- Performance dashboards
- Intelligent alerting
```

**Output:** Observability roadmap

***

## 12.2 METRICS

### 12.2.1 Metric Types

**Metric categories:**

| Type | Description | Example |
|------|-------------|---------|
| **Counter** | Cumulative, only increases | Total requests, errors |
| **Gauge** | Point-in-time value | Current memory usage, queue size |
| **Histogram** | Distribution of values | Request latency percentiles |
| **Summary** | Similar to histogram, client-side | Request duration quantiles |

**USE Method (Resources):**
```
For each resource (CPU, memory, disk, network):
- Utilization: % time resource is busy
- Saturation: Queue length, backlog
- Errors: Error count
```

**RED Method (Services):**
```
For each service:
- Rate: Requests per second
- Errors: Errors per second
- Duration: Latency distribution
```

**Golden signals (Google SRE):**
```
1. Latency: How long requests take
2. Traffic: How many requests
3. Errors: How many requests fail
4. Saturation: How full is the system
```

**Output:** Metrics taxonomy document

***

### 12.2.2 Application Metrics

**Essential application metrics:**

```javascript
// Metric naming convention: <namespace>_<subsystem>_<name>_<unit>
// Example: app_http_request_duration_seconds

// HTTP metrics
http_requests_total { method, path, status }       // Counter
http_request_duration_seconds { method, path }     // Histogram
http_requests_in_flight                            // Gauge

// Business metrics
orders_created_total { type }                      // Counter
order_value_dollars { type }                       // Histogram
active_users_gauge                                 // Gauge

// Database metrics
db_query_duration_seconds { query_type }           // Histogram
db_connections_active                              // Gauge
db_connections_max                                 // Gauge

// Queue metrics
queue_messages_total { queue, action }             // Counter
queue_depth { queue }                              // Gauge
queue_processing_duration_seconds { queue }        // Histogram
```

**Metric instrumentation (Node.js + Prometheus):**
```javascript
const prometheus = require('prom-client');

// Request duration histogram
const httpDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 2, 5, 10]
});

// Middleware to record metrics
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpDuration
      .labels(req.method, req.route?.path || 'unknown', res.statusCode)
      .observe(duration);
  });
  
  next();
});
```

**Output:** Metrics implementation guide

***

### 12.2.3 Infrastructure Metrics

**System metrics to collect:**

```
CPU:
- cpu_usage_percent (per core, total)
- cpu_load_average (1m, 5m, 15m)

Memory:
- memory_used_bytes
- memory_available_bytes
- memory_swap_used_bytes

Disk:
- disk_used_bytes (per mount)
- disk_read_bytes_total
- disk_write_bytes_total
- disk_io_time_seconds

Network:
- network_receive_bytes_total
- network_transmit_bytes_total
- network_errors_total

Container:
- container_cpu_usage_seconds
- container_memory_usage_bytes
- container_restart_count
```

**Cloud infrastructure metrics:**
```
AWS:
- CloudWatch metrics (auto-collected)
- RDS: connections, CPU, storage
- ECS/EKS: task health, container metrics
- ALB: request count, latency, 5xx errors

GCP:
- Cloud Monitoring (Stackdriver)
- Similar metric categories

Azure:
- Azure Monitor
- Similar metric categories
```

**Output:** Infrastructure metrics checklist

***

## 12.3 LOGGING

### 12.3.1 Logging Strategy

**Log levels:**

| Level | Usage | Example |
|-------|-------|---------|
| **ERROR** | Failures requiring attention | Payment processing failed |
| **WARN** | Potential issues, not failures | Retry attempt 2 of 3 |
| **INFO** | Normal operations, key events | User logged in, order created |
| **DEBUG** | Detailed debugging info | Function parameters, state |
| **TRACE** | Very detailed, rarely enabled | Every function entry/exit |

**What to log:**
```
✅ DO LOG:
- Request start/end with duration
- Authentication events (login, logout, failures)
- Business events (order created, payment processed)
- Errors with stack traces
- External service calls (with latency)
- State transitions (status changes)
- Security-relevant events

❌ DON'T LOG:
- Passwords, tokens, secrets
- Full credit card numbers
- Personal data without masking (consider GDPR)
- Sensitive business data
- Health checks (too noisy)
- Every database query (unless debugging)
```

**Output:** Logging policy document

***

### 12.3.2 Structured Logging

**JSON structured logs:**
```javascript
// ❌ BAD: Unstructured log
console.log(`User ${userId} logged in from ${ip}`);
// Output: "User 123 logged in from 192.168.1.1"

// ✅ GOOD: Structured log
logger.info('User logged in', {
  userId: '123',
  ip: '192.168.1.1',
  userAgent: 'Mozilla/5.0...',
  correlationId: 'abc-123'
});
// Output: {"level":"info","message":"User logged in","userId":"123","ip":"192.168.1.1",...}
```

**Standard log schema:**
```json
{
  "timestamp": "2024-01-15T10:30:00.123Z",
  "level": "info",
  "message": "Request completed",
  "service": "api-gateway",
  "environment": "production",
  "correlationId": "req-abc-123",
  "traceId": "trace-xyz-789",
  "userId": "user-456",
  "duration_ms": 234,
  "statusCode": 200,
  "method": "GET",
  "path": "/api/users/123",
  "extra": {
    // Request-specific context
  }
}
```

**Logger configuration (Winston example):**
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: 'api-server',
    environment: process.env.NODE_ENV
  },
  transports: [
    new winston.transports.Console(),
    // Production: ship to log aggregator
    ...(process.env.NODE_ENV === 'production' ? [
      new winston.transports.Http({
        host: 'logs.example.com',
        port: 443,
        ssl: true
      })
    ] : [])
  ]
});
```

**Output:** Structured logging implementation guide

***

### 12.3.3 Correlation IDs

**Request tracing with correlation IDs:**
```javascript
const { v4: uuidv4 } = require('uuid');

// Middleware to add correlation ID
app.use((req, res, next) => {
  // Use existing ID from header or generate new one
  const correlationId = req.headers['x-correlation-id'] || uuidv4();
  
  // Attach to request
  req.correlationId = correlationId;
  
  // Add to response headers
  res.setHeader('x-correlation-id', correlationId);
  
  // Add to async context (for access in services)
  asyncLocalStorage.run({ correlationId }, () => {
    next();
  });
});

// All logs include correlation ID automatically
const log = (level, message, extra = {}) => {
  const store = asyncLocalStorage.getStore();
  logger.log(level, message, {
    correlationId: store?.correlationId,
    ...extra
  });
};
```

**Cross-service propagation:**
```javascript
// When calling another service
const response = await fetch('https://api.service-b.com/data', {
  headers: {
    'x-correlation-id': req.correlationId,
    'x-trace-id': req.traceId
  }
});
```

**Output:** Correlation ID implementation guide

***

### 12.3.4 Log Aggregation

**Log aggregation tools:**

| Tool | Best For | Complexity |
|------|----------|------------|
| **ELK Stack** (Elasticsearch, Logstash, Kibana) | Self-hosted, powerful | High |
| **Loki + Grafana** | Kubernetes, cost-effective | Medium |
| **CloudWatch Logs** | AWS-native | Low |
| **Datadog Logs** | Full platform, expensive | Low |
| **Splunk** | Enterprise, compliance | High |

**Log retention policy:**
```
Production logs:
- Hot storage (fast queries): 7-14 days
- Warm storage (slower queries): 30-90 days
- Cold storage (archive): 1-7 years (compliance)

Development/Staging:
- 7-14 days (no need for long retention)

Security logs:
- Minimum 1 year (compliance requirements)
```

**Output:** Log aggregation architecture

***

## 12.4 DISTRIBUTED TRACING

### 12.4.1 Tracing Fundamentals

**Why distributed tracing:**
```
Without tracing:
  Request → [Black Box: 500ms] → Response
  
With tracing:
  Request → [API: 20ms] → [Auth: 50ms] → [DB: 380ms] → [Response: 50ms]
                                              ↑
                                        Bottleneck identified!
```

**Trace components:**
```
Trace: End-to-end request journey
├── Span: Individual operation (e.g., "database query")
│   ├── Name: "users.findById"
│   ├── Duration: 45ms
│   ├── Status: OK
│   ├── Attributes: { userId: "123", db: "postgres" }
│   └── Events: Errors, logs within span
└── Context: Trace ID, Span ID, propagated across services
```

**Trace visualization:**
```
┌─────────────────────────────────────────────────────────────────────┐
│  TRACE: req-abc-123                                Total: 523ms     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  [API Gateway]═════════════════════════════════════════════════════│
│       ↓                                                              │
│  [Auth Service]════════════════                                     │
│       ↓           (100ms)                                            │
│  [User Service]═══════════════════════════════════════════         │
│       ↓                           (300ms)                            │
│  [PostgreSQL]═══════════════════                                    │
│                  (150ms)                                             │
│       ↓                                                              │
│  [Redis Cache]════                                                   │
│                (23ms)                                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Output:** Tracing architecture document

***

### 12.4.2 OpenTelemetry Implementation

**OpenTelemetry (OTEL) - industry standard:**
```javascript
// opentelemetry.js - Initialize before app code
const { NodeSDK } = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-http');

const sdk = new NodeSDK({
  serviceName: 'api-server',
  traceExporter: new OTLPTraceExporter({
    url: 'https://otel-collector.example.com/v1/traces'
  }),
  instrumentations: [
    getNodeAutoInstrumentations({
      '@opentelemetry/instrumentation-fs': { enabled: false },
      '@opentelemetry/instrumentation-http': { enabled: true },
      '@opentelemetry/instrumentation-express': { enabled: true },
      '@opentelemetry/instrumentation-pg': { enabled: true }
    })
  ]
});

sdk.start();
```

**Custom spans:**
```javascript
const { trace } = require('@opentelemetry/api');

async function processOrder(order) {
  const tracer = trace.getTracer('order-service');
  
  return tracer.startActiveSpan('processOrder', async (span) => {
    try {
      span.setAttribute('order.id', order.id);
      span.setAttribute('order.total', order.total);
      
      // Validate order
      await tracer.startActiveSpan('validateOrder', async (validateSpan) => {
        await validateOrder(order);
        validateSpan.end();
      });
      
      // Process payment
      await tracer.startActiveSpan('processPayment', async (paymentSpan) => {
        paymentSpan.setAttribute('payment.method', order.paymentMethod);
        await processPayment(order);
        paymentSpan.end();
      });
      
      span.setStatus({ code: SpanStatusCode.OK });
    } catch (error) {
      span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
      span.recordException(error);
      throw error;
    } finally {
      span.end();
    }
  });
}
```

**Output:** OpenTelemetry setup guide

***

## 12.5 ERROR TRACKING

### 12.5.1 Error Tracking Setup

**Error tracking tools:**

| Tool | Best For | Features |
|------|----------|----------|
| **Sentry** | All platforms | Excellent DX, releases, replays |
| **Bugsnag** | Mobile focus | Stability scoring |
| **Rollbar** | Simple setup | Real-time grouping |
| **Datadog APM** | Full platform | Combined with APM |

**Sentry integration:**
```javascript
const Sentry = require('@sentry/node');

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  release: process.env.GIT_SHA,
  
  // Performance monitoring
  tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
  
  // Filter sensitive data
  beforeSend(event) {
    // Remove sensitive data from payload
    if (event.request?.data) {
      delete event.request.data.password;
      delete event.request.data.creditCard;
    }
    return event;
  },
  
  integrations: [
    new Sentry.Integrations.Http({ tracing: true }),
    new Sentry.Integrations.Express({ app }),
    new Sentry.Integrations.Postgres()
  ]
});

// Express error handler (after all routes)
app.use(Sentry.Handlers.errorHandler());
```

**Error context enrichment:**
```javascript
// Add user context
app.use((req, res, next) => {
  if (req.user) {
    Sentry.setUser({
      id: req.user.id,
      email: req.user.email,
      username: req.user.username
    });
  }
  next();
});

// Add custom context to errors
try {
  await processOrder(order);
} catch (error) {
  Sentry.setContext('order', {
    orderId: order.id,
    total: order.total,
    items: order.items.length
  });
  Sentry.captureException(error);
  throw error;
}
```

**Output:** Error tracking implementation guide

***

### 12.5.2 Error Alerting

**Error alert priorities:**
```
Critical (Page immediately):
- Error rate > 5% of requests
- 100+ errors in 5 minutes
- Any error in payment flow
- Authentication service down

High (Alert within 15 min):
- Error rate > 1% of requests
- New error type (never seen before)
- Repeated errors for same user

Medium (Review daily):
- Rare errors (< 10/day)
- Known issues with workarounds
- Third-party API errors
```

**Output:** Error alerting rules

***

## 12.6 DASHBOARDS

### 12.6.1 Dashboard Design

**Dashboard hierarchy:**
```
┌─────────────────────────────────────────────────────────────────────┐
│  LEVEL 1: EXECUTIVE / STATUS PAGE                                   │
│  • Is the system healthy? (Green/Yellow/Red)                        │
│  • Key business metrics (orders, revenue)                           │
│  Audience: Everyone, Status page                                    │
├─────────────────────────────────────────────────────────────────────┤
│  LEVEL 2: SERVICE OVERVIEW                                          │
│  • Each service: latency, error rate, request rate                  │
│  • Infrastructure health                                            │
│  Audience: On-call engineers                                        │
├─────────────────────────────────────────────────────────────────────┤
│  LEVEL 3: SERVICE DEEP-DIVE                                         │
│  • Endpoint-level metrics                                           │
│  • Database query performance                                       │
│  • Cache hit rates                                                  │
│  Audience: Service owners                                           │
├─────────────────────────────────────────────────────────────────────┤
│  LEVEL 4: DEBUGGING                                                 │
│  • Traces, logs                                                     │
│  • Individual request analysis                                      │
│  Audience: Engineers during incidents                               │
└─────────────────────────────────────────────────────────────────────┘
```

**Dashboard best practices:**
- One glance = system health assessment
- Group related metrics together
- Use consistent color coding (green=good, red=bad)
- Include comparison baselines (vs yesterday, vs last week)
- Link to related dashboards and runbooks

**Output:** Dashboard design guidelines

***

### 12.6.2 Key Dashboards

**System Health Dashboard:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                    SYSTEM HEALTH DASHBOARD                          │
├──────────────────────────┬──────────────────────────────────────────┤
│  Overall Status: 🟢 OK   │  Active Incidents: 0                     │
├──────────────────────────┴──────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────┐  ┌─────────────────────────┐           │
│  │ Requests/sec: 1,234     │  │ Error Rate: 0.12%       │           │
│  │ [████████████████░░░░]  │  │ [░░░░░░░░░░░░░░░░░░░░]  │           │
│  │ ↑ 5% vs yesterday       │  │ ↓ -0.05% vs yesterday   │           │
│  └─────────────────────────┘  └─────────────────────────┘           │
│                                                                      │
│  ┌─────────────────────────┐  ┌─────────────────────────┐           │
│  │ p99 Latency: 234ms      │  │ Active Users: 5,432     │           │
│  │ [████████░░░░░░░░░░░░]  │  │ [████████████████████]  │           │
│  │ ↓ -12ms vs yesterday    │  │ ↑ 12% vs yesterday      │           │
│  └─────────────────────────┘  └─────────────────────────┘           │
│                                                                      │
│  Service Health:                                                     │
│  [API Gateway: 🟢] [Auth: 🟢] [Orders: 🟢] [Payments: 🟢]           │
│                                                                      │
│  Infrastructure:                                                     │
│  [CPU: 45%] [Memory: 62%] [DB Connections: 78/100]                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Output:** Dashboard templates

***

## 12.7 ALERTING

### 12.7.1 Alert Design

**Alert philosophy:**
```
Every alert should:
1. Be actionable (someone can do something)
2. Be urgent (needs attention now)
3. Have a clear runbook link
4. Reduce noise (avoid alert fatigue)

If an alert fires and no one acts → delete the alert
```

**Alert structure:**
```
Alert name: High Error Rate - API Gateway
Severity: Critical

Condition: error_rate > 5% for 5 minutes

Context:
- Current error rate: 7.2%
- Baseline (last week): 0.3%
- Affected endpoints: /api/orders, /api/checkout
- Sample errors: [link to log query]

Runbook: https://runbooks.example.com/high-error-rate

Actions:
1. Check recent deployments
2. Check external dependencies
3. Consider rollback if deployment-related
```

**Alert thresholds:**
```
Error rate:
- Warning: > 1% for 5 minutes
- Critical: > 5% for 2 minutes

Latency (p99):
- Warning: > 500ms for 5 minutes
- Critical: > 2s for 2 minutes

Availability:
- Warning: < 99.9% (4.3 min downtime/month exceeded)
- Critical: < 99.5% (21.6 min downtime/month exceeded)

Resource usage:
- Warning: CPU > 80% for 10 minutes
- Critical: Memory > 90% for 5 minutes
```

**Output:** Alert configuration guide

***

### 12.7.2 On-Call & Escalation

**On-call rotation:**
```
Schedule:
- Primary on-call: 1 week rotation
- Secondary on-call: Backup if primary unavailable
- Escalation: Manager → VP Engineering → CTO

Response SLAs:
- Critical: Acknowledge within 5 minutes
- High: Acknowledge within 15 minutes
- Medium: Acknowledge within 1 hour

Handoff:
- End of shift: Review open incidents
- Document current state
- Pass any warnings/concerns
```

**Escalation policy:**
```
Escalation timeline:
0 min  → Alert fires, page primary on-call
5 min  → No ack, page secondary on-call
15 min → No ack, page engineering manager
30 min → No ack, page VP engineering
60 min → No ack, page CTO + incident commander
```

**Output:** On-call policy document

***

## 12.8 HEALTH CHECKS

### 12.8.1 Health Check Implementation

**Health check endpoints:**
```javascript
// Basic liveness (is the process alive?)
app.get('/health/live', (req, res) => {
  res.status(200).json({ status: 'ok' });
});

// Readiness (can it serve traffic?)
app.get('/health/ready', async (req, res) => {
  const checks = {
    database: await checkDatabase(),
    redis: await checkRedis(),
    externalApi: await checkExternalApi()
  };
  
  const allHealthy = Object.values(checks).every(c => c.healthy);
  
  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? 'ready' : 'not ready',
    checks,
    timestamp: new Date().toISOString()
  });
});

// Detailed health (for debugging)
app.get('/health/detailed', requireAdminAuth, async (req, res) => {
  res.json({
    status: 'ok',
    version: process.env.VERSION,
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    connections: {
      database: dbPool.totalCount,
      redis: redisClient.status
    }
  });
});
```

**Health check response:**
```json
{
  "status": "ready",
  "checks": {
    "database": {
      "healthy": true,
      "latency_ms": 2
    },
    "redis": {
      "healthy": true,
      "latency_ms": 1
    },
    "externalApi": {
      "healthy": true,
      "latency_ms": 45
    }
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**Output:** Health check implementation guide

***

### 12.8.2 Synthetic Monitoring

**Synthetic checks (from outside your network):**
```
What to monitor:
- Homepage loads successfully
- Login flow works
- Critical API endpoints respond
- SSL certificate valid (> 30 days remaining)
- DNS resolution working

Frequency:
- Critical paths: Every 1 minute
- Secondary paths: Every 5 minutes
- From multiple geographic locations
```

**Tools:**
- **Pingdom**: Simple uptime monitoring
- **Datadog Synthetics**: Full browser tests
- **Checkly**: API + browser monitoring
- **UptimeRobot**: Free tier available

**Output:** Synthetic monitoring configuration

***

## 12.9 STATUS PAGE

### 12.9.1 Status Page Setup

**Status page content:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                      STATUS.EXAMPLE.COM                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Current Status: All Systems Operational         🟢                 │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │ Component          │ Status      │ Details                   │  │
│  ├───────────────────────────────────────────────────────────────│  │
│  │ Website            │ 🟢 Operational │                          │  │
│  │ API                │ 🟢 Operational │                          │  │
│  │ Dashboard          │ 🟢 Operational │                          │  │
│  │ Payment Processing │ 🟡 Degraded    │ Slower than usual        │  │
│  │ Mobile App         │ 🟢 Operational │                          │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Past Incidents:                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │ Jan 12 - API Latency Increase (Resolved)                      │  │
│  │ Jan 8 - Payment Provider Outage (Resolved)                    │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Uptime (90 days): 99.98%                                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Status page tools:**
- **Statuspage.io** (Atlassian)
- **Instatus**
- **Cachet** (self-hosted)

**Output:** Status page configuration

***

## 12.10 DELIVERABLES & GATES

### Deliverable 1: Monitoring Architecture
**Contains:**
- Tool selection for metrics, logs, traces
- Data retention policies
- Integration diagram

### Deliverable 2: Instrumentation
**Contains:**
- Metrics implementation in code
- Structured logging setup
- Distributed tracing configuration

### Deliverable 3: Dashboards
**Contains:**
- System health dashboard
- Service dashboards
- Business metrics dashboard

### Deliverable 4: Alerting
**Contains:**
- Alert definitions
- Escalation policies
- Runbooks for each alert

***

## 12.11 PHASE 12 CHECKLIST

- [ ] Metrics collection configured (application + infrastructure)
- [ ] Structured logging implemented
- [ ] Log aggregation working
- [ ] Distributed tracing enabled
- [ ] Error tracking integrated
- [ ] Health check endpoints created
- [ ] System health dashboard live
- [ ] Critical alerts defined
- [ ] On-call rotation established
- [ ] Status page created
- [ ] Synthetic monitoring active
- [ ] Runbooks for common issues documented

***

## 12.12 ESTIMATED DURATION

**Timeline: 1-2 weeks**

- Tool selection and setup: 2-3 days
- Instrumentation: 3-4 days
- Dashboards: 2-3 days
- Alerting and runbooks: 2-3 days

***

**END OF PHASE 12**

The best incident is the one that never reaches users because monitoring caught it first. Invest in observability early—it pays dividends every time something goes wrong.
