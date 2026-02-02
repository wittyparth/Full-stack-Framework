# PHASE 10: DOCUMENTATION - PRODUCTION-GRADE ENGINEERING

**Documentation is not optional. It's infrastructure. Bad documentation kills projects.**

You can have the best code in the world. Without documentation, it's worthless. Teams waste months guessing. Oncall engineers panic at 3 AM. Users file support tickets. Your system fails not because the code is bad, but because nobody understands how it works.

This phase creates the documentation system that keeps your organization functioning at scale.

---

## 10.1 TECHNICAL DOCUMENTATION ARCHITECTURE

### 10.1.1 Documentation Structure (The 4-Layer System)

```
┌─────────────────────────────────────────────────────────┐
│ LAYER 1: REFERENCE DOCUMENTATION                       │
│ What: API docs, database schema, config options        │
│ Format: OpenAPI 3.1 + auto-generated from code          │
│ Audience: Developers integrating with your system       │
│ Automation: 100% generated from code comments           │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│ LAYER 2: ARCHITECTURE DOCUMENTATION                    │
│ What: System design, data flows, component relationships│
│ Format: Markdown + diagrams (C4 model or Mermaid)       │
│ Audience: Engineers implementing features               │
│ Update: Quarterly or with major architectural changes   │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│ LAYER 3: OPERATIONAL DOCUMENTATION                     │
│ What: Runbooks, incident response, scaling procedures  │
│ Format: Step-by-step checklists + automation scripts    │
│ Audience: On-call engineers, SRE team                   │
│ Update: Weekly or with new alerts/incidents             │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│ LAYER 4: USER DOCUMENTATION                            │
│ What: How to use features, troubleshooting, FAQs        │
│ Format: Markdown guides + interactive tutorials         │
│ Audience: Product users, admins, customers             │
│ Update: With every product release                      │
└─────────────────────────────────────────────────────────┘
```

**Why this structure:**
- Separates concerns (different audiences, update cadences)
- Each layer is owned by different people
- Automation reduces manual documentation burden
- Clear update responsibility (not "someone should update docs")

### 10.1.2 API Documentation (OpenAPI 3.1 Standard)

**Every API must have OpenAPI specification. Non-negotiable.**

```yaml
# docs/api.openapi.yaml
openapi: 3.1.0
info:
  title: Orders API
  version: 1.0.0
  description: Production order management system
  contact:
    name: API Support
    url: https://support.company.com
    email: api-support@company.com
  license:
    name: Proprietary
    url: https://company.com/license

servers:
  - url: https://api.company.com/v1
    description: Production
  - url: https://api-staging.company.com/v1
    description: Staging (non-production data)

tags:
  - name: Orders
    description: Order operations
  - name: Authentication
    description: Auth endpoints
  - name: Users
    description: User management

paths:
  /orders:
    post:
      summary: Create a new order
      description: |
        Creates a new order in the system.
        
        **Authorization**: Requires ROLE_USER or higher
        **Rate Limit**: 100 requests per minute
        **Idempotency**: Use X-Idempotency-Key header to prevent duplicates
      operationId: createOrder
      tags:
        - Orders
      security:
        - BearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrderRequest'
            examples:
              simple:
                summary: Simple order
                value:
                  items:
                    - productId: "prod-123"
                      quantity: 2
              withShipping:
                summary: Order with custom shipping
                value:
                  items:
                    - productId: "prod-456"
                      quantity: 1
                  shippingAddress:
                    street: "123 Main St"
                    city: "New York"
                    zipCode: "10001"
      responses:
        '201':
          description: Order created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '400':
          description: Invalid request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                invalidItems:
                  summary: Items list is empty
                  value:
                    code: "INVALID_REQUEST"
                    message: "Items list cannot be empty"
                    details: ["items must have at least 1 element"]
        '401':
          $ref: '#/components/responses/UnauthorizedError'
        '429':
          $ref: '#/components/responses/RateLimitError'
        '500':
          $ref: '#/components/responses/ServerError'

    get:
      summary: List all orders
      description: Paginated list of orders for the authenticated user.
      operationId: listOrders
      tags:
        - Orders
      security:
        - BearerAuth: []
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            minimum: 1
            default: 1
          description: Page number (1-indexed)
        - name: limit
          in: query
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
          description: Items per page
        - name: status
          in: query
          schema:
            type: string
            enum: [PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED]
          description: Filter by order status
      responses:
        '200':
          description: Orders retrieved
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderListResponse'
        '401':
          $ref: '#/components/responses/UnauthorizedError'
        '500':
          $ref: '#/components/responses/ServerError'

  /orders/{orderId}:
    parameters:
      - name: orderId
        in: path
        required: true
        schema:
          type: string
        description: The unique order identifier
    
    get:
      summary: Get order details
      operationId: getOrder
      tags:
        - Orders
      security:
        - BearerAuth: []
      responses:
        '200':
          description: Order details
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '404':
          description: Order not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '401':
          $ref: '#/components/responses/UnauthorizedError'

components:
  schemas:
    CreateOrderRequest:
      type: object
      required:
        - items
      properties:
        items:
          type: array
          minItems: 1
          items:
            $ref: '#/components/schemas/OrderItem'
        shippingAddress:
          $ref: '#/components/schemas/Address'
        customerNotes:
          type: string
          maxLength: 500

    Order:
      type: object
      required:
        - id
        - userId
        - items
        - status
        - totalAmount
        - createdAt
      properties:
        id:
          type: string
          example: "order-abc123"
        userId:
          type: string
          example: "user-123"
        items:
          type: array
          items:
            $ref: '#/components/schemas/OrderItem'
        status:
          type: string
          enum: [PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED]
          description: Current order status
        totalAmount:
          type: number
          format: decimal
          example: 99.99
        shippingAddress:
          $ref: '#/components/schemas/Address'
        createdAt:
          type: string
          format: date-time
        updatedAt:
          type: string
          format: date-time

    OrderItem:
      type: object
      required:
        - productId
        - quantity
        - price
      properties:
        productId:
          type: string
        quantity:
          type: integer
          minimum: 1
        price:
          type: number
          format: decimal

    Address:
      type: object
      required:
        - street
        - city
        - zipCode
      properties:
        street:
          type: string
        city:
          type: string
        state:
          type: string
        zipCode:
          type: string
        country:
          type: string
          default: "US"

    ErrorResponse:
      type: object
      required:
        - code
        - message
      properties:
        code:
          type: string
          description: Error code (e.g., INVALID_REQUEST, NOT_FOUND)
        message:
          type: string
          description: Human-readable error message
        details:
          type: array
          items:
            type: string
          description: Additional error details

    OrderListResponse:
      type: object
      required:
        - data
        - pagination
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/Order'
        pagination:
          type: object
          required:
            - page
            - limit
            - total
          properties:
            page:
              type: integer
            limit:
              type: integer
            total:
              type: integer

  responses:
    UnauthorizedError:
      description: Authentication required or invalid token
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'

    RateLimitError:
      description: Rate limit exceeded
      headers:
        X-RateLimit-Limit:
          schema:
            type: integer
        X-RateLimit-Remaining:
          schema:
            type: integer
        X-RateLimit-Reset:
          schema:
            type: integer
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'

    ServerError:
      description: Internal server error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'

  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: |
        Authentication using JWT bearer token.
        Obtain token at /auth/login.
        Format: Authorization: Bearer <token>
```

**Generate API docs from OpenAPI:**
```bash
# Redoc (beautiful static docs)
docker run -p 8080:80 -e SPEC_URL=http://api.example.com/openapi.yaml redocly/redoc

# SwaggerUI (interactive, try-it-out)
docker run -p 8080:8080 -v $PWD/docs/api.openapi.yaml:/usr/share/nginx/html/openapi.yaml swaggerapi/swagger-ui

# Postman Collection (auto-generate Postman from OpenAPI)
npx openapi-to-postman -s docs/api.openapi.yaml -o postman.json
```

### 10.1.3 Architecture Documentation (C4 Model)

**Context → Container → Component → Code (4 levels of detail)**

```markdown
# System Architecture

## Level 1: System Context
Shows how the software system fits in the broader landscape.

```
User → Web Application → API Server → Database
         (Frontend)        (Backend)
                               ↓
                        Third-party: Stripe
                        Third-party: SendGrid
```

User story: User buys product online.
- Frontend sends POST /orders to API
- API validates and persists to database
- API calls Stripe for payment
- API calls SendGrid for confirmation email

## Level 2: Container Diagram
Shows major containers (applications, databases, services).

```
┌─────────────────────────────────────────────────────────┐
│ User                                                    │
└──────────────┬──────────────────────────────────────────┘
               │ HTTPS
               ↓
┌──────────────────────────────────────────────────────────┐
│ Single Page Application (React)                         │
│ - Runs in user's browser                               │
│ - Authenticates user                                   │
│ - Displays orders, products, checkout                  │
└──────────────┬──────────────────────────────────────────┘
               │ REST API (JSON over HTTPS)
               ↓
┌──────────────────────────────────────────────────────────┐
│ API Server (Node.js/Express)                           │
│ - Handles authentication (JWT)                         │
│ - Manages orders, products, users                      │
│ - Business logic                                       │
│ - Rate limiting, validation                            │
└──────────────┬──────────────────────────────────────────┘
               │ TCP Connection
               ↓
┌──────────────────────────────────────────────────────────┐
│ PostgreSQL Database                                     │
│ - Stores users, orders, products                       │
│ - Persists all system data                             │
└──────────────────────────────────────────────────────────┘
```

Technology choices:
- Frontend: React 18 (SPAs can be complex; simpler UIs could use server-rendered HTML)
- Backend: Node.js/Express (JavaScript ecosystem, fast development)
- Database: PostgreSQL (ACID compliance, structured data)

## Level 3: Component Diagram
Shows major components within a container.

```
API Server Components:

┌──────────────────────────────────────────────────────────┐
│ API Server                                              │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Auth Controller       Products Controller              │
│  ├─ POST /auth/login   ├─ GET /products                │
│  ├─ POST /auth/logout  ├─ GET /products/:id            │
│  └─ GET /auth/me       └─ POST /products (admin)        │
│                                                          │
│  Orders Controller                                      │
│  ├─ POST /orders       Auth Service   Payment Service  │
│  ├─ GET /orders        ├─ verifyToken ├─ charge()      │
│  ├─ GET /orders/:id    └─ hashPassword └─ refund()     │
│  └─ PATCH /orders/:id                                  │
│                                                          │
│  Database Layer        Cache Layer (Redis)              │
│  ├─ UserRepository     ├─ SessionCache                 │
│  ├─ OrderRepository    └─ QueryCache                   │
│  └─ ProductRepository                                  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

Responsibilities:
- **Auth Controller**: Handles login/logout, validates JWT tokens
- **Orders Controller**: CRUD for orders, validates order data
- **Products Controller**: Manages product catalog
- **Auth Service**: Password hashing, token generation
- **Payment Service**: Integrates with Stripe, handles refunds
- **Database Layer**: Abstract database access (prevents SQL in controllers)
- **Cache Layer**: Redis for sessions and query caching

## Level 4: Code (Class Diagrams)
Detailed class relationships. Usually generated from code, not written manually.

```
// User entity relationships

User
├─ id: UUID
├─ email: string
├─ password: string (hashed)
├─ createdAt: timestamp
└─ orders: Order[]

Order
├─ id: UUID
├─ userId: UUID
├─ items: OrderItem[]
├─ status: enum (PENDING, SHIPPED, DELIVERED)
├─ totalAmount: decimal
├─ createdAt: timestamp
└─ User

OrderItem
├─ id: UUID
├─ orderId: UUID
├─ productId: UUID
├─ quantity: int
└─ price: decimal
```

## Data Flow Diagrams

### User Places Order (Happy Path)
```
1. Frontend: User clicks "Checkout"
2. Frontend → API: POST /orders { items, shippingAddress }
3. API: Validate order (items exist, user has balance)
4. API → Database: Insert order (status=PENDING)
5. API → Stripe: Charge credit card
6. Stripe → API: Payment confirmed
7. API: Update order (status=CONFIRMED)
8. API → SendGrid: Send confirmation email
9. API → Frontend: { orderId, status }
10. Frontend: Show "Order Confirmed" screen

### Error Case: Payment Fails
```
1-5. [Same as above]
6. Stripe → API: Payment declined
7. API: Delete order (rollback)
8. API → Frontend: { error: "Card declined" }
9. Frontend: Show error message
10. User retries with different card
```

## Integration Points

**External Services:**
- **Stripe**: Payment processing
  - Endpoint: https://api.stripe.com
  - Rate limit: 100 requests/sec
  - Fallback: Manual payment review (queue for admin)
  
- **SendGrid**: Email delivery
  - Endpoint: https://api.sendgrid.com
  - Rate limit: 600 emails/minute
  - Fallback: Queue emails, retry hourly

**Internal Integrations:**
- Frontend ↔ API: JSON REST over HTTPS
- API ↔ Database: PostgreSQL protocol
- API ↔ Cache: Redis (for sessions, query results)
- API ↔ Message Queue: RabbitMQ (async tasks like emails)
```

### 10.1.4 Database Schema Documentation

```markdown
# Database Schema

## Users Table
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  
  -- Credentials (never in logs)
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,  -- bcrypt, never plaintext
  
  -- Profile
  name VARCHAR(255) NOT NULL,
  avatar_url VARCHAR(2048),
  
  -- Metadata
  role ENUM ('user', 'admin', 'support') DEFAULT 'user',
  status ENUM ('active', 'suspended', 'deleted') DEFAULT 'active',
  
  -- Tracking
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP,  -- Soft delete
  
  -- Indexes (critical for performance)
  CONSTRAINT email_lower CHECK (email = LOWER(email))
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_users_created_at ON users(created_at DESC);
```

**Notes:**
- `id`: Use UUID v4 (not sequential integers - don't leak data about user count)
- `password_hash`: Store bcrypt hash, NEVER plaintext. Hash with salt factor 12.
- `role`: User role determines authorization level
- `deleted_at`: Soft delete prevents data loss; use WHERE deleted_at IS NULL in queries
- **Indexes**: Critical for query performance. Index by: id (auto), email (login query), status (filtering)

## Orders Table
```sql
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  
  -- Order metadata
  status ENUM ('pending', 'confirmed', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
  total_amount DECIMAL(10, 2) NOT NULL,
  
  -- Shipping
  shipping_address JSONB NOT NULL,  -- Denormalize for simplicity
  shipping_method ENUM ('standard', 'express', 'overnight') DEFAULT 'standard',
  
  -- Payment
  payment_method ENUM ('credit_card', 'debit_card', 'paypal') NOT NULL,
  stripe_payment_id VARCHAR(255),  -- Reference to external payment
  
  -- Metadata
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);
```

**Notes:**
- `shipping_address`: Stored as JSONB (NoSQL-like flexibility) instead of separate table
- `stripe_payment_id`: Critical for reconciliation; links our order to Stripe's record
- Indexes allow fast queries: "Show me all orders from this user", "Count pending orders"

## Order Items Table
```sql
CREATE TABLE order_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id UUID NOT NULL REFERENCES products(id),
  
  -- Quantity and pricing
  quantity INT NOT NULL CHECK (quantity > 0),
  price_at_purchase DECIMAL(10, 2) NOT NULL,  -- Store snapshot (product price changes)
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_order_items_order_id ON order_items(order_id);
```

**Notes:**
- `price_at_purchase`: Store product price AT TIME OF ORDER (not reference to products table)
- Why? If product price changes later, old orders show correct historical price
- Prevents bugs: "Why does my old order show $5 but the product is $10 now?"

## Products Table
```sql
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  
  -- Basic info
  name VARCHAR(255) NOT NULL,
  description TEXT,
  sku VARCHAR(100) UNIQUE NOT NULL,
  
  -- Pricing
  price DECIMAL(10, 2) NOT NULL,
  cost DECIMAL(10, 2),  -- Internal cost (not exposed to users)
  
  -- Inventory
  quantity_in_stock INT DEFAULT 0,
  reserved_quantity INT DEFAULT 0,  -- Subtract from stock when order pending
  
  -- Metadata
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP  -- Soft delete
);

CREATE INDEX idx_products_sku ON products(sku);
CREATE INDEX idx_products_name ON products(name);
```

## Migrations (Schema Versioning)

```sql
-- migrations/20250124_initial_schema.sql
-- Version: 1
-- Description: Initial database schema

BEGIN;
  CREATE TABLE users (id UUID PRIMARY KEY, ...);
  CREATE TABLE orders (id UUID PRIMARY KEY, ...);
  CREATE TABLE order_items (id UUID PRIMARY KEY, ...);
  CREATE TABLE products (id UUID PRIMARY KEY, ...);
COMMIT;

-- migrations/20250124_add_user_roles.sql
-- Version: 2
-- Description: Add role-based access control
-- Rollback: ALTER TABLE users DROP COLUMN role;

BEGIN;
  ALTER TABLE users ADD COLUMN role ENUM ('user', 'admin', 'support') DEFAULT 'user';
  CREATE INDEX idx_users_role ON users(role);
COMMIT;
```

**Why versioning?**
- Track schema evolution
- Enable rollbacks (if migration fails, revert to previous)
- Document why changes made (comments in migration files)
- Tools (Flyway, Liquibase) automate schema deployment
```

---

## 10.2 OPERATIONAL DOCUMENTATION (RUNBOOKS)

### 10.2.1 Runbook Template (Production-Grade)

```markdown
# Runbook: High CPU Alert

**Severity**: P2 (Service degraded but operational)
**Affected System**: API Server
**Typical Duration**: 5-10 minutes
**Owned By**: SRE Team

---

## Quick Response (First 1 minute)

1. **Acknowledge Alert**
   ```bash
   # In PagerDuty: Accept incident
   # This starts the on-call clock and prevents escalation
   ```

2. **Check Current Status**
   ```bash
   # SSH to bastion
   ssh bastion.company.com
   
   # Check CPU metrics
   curl -s https://monitoring.company.com/api/v1/metrics/cpu?host=api-prod-01 | jq
   
   # Expected: CPU should be < 80%
   # If CPU > 95%: Go to "Immediate Mitigation" below
   ```

3. **Notify Stakeholders**
   - Slack #incidents: "High CPU on api-prod. Investigating."
   - Page on-call engineer if not already

---

## Immediate Mitigation (If CPU > 95%)

**Goal: Restore service within 5 minutes. Root cause investigation comes later.**

```bash
# Step 1: Check for runaway processes
ssh api-prod-01
top -b -n 1 | head -20  # Show top 20 processes by CPU

# Step 2: If specific service is consuming CPU
systemctl status api-server
journalctl -u api-server -n 50  # Last 50 log lines

# Step 3: If logs show errors, restart service
sudo systemctl restart api-server

# Step 4: Monitor CPU for 2 minutes
watch -n 2 'curl -s https://monitoring.company.com/api/v1/metrics/cpu?host=api-prod-01 | jq .cpu'

# Step 5: If CPU returns to normal (< 70%), declare incident resolved
# If CPU remains high, escalate to backend team and go to "Deep Diagnosis"
```

---

## Deep Diagnosis (If CPU High After Restart)

**This is not a quick fix. Gather data for the incident review.**

```bash
# 1. Check for stuck queries
ssh api-prod-01
sudo -u postgres psql -c "SELECT pid, query, query_start FROM pg_stat_activity WHERE state != 'idle' ORDER BY query_start;"

# If queries running for > 5 minutes:
# - Kill the query: SELECT pg_terminate_backend(pid);
# - Notify database team immediately

# 2. Check for memory leaks (Java/Node.js)
ps aux | grep api-server  # Check RSS column (resident memory)
node --max-old-space-size=4096 /path/to/app.js  # Restart with more memory if needed

# 3. Check for file descriptor exhaustion
lsof -p $(pgrep -f api-server) | wc -l  # Should be < 1024
# If high: check for unclosed connections or file handles

# 4. If root cause unknown, scale horizontally
kubectl scale deployment api-server --replicas=5  # Add more pods
# This is band-aid, NOT a fix. Root cause must be found.
```

---

## Communication Template

```
# Immediate (< 2 min)
@channel High CPU alert on api-prod. Investigating cause.

# 5 minutes
Status update: Restarted api-server service. CPU returning to normal.
Monitoring for next 10 minutes.

# Resolved
✅ Incident resolved. CPU stable at 45%. Root cause: missing DB index on orders table.
FYI: @db-team will implement index in next deployment.
Post-incident review scheduled for Thursday.
```

---

## After-Action (Post-Incident Review)

**Schedule within 24 hours. If frequency is > 1x per week, this is critical.**

1. **Timeline** (What happened?)
   - 14:32 UTC: Alert triggered (CPU > 90%)
   - 14:33 UTC: On-call engineer paged
   - 14:35 UTC: Restarted service
   - 14:37 UTC: CPU returned to normal
   - **MTTR (Mean Time To Resolution): 5 minutes**

2. **Root Cause** (Why did it happen?)
   - Database query on orders table missing index
   - Query taking 30 seconds instead of 50ms
   - Every request from web app triggered this query
   - Under normal load: 10 slow queries/min (unnoticed)
   - Under peak load: 100 slow queries/min (CPU spike)

3. **Prevention** (How do we prevent recurrence?)
   - [ ] Create index: `CREATE INDEX idx_orders_user_id ON orders(user_id);`
   - [ ] Add test: Slow query monitor (alert if query > 1s)
   - [ ] Deploy monitoring: Query execution time dashboard
   - [ ] Owner: @db-team, Deadline: This sprint

4. **Learning**
   - Better: Catch slow queries during load testing (before production)
   - Better: Set up slow query alert (> 1s) to catch early

---

## Escalation Path

- **Level 1** (This runbook): SRE on-call, should resolve in 5 minutes
- **Level 2** (Unresolved): Page backend engineering lead
- **Level 3** (> 15 min): Page engineering manager + VP of Engineering

---

## Automation (Intelligent Runbook)

```yaml
# This runbook can be automated with Harness/PagerDuty/Opsgenie

trigger:
  - alert: cpu_high
    condition: cpu > 95%
    
auto_response:
  - send_slack: "High CPU on {hostname}. Investigating..."
  - run_script: /ops/check_processes.sh
  - if: [ cpu_still_high ]
    - run_command: systemctl restart api-server
    - wait: 120s  # 2 minutes
  - if: [ cpu_normal ]
    - send_slack: "✅ Incident resolved. CPU restored."
    - create_pagerduty_incident: "High CPU - Resolved"
  - if: [ cpu_still_high ]
    - page_oncall_engineer: "CPU high after restart"
    - send_slack: "@oncall, restart failed. Manual intervention needed."
```

---

## Testing This Runbook

Every quarter, simulate this incident:

```bash
# In staging environment
stress-ng --cpu 4 --timeout 5m  # Spike CPU for 5 minutes

# Test runbook steps
# 1. Alert fires
# 2. Follow runbook steps
# 3. Document actual vs. expected times
# 4. Update runbook if timing changes
```
```

### 10.2.2 Incident Response Playbook

```markdown
# Incident Response Playbook

## Response Levels

| Level | Definition | Response Time | Who |
|-------|-----------|---|---|
| **P1** | Production down, all users affected | < 5 min | VP Eng + all leads |
| **P2** | Service degraded, some users affected | < 15 min | On-call eng + team lead |
| **P3** | Non-critical issue, workaround exists | < 1 hour | Team |
| **P4** | Minor bug, no workaround | < 1 day | Backlog |

## P1 Response Steps

```
T+0min: Alert fires
├─ PagerDuty pages on-call engineer
├─ Auto-post to #incidents Slack channel
└─ Create incident in incident management system

T+1min: On-call acknowledges
├─ Accept PagerDuty alert (stops escalation)
├─ Join war room (Google Meet/Zoom link in alert)
├─ Post status: "Investigating cause"
└─ If cause unknown in 5 min → Page manager

T+5min: Investigation continues
├─ Check monitoring dashboards
├─ Review error logs
├─ Check recent deployments (rollback candidate?)
└─ If no clear cause → Escalate

T+15min: Decision point
├─ If clear mitigation: Apply fix
├─ If rollback safe: Rollback last deployment
├─ If neither: Scale up + notify customers

T+30min: All-hands meeting (if still ongoing)
├─ Engineering manager
├─ Product/ops lead
├─ Customer success (what to tell customers)
└─ Plan next steps
```

## War Room Discipline

```
🚨 Rules for effective incident response:

1. Single leader (incident commander)
   - Makes decisions
   - Updates status
   - Nothing else

2. Separate roles:
   - Investigator (finds root cause)
   - Mitigator (applies fix)
   - Communicator (updates customers/team)

3. Focus on MTTR (Mean Time To Resolution)
   - Restore service FIRST (5 min)
   - Root cause analysis SECOND (post-incident)
   - Do NOT debug in production under pressure

4. Avoid blame
   - "What failed?" not "Who failed?"
   - Mistakes happen at scale. Learn.

5. Document everything
   - What time each action taken
   - Who did what
   - Decisions made and why
   - This data feeds post-mortem
```

## Post-Incident Review (Blameless)

Template (schedule within 24 hours):

```markdown
# Incident Review: [Service] Down

**Date**: 2025-01-24
**Duration**: 47 minutes
**MTTR**: 47 minutes
**MTBF** (Mean Time Between Failures): 15 days
**Severity**: P1

## Timeline

| Time | Event |
|------|-------|
| 14:30 | Alert fires: API server CPU high |
| 14:31 | Engineer paged |
| 14:33 | Restarted service |
| 14:45 | Service operational |
| 15:17 | Root cause identified |

## Root Cause

N+1 query problem in product listing endpoint.
When fetching 50 products, query joined users table for each product (50 queries instead of 1).
Under normal load: unnoticed.
Under peak load: CPU spike.

## Why It Happened

1. Feature: Added "product owner" field to listing
2. Implementation: Lazy loaded from separate query
3. Bug: Didn't use eager loading / batch loading
4. Missed in code review: Reviewer unfamiliar with ORM patterns
5. No load test caught it: Load tests used small product set

## Prevention

- [ ] Add eager loading in ORM (developer training)
- [ ] Code review checklist: "Check for N+1 queries"
- [ ] Load test with realistic data (1M products)
- [ ] Slow query alert (queries > 500ms)

## Improvement Ideas

1. **Short term** (this sprint)
   - Fix query with eager loading
   - Add test case for this scenario
   - Document ORM best practices in wiki

2. **Medium term** (next quarter)
   - Automated slow query detection in CI
   - Load testing as part of release criteria

3. **Long term**
   - ML model to detect N+1 patterns in code review
   - Query optimization tool in IDE

## Action Items

| Task | Owner | Deadline |
|------|-------|----------|
| Fix eager loading | @alice | This sprint |
| Add N+1 detector to CI | @bob | 2 weeks |
| Load test infrastructure | @charlie | Next month |
```
```

---

## 10.3 USER DOCUMENTATION

### 10.3.1 User Guide Structure

```markdown
# Orders App - User Guide

## Table of Contents
1. Getting Started
2. Creating Your First Order
3. Managing Orders
4. Troubleshooting
5. FAQ

## 1. Getting Started

### Prerequisites
- Email address (for login)
- Valid payment method (credit/debit card)

### Your First Login
1. Go to https://app.company.com
2. Click "Sign Up"
3. Enter email and create password
4. Verify email (click link in inbox)
5. You're in!

### Resetting Your Password
Forgot password?
1. Click "Forgot Password" on login page
2. Enter your email
3. Click link in reset email
4. Create new password
5. Log back in

## 2. Creating Your First Order

### Step 1: Search for Products
1. Click "Products" in menu
2. Search by name or category
3. Read product details, reviews, images

### Step 2: Add to Cart
1. Click product
2. Select quantity
3. Click "Add to Cart"
4. (Optional) Continue shopping

### Step 3: Checkout
1. Click cart icon
2. Review items (can edit quantities)
3. Choose shipping method:
   - **Standard** (5-7 days): Free
   - **Express** (2-3 days): $5.99
   - **Overnight** (next day): $14.99
4. Enter shipping address
5. Choose payment method
6. Review order summary
7. Click "Place Order"

### Step 4: Confirmation
You'll see:
- Order confirmation on screen
- Confirmation email in inbox
- Can download invoice as PDF

## 3. Managing Orders

### Viewing Your Orders
1. Click "My Orders" in menu
2. See list of all past orders
3. Orders show status, date, total

### Order Statuses
- **Pending**: Order received, processing payment
- **Confirmed**: Payment successful, preparing to ship
- **Shipped**: Package is on the way (tracking # provided)
- **Delivered**: Package arrived
- **Cancelled**: Order was cancelled

### Tracking Your Package
1. Go to "My Orders"
2. Click order
3. Find "Tracking Number"
4. Paste in carrier website (FedEx, UPS, etc)

### Modifying an Order
**Can only modify before it ships.**
1. Go to "My Orders"
2. Click order (if status is "Pending" or "Confirmed")
3. Click "Modify Order"
4. Change quantity or address
5. Click "Save Changes"

### Cancelling an Order
**Can only cancel before it ships.**
1. Go to "My Orders"
2. Click order
3. Click "Cancel Order"
4. Confirm cancellation
5. Refund processed within 3-5 business days

## 4. Troubleshooting

### Q: "Payment declined" error
**Possible causes and solutions:**
- Card expired: Update payment method
- Insufficient funds: Check account balance
- Card flagged by bank: Contact your bank
- Address mismatch: Ensure billing address matches card
- Try again: Sometimes temporary issue

### Q: Order not showing in "My Orders"
**Possible causes:**
- Just ordered (may take 2-3 minutes to appear)
- Logged in with different email
- Used guest checkout (orders not saved)

**Solution:** Check email inbox for confirmation. Confirmation email has order #.

### Q: Forgot email address associated with account
Log in with your alternate email address or phone number.

### Q: Need to change email address
1. Account Settings
2. Click "Email Address"
3. Enter new email
4. Verify with link sent to new email

## 5. FAQ

**Q: When will I receive my order?**
A: Delivery time depends on shipping method selected:
- Standard: 5-7 business days
- Express: 2-3 business days
- Overnight: Next business day
(Not including weekends/holidays)

**Q: Can I change my shipping address after ordering?**
A: Only if order hasn't shipped yet. Go to "My Orders" → order → "Modify Order".

**Q: What payment methods do you accept?**
A: 
- Visa, Mastercard, American Express
- Apple Pay, Google Pay
- PayPal

**Q: Is my payment secure?**
A: Yes! We use industry-standard encryption (SSL/TLS).
Payment info handled by Stripe (PCI Level 1 certified).
We never store your full credit card number.

**Q: Do you ship internationally?**
A: Currently US only. International shipping coming Q2 2025.

**Q: Can I return items?**
A: Yes! 30-day return window.
1. Go to "My Orders"
2. Click order
3. Click "Return Items"
4. Follow steps
5. Print return label
6. Mail back (free shipping)
7. Refund processed within 7 days of receipt
```

### 10.3.2 Admin Documentation

```markdown
# Admin Guide

## Admin Dashboard

**Access**: Only users with "admin" role can access.
URL: https://app.company.com/admin

### User Management

#### View All Users
Dashboard → Users
- Search by email
- Filter by status (active, suspended)
- Sort by join date, last login

#### Suspend User Account
(Use if user violates terms)
1. Find user
2. Click user row
3. Click "Suspend Account"
4. (Optional) Enter reason
5. Confirm
- User cannot login
- All active sessions end
- User can appeal suspension via support

#### Delete User Account (GDPR)
(Fully removes user data)
1. Find user
2. Click user row
3. Click "Delete Account"
4. (REQUIRED) Enter reason for audit
5. Confirm
- User deleted
- All orders kept (for legal/tax reasons)
- Payments reconciled

#### Reset User Password
(For user who forgot password)
1. Find user
2. Click "Reset Password"
3. Temporary password sent to their email
4. User must change on first login

### Order Management

#### View Orders
Dashboard → Orders
- Filter by status, date range, user
- Search by order #
- See gross revenue, refunds, net revenue

#### Refund Order
(If customer dissatisfied)
1. Find order
2. Click order
3. If status "Delivered" and within 30 days:
4. Click "Process Refund"
5. Reason (required for audit)
6. Confirm
- Refund processed to original payment method
- Takes 3-5 business days
- Email sent to customer

#### Override Price
(Emergency situations only)
1. Create order manually
2. Set custom price
3. **AUDIT**: Log reason and approver
4. Never for routine orders (use coupons instead)

### Promotions & Coupons

#### Create Coupon
Dashboard → Coupons → Create
- Code: Unique code (e.g., SAVE20)
- Type: Percentage (20% off) or fixed ($10 off)
- Max uses: Limit per customer
- Expiry: When does coupon expire
- Min purchase: Minimum order size

#### Monitor Coupon Performance
Dashboard → Coupons
- Redemptions: How many times used
- Revenue impact: Lost revenue from discount
- Trending: Which coupons most popular

#### Disable Coupon
If experiencing fraud/abuse:
1. Find coupon
2. Click "Disable"
3. No new orders can use this code
4. (Optional) Refund existing abuse

### Reports

#### Revenue Report
Dashboard → Reports → Revenue
- Daily/weekly/monthly
- By product, category, shipping method
- YoY growth comparison
- Export to CSV/Excel

#### Customer Cohort Analysis
- Retention: % of customers who return
- LTV (Lifetime Value): Average $ per customer
- Churn: % who don't return
- Used for marketing strategy

### System Settings

#### Email Templates
Settings → Email Templates
- Order confirmation
- Shipping notification
- Refund confirmation
- Custom branding (logo, colors)

#### Tax Settings
Settings → Taxes
- Sales tax by state
- Automatically applied at checkout
- Compliance: Tax reports for filing

#### Payment Settings
Settings → Payments
- Stripe API keys (production/staging)
- Test mode: Use test cards for QA
- Webhooks: What notifications to receive
```

---

## 10.4 DEVELOPER DOCUMENTATION

### 10.4.1 Development Workflow

```markdown
# Developer Setup & Workflow

## Prerequisites
- Node.js 18+ (check: `node --version`)
- Docker & Docker Compose
- Git
- Text editor (VS Code recommended)

## Project Setup (First Time)

```bash
# 1. Clone repository
git clone https://github.com/company/orders-app.git
cd orders-app

# 2. Install dependencies
npm install

# 3. Copy environment variables
cp .env.example .env

# 4. Start services (database, redis, etc)
docker-compose up -d

# 5. Initialize database (create tables)
npm run db:migrate

# 6. Seed test data (optional)
npm run db:seed

# 7. Start development server
npm run dev

# Application is now at http://localhost:3000
```

## Daily Workflow

```bash
# Before starting work
git pull origin main
npm install  # In case dependencies changed

# Create feature branch
git checkout -b feature/user-stories-123

# Make changes
# (Edit files, run tests frequently)
npm test  # Run unit tests
npm run test:e2e  # Run E2E tests
npm run lint  # Check code style

# Commit frequently
git add .
git commit -m "feat: add order status tracking"

# Before pushing
npm run build  # Verify production build works

# Push to GitHub
git push origin feature/user-stories-123

# Create Pull Request on GitHub
# Link to issue: "Closes #123"
# Request review from team
```

## Testing

```bash
# Unit tests
npm test

# Unit tests with coverage
npm test:coverage

# Integration tests (with real database)
npm run test:integration

# E2E tests (with real UI)
npm run test:e2e

# E2E with headed browser (see what's happening)
npm run test:e2e -- --headed

# Performance testing
npm run test:performance
```

## Code Review Checklist

Before submitting PR:
- [ ] Tests pass locally (`npm test`)
- [ ] No console.logs in production code
- [ ] No hardcoded secrets (use env vars)
- [ ] Types are correct (TS strict mode)
- [ ] Error handling implemented
- [ ] Accessibility tested (Tab, Enter, Escape)
- [ ] Performance acceptable (no N+1 queries)

## Debugging

```bash
# Debug with Chrome DevTools
node --inspect-brk ./node_modules/.bin/jest --runInBand

# Debug API requests
# In browser: F12 → Network tab

# View API logs
tail -f logs/api.log

# Monitor database
docker exec orders-db psql -U user -d orders_dev -c "SELECT * FROM orders;"

# Monitor cache (Redis)
docker exec orders-redis redis-cli KEYS '*'
```

## Database Migrations

```bash
# Create new migration
npx knex migrate:make add_user_roles

# Edit migration in migrations/
# Contains: up() and down() functions

# Run migrations
npm run db:migrate

# Rollback last migration (emergency only)
npm run db:rollback

# Check migration status
npm run db:status
```

## Code Style & Linting

```bash
# Auto-fix style issues
npm run lint:fix

# Check types
npm run type-check

# Format code
npm run format
```

## Deployment

```bash
# Deploy to staging
git push origin feature/branch  # Creates PR
# After PR approved: merge to staging branch
# GitHub Actions auto-deploys to staging

# Deploy to production
# Only from main branch, only after testing
git checkout main
git pull
git merge staging --no-ff -m "Release v1.2.3"
git tag v1.2.3
git push origin main v1.2.3
# GitHub Actions auto-deploys to production
```
```

### 10.4.2 Code Contribution Guidelines

```markdown
# Contributing

## Commit Message Format

Format:
```
<type>(<scope>): <subject>

<body>

<footer>
```

Example:
```
feat(orders): add order status tracking

Users can now view order status changes in real-time.
Added WebSocket support for push updates.

Closes #456
```

Types:
- **feat**: New feature
- **fix**: Bug fix
- **perf**: Performance improvement
- **refactor**: Code restructure (no behavior change)
- **docs**: Documentation only
- **test**: Test changes only
- **chore**: Dependency updates, cleanup

## Pull Request Template

```markdown
## Description
What does this PR do? (2-3 sentences)

## Testing
How did you test this? List test commands.

## Related Issues
Closes #123

## Screenshots
(If UI change: add before/after)

## Deployment Notes
Any special deployment steps? Database migrations?
Breaking changes?
```

## Code Review Process

1. **Author** submits PR
2. **Reviewer** assigns task
3. **Reviewer** comments on code
4. **Author** addresses comments
5. **Reviewer** approves
6. **Author** merges (only after approval)
7. **CI/CD** automatically deploys

**Review SLA**: 24 hours

## Breaking Changes Policy

If your change breaks existing API/behavior:
1. Add migration path (old way still works for deprecation period)
2. Document in CHANGELOG
3. Bump major version
4. Notify users via email/blog post

Example:
```
// Old: GET /orders?page=1
// New: GET /orders?offset=0&limit=20

// During migration period, both work:
if (query.page) {
  // Convert old pagination format
  offset = (query.page - 1) * 20;
  limit = 20;
}
```
```

---

## 10.5 DOCUMENTATION GOVERNANCE

### 10.5.1 Documentation Lifecycle

```
Create → Review → Publish → Maintain → Archive
```

**Create (Week 1)**
- Developer writes documentation with code
- Format: Markdown, OpenAPI, or code comments
- Completeness: Include examples, error cases

**Review (Week 1)**
- Tech lead reviews for accuracy
- Reviewer checks: "Would someone new understand this?"
- Approval gate: Must pass review

**Publish (Week 1-2)**
- Merge to main branch
- Deploy to documentation site
- Link from relevant pages

**Maintain (Ongoing)**
- Update with code changes
- Fix broken links/examples monthly
- Audit outdated content quarterly

**Archive (When deprecated)**
- Move to /deprecated/ folder
- Keep for historical reference
- Update all links to new documentation

### 10.5.2 Documentation Checklist (QA)

Before considering documentation "done":

**Completeness:**
- [ ] Covers all happy paths and error cases
- [ ] Includes working code examples
- [ ] Has clear prerequisites
- [ ] Documents all parameters/fields
- [ ] Includes troubleshooting section

**Clarity:**
- [ ] No jargon (or explained if necessary)
- [ ] Logical flow (prerequisites → basic → advanced)
- [ ] Consistent terminology
- [ ] Tone matches audience

**Accuracy:**
- [ ] Examples actually run without errors
- [ ] Screenshots current (not outdated)
- [ ] Links not broken
- [ ] Tested in actual environment

**Accessibility:**
- [ ] Readable font size (not too small)
- [ ] High contrast colors
- [ ] Images have alt-text
- [ ] Proper heading hierarchy (H1, H2, H3)
- [ ] Keyboard navigable

**Findability:**
- [ ] Linked from related docs
- [ ] Has search keywords
- [ ] Table of contents (if > 2000 words)
- [ ] Appears in navigation menu
- [ ] SEO-friendly title

### 10.5.3 Documentation Tools

```yaml
# Recommended tech stack

# Hosting
- Docs site: Vercel (fast, automatic deployments)
- Backup: GitHub Pages (free, built-in)

# Writing
- Format: Markdown (portable, versionable)
- Editor: VS Code + Markdown preview
- Diagrams: Mermaid (renders in GitHub/web)

# Collaboration
- Version control: GitHub (alongside code)
- Reviews: Pull request comments
- Issues: Track doc improvements

# Reference docs (code-generated)
- API docs: Redoc (from OpenAPI spec)
- SDK docs: Typedoc (from TypeScript)

# Search
- Engine: Algolia (fast, typo-tolerant)
- Index: Update on each deployment

# Analytics
- Track: Which docs most viewed
- Feedback: "Was this helpful?" buttons
- Alerts: Broken links detected weekly

# Automation
- CI: Validate links, spell-check, formatting
- Deployment: Auto-publish on merge to main
- Notifications: Slack alert when docs updated
```

### 10.5.4 Documentation SLA (Service Level Agreement)

```
Update frequency by document type:

API Documentation
- Lag: 0 days (auto-generated from OpenAPI spec)
- Review: Must pass API team review

Architecture Documentation
- Lag: 1 week (document after code merged)
- Review: Tech lead approval
- Audit: Quarterly (ensure still accurate)

Runbooks
- Lag: Same day (must test in production)
- Review: SRE team + author who wrote code
- Audit: Test quarterly (simulate incident)

User Documentation
- Lag: Release day (ship with feature)
- Review: Product + UX before release
- Audit: Monthly (check for broken links)

Blog Posts / Case Studies
- Lag: 2 weeks (after feature stable)
- Review: Exec approval if external
- Audit: Annual (check for outdated claims)

Support FAQs
- Lag: Real-time (update as support tickets come in)
- Review: Support manager approval
- Audit: Monthly (consolidate related FAQs)
```

---

## 10.6 DOCUMENTATION QUALITY METRICS

### How to Measure Documentation Quality

```
Metric: Average time to resolve using docs

Hypothesis: Better docs = faster resolution
Measure: When support ticket filed, log "time to resolution using docs"

Good docs:   < 2 hours
Okay docs:   2-8 hours
Bad docs:    > 8 hours (person gives up, calls support)
No docs:     Always > 1 hour (no docs available)

Action: If > 8 hours average, docs need improvement.
Target: 80% of users self-serve using docs (no support contact).
```

```
Metric: Search analytics

Track what users search for:
- "How to reset password" (frequently searched?)
  → If high: docs didn't answer, or hard to find
  → Add FAQ section, improve search keywords
  
- "undefined is not a function" (error doc needed?)
  → If high: users hitting errors
  → Add troubleshooting section, improve error messages

Action: Top 10 searches → docs need attention
```

```
Metric: Documentation coverage

Formula: (Documented features / Total features) × 100

Target: 100% coverage
- Every API endpoint has docs
- Every admin feature documented
- Every error code explained
- Every configuration option documented

Tool: Generate coverage report
```

---

## 10.7 CRITICAL DOCUMENTATION RULES

**These are non-negotiable:**

1. **Every feature ships with documentation**
   - Code review blocks if docs missing
   - Deploy blocks if docs not reviewed

2. **Examples must be runnable**
   - Copy/paste from docs should work
   - Test examples in CI pipeline
   - Update examples when code changes

3. **Errors documented**
   - Every error code has explanation
   - Links to troubleshooting
   - What to do if it happens

4. **No dead links**
   - CI pipeline checks for broken links weekly
   - Alert immediately if found
   - Fix within 24 hours

5. **No outdated content**
   - Archive old docs (don't delete)
   - Add "This is outdated" banners
   - Date all docs ("Last updated: Jan 2025")

6. **Accessible to all**
   - No jargon without explanation
   - Screenshots with alt-text
   - High contrast colors
   - Mobile-friendly

---

## 10.8 DELIVERABLES & SIGN-OFF

### Phase 10 Deliverables

**Gate 1: Documentation Architecture Review**
- [ ] 4-layer structure defined
- [ ] Owners assigned (who maintains each layer)
- [ ] Update cadence defined
- [ ] Tools selected
- **Approval**: Tech lead + product

**Gate 2: API Documentation**
- [ ] OpenAPI 3.1 spec complete
- [ ] All endpoints documented
- [ ] All error codes documented
- [ ] Examples provided for each endpoint
- [ ] Auto-deployed to Redoc/SwaggerUI
- **Approval**: API team

**Gate 3: Architecture Documentation**
- [ ] C4 model diagrams (all 4 levels)
- [ ] Data flow diagrams
- [ ] Integration points documented
- [ ] Technology choices justified
- **Approval**: Tech lead

**Gate 4: Operational Documentation**
- [ ] P1/P2/P3 runbooks created
- [ ] Incident response playbook
- [ ] Escalation paths defined
- [ ] Post-incident template
- [ ] Tested in staging (runbook actually works)
- **Approval**: SRE + on-call engineer

**Gate 5: User Documentation**
- [ ] User guide (all features)
- [ ] Admin guide (all admin features)
- [ ] Getting started guide
- [ ] Troubleshooting section
- [ ] FAQ section
- **Approval**: Product + UX

**Gate 6: Developer Documentation**
- [ ] Setup instructions (tested fresh)
- [ ] Development workflow documented
- [ ] Testing guide (all test types)
- [ ] Deployment process documented
- [ ] Code contribution guidelines
- [ ] Database schema documented
- **Approval**: Engineering lead

**Gate 7: Documentation Governance**
- [ ] Style guide created
- [ ] Review process defined (SLA: 24 hours)
- [ ] Update calendar established
- [ ] Metrics dashboard set up
- [ ] Automated checks in CI (links, spelling, formatting)
- **Approval**: Documentation owner + manager

### Sign-Off Criteria

Before shipping:

```
✅ Code + Documentation are reviewed together
   - No documentation-only PRs (added after code merge)
   - Runbooks tested in staging/prod-like environment
   - Examples verified to work

✅ Documentation is discoverable
   - Linked from relevant pages
   - Appears in search
   - Included in navigation menu

✅ Documentation is maintained
   - Owner assigned (who updates if it breaks)
   - Update frequency clear
   - Audit schedule set

✅ Quality gates met
   - No broken links
   - No spelling errors
   - No outdated screenshots
   - Accessibility checks passed

✅ Audience can actually use it
   - User study: 5 users follow guide end-to-end
   - 80%+ complete task without asking questions
   - Average time to understand < 5 minutes
```

---

**Documentation is not a nice-to-have. It's critical infrastructure.**

Teams with excellent documentation move 3x faster.
Teams with poor documentation spiral into chaos.

Invest now, or pay the price later.
