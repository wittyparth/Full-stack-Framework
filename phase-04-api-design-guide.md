# PHASE 4: API DESIGN (CONTRACT-FIRST)

## Why Contract-First Matters

Before writing a single line of backend code, you define the API contract—the exact shape of requests, responses, errors, and authentication. This prevents miscommunication between frontend and backend teams, enables parallel development, and creates a single source of truth. 

> **A 20-year veteran's rule:** The worst technical debt comes from API misalignment. Fix it here.

The specification **IS** the contract. Your API spec lives in `openapi.yaml` at the root of your backend repo. Every endpoint, every field, every error response is documented before code is written.

### Why Swagger/OpenAPI

- **Machine-readable format** - Tools generate code, tests, docs automatically
- **Industry standard** - Every API team you join uses it
- **Generates interactive documentation instantly**
- **Enables API validation at runtime** - Reject malformed requests
- **Frontend teams can mock the API** - Generate fake endpoints from spec
- **Contract testing** - Verify backend and frontend match the spec

---

## 4.1 API CONTRACT DEFINITION

### 4.1.1 OpenAPI/Swagger Specification

#### Minimal OpenAPI Spec Structure

```yaml
openapi: 3.0.0
info:
  title: E-Commerce API
  version: 1.0.0
  description: Order processing, inventory, payments
  contact:
    name: Backend Team
    email: backend@company.com

servers:
  - url: https://api.company.com/v1
    description: Production
  - url: https://staging-api.company.com/v1
    description: Staging
  - url: http://localhost:3000/v1
    description: Development (local)

components:
  schemas:
    Order:
      type: object
      required: [id, customer_id, total, status]
      properties:
        id:
          type: string
          format: uuid
          description: Unique order identifier
        customer_id:
          type: string
          format: uuid
        total:
          type: number
          format: double
          minimum: 0.01
          description: Order total in USD
        status:
          type: string
          enum: [pending, processing, shipped, delivered, cancelled]
        items:
          type: array
          items:
            $ref: '#/components/schemas/OrderItem'
        created_at:
          type: string
          format: date-time
    
    OrderItem:
      type: object
      required: [sku, quantity, unit_price]
      properties:
        sku:
          type: string
          minLength: 3
          maxLength: 20
        quantity:
          type: integer
          minimum: 1
        unit_price:
          type: number
          format: double
          minimum: 0.01
    
    Error:
      type: object
      required: [code, message]
      properties:
        code:
          type: string
          enum: [INVALID_INPUT, NOT_FOUND, UNAUTHORIZED, FORBIDDEN, RATE_LIMITED]
        message:
          type: string
          description: Human-readable error message
        details:
          type: object
          description: Additional context (validation errors, etc.)
  
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: JWT token issued by /auth/login endpoint

paths:
  /orders:
    get:
      summary: List orders for authenticated user
      tags: [Orders]
      security:
        - bearerAuth: []
      parameters:
        - name: status
          in: query
          schema:
            type: string
            enum: [pending, processing, shipped, delivered, cancelled]
          description: Filter by order status
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            minimum: 1
            maximum: 100
        - name: offset
          in: query
          schema:
            type: integer
            default: 0
            minimum: 0
      responses:
        '200':
          description: Orders retrieved successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/Order'
                  total:
                    type: integer
                  limit:
                    type: integer
                  offset:
                    type: integer
        '401':
          description: Unauthorized (missing or invalid JWT)
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
        '429':
          description: Rate limit exceeded
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
    
    post:
      summary: Create new order
      tags: [Orders]
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [customer_id, items, shipping_address]
              properties:
                items:
                  type: array
                  minItems: 1
                  items:
                    $ref: '#/components/schemas/OrderItem'
                shipping_address:
                  $ref: '#/components/schemas/Address'
                payment_method_id:
                  type: string
                  description: Stripe payment method ID
      responses:
        '201':
          description: Order created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '400':
          description: Invalid request body
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
        '409':
          description: Conflict (e.g., insufficient inventory)
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
  
  /orders/{order_id}:
    get:
      summary: Get order details
      tags: [Orders]
      security:
        - bearerAuth: []
      parameters:
        - name: order_id
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: Order details
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '404':
          description: Order not found
  
  /auth/login:
    post:
      summary: Authenticate user, return JWT token
      tags: [Auth]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [email, password]
              properties:
                email:
                  type: string
                  format: email
                password:
                  type: string
                  minLength: 8
      responses:
        '200':
          description: Authentication successful
          content:
            application/json:
              schema:
                type: object
                properties:
                  access_token:
                    type: string
                    description: JWT token (15-minute expiry)
                  refresh_token:
                    type: string
                    description: Refresh token (HTTP-only cookie)
                  user:
                    type: object
                    properties:
                      id:
                        type: string
                        format: uuid
                      email:
                        type: string
                      roles:
                        type: array
                        items:
                          type: string
                          enum: [user, admin, moderator]
        '401':
          description: Invalid email or password
```

#### Key Principles in This Spec

1. **Required fields explicitly stated** - `required: [id, customer_id, total, status]` means backend MUST include these, frontend MUST expect them
2. **Constraints documented** - `minimum: 0.01`, `minLength: 3`, `maxItems: 100` prevent invalid data at the source
3. **Multiple response codes** - 200 (success), 400 (bad input), 401 (auth), 404 (not found), 409 (conflict), 429 (rate limit)
4. **Reusable components** - `$ref: '#/components/schemas/Order'` prevents duplication, single source of truth
5. **Security defined upfront** - `bearerAuth` shows which endpoints need authentication

#### Maintaining the Spec

- Spec lives in version control (git), reviewed in pull requests
- Any API change requires spec change first (contract-first)
- Frontend team reviews spec changes before backend starts coding
- Generate server stubs from spec (scaffold routes, validation)
- Generate client SDKs from spec (frontend uses generated TypeScript)

---

### 4.1.2 Endpoint URL Structure Design

**Rule:** URLs represent **RESOURCES**, not **ACTIONS**.

#### ❌ Bad (action-based)

```
GET /api/getOrders
POST /api/createOrder
PUT /api/updateOrder/123
DELETE /api/cancelOrder/123
GET /api/getOrderDetails/123
```

#### ✅ Good (resource-based)

```
GET    /api/v1/orders           # List all orders
POST   /api/v1/orders           # Create new order
GET    /api/v1/orders/{id}      # Get specific order
PUT    /api/v1/orders/{id}      # Update order
DELETE /api/v1/orders/{id}      # Delete order
GET    /api/v1/orders/{id}/items # Get order items (sub-resource)
```

#### URL Design Rules

**1. Use plural nouns for collections**

```
/api/v1/orders (not /api/v1/order)
/api/v1/customers/{id}/orders (nested resource)
```

**2. Use HTTP method to indicate action**

- `GET` = retrieve (safe, idempotent)
- `POST` = create (not idempotent, creates new resource)
- `PUT` = replace entire resource (idempotent)
- `PATCH` = partial update (idempotent)
- `DELETE` = remove (idempotent after first call)

**3. Avoid deeply nested URLs (max 2 levels)**

```
✅ /api/v1/orders/{id}/items (order items)
✅ /api/v1/customers/{id}/orders (customer orders)
❌ /api/v1/customers/{cid}/orders/{oid}/items/{iid}/tracking (too deep)
```

Deep nesting creates brittle APIs. Instead:
```
/api/v1/order-items/{id} (query by parent: ?order_id=123)
```

**4. Query parameters for filtering, not path segments**

```
GET /api/v1/orders?status=shipped&customer_id=123&created_after=2024-01-01

(not: /api/v1/orders/shipped/customer/123/created-after/2024-01-01)
```

**5. Consistent naming across resources**

- Always use `snake_case` in URLs: `/api/v1/order-items`, not `/api/v1/OrderItems`
- Always use `snake_case` in response fields: `customer_id`, not `customerId`
- This is JSON convention, consistent across the industry

#### URL Structure Examples

**Orders API:**
```
├─ GET    /api/v1/orders                    → List (with filtering)
├─ POST   /api/v1/orders                    → Create
├─ GET    /api/v1/orders/{order_id}         → Get one
├─ PATCH  /api/v1/orders/{order_id}         → Update fields
├─ DELETE /api/v1/orders/{order_id}         → Cancel
├─ POST   /api/v1/orders/{order_id}/ship    → Ship order (action on resource)
└─ GET    /api/v1/orders/{order_id}/items   → Get items in order
```

**Inventory API:**
```
├─ GET   /api/v1/inventory?sku=ABC123              → Check stock
├─ GET   /api/v1/inventory/{sku}                   → Get full inventory record
├─ PATCH /api/v1/inventory/{sku}                   → Update stock level
└─ GET   /api/v1/inventory/report?group_by=category
```

**Customers API:**
```
├─ GET  /api/v1/customers                     → List
├─ POST /api/v1/customers                     → Register
├─ GET  /api/v1/customers/{customer_id}       → Profile
└─ GET  /api/v1/customers/{customer_id}/orders → Their orders
```

**Auth API:**
```
├─ POST /api/v1/auth/login          → Login
├─ POST /api/v1/auth/logout         → Logout
├─ POST /api/v1/auth/refresh        → Refresh JWT
└─ POST /api/v1/auth/reset-password → Password reset
```

---

### 4.1.3 HTTP Methods & Status Codes

Critical mapping (this determines if your API scales with team knowledge):

#### GET - Safe (no side effects), Idempotent (same result every time)
```
├─ 200 OK          - Resource found
├─ 404 Not Found   - Resource doesn't exist
├─ 401 Unauthorized - Need authentication
├─ 403 Forbidden   - Authenticated but no permission
└─ 410 Gone        - Resource deleted permanently (vs 404 which might exist)
```

#### POST - NOT safe, NOT idempotent (creates new resource each time)
```
├─ 201 Created          - Resource created, include Location header with URL
├─ 202 Accepted         - Request accepted, processing async (batch jobs)
├─ 400 Bad Request      - Invalid input (validation failed)
├─ 409 Conflict         - Can't create (duplicate, constraint violation)
└─ 429 Too Many Requests - Rate limited
```

#### PUT - NOT safe, IS idempotent (replace entire resource)
```
├─ 200 OK          - Resource updated
├─ 201 Created     - Resource didn't exist, created it
├─ 400 Bad Request - Invalid input
├─ 404 Not Found   - Resource doesn't exist
└─ 409 Conflict    - Can't update (concurrency issue)
```

#### PATCH - NOT safe, IS idempotent (partial update)
```
├─ 200 OK          - Resource partially updated
├─ 400 Bad Request - Invalid input
└─ 404 Not Found   - Resource doesn't exist
```

#### DELETE - NOT safe, IS idempotent (first time removes, subsequent calls do nothing)
```
├─ 204 No Content - Deleted successfully (no response body)
├─ 404 Not Found  - Already deleted or never existed
└─ 409 Conflict   - Can't delete (dependent resources exist)
```

#### Why This Matters

A browser requests `GET /api/orders/123` 3 times (network hiccup). Since GET is idempotent, it's safe to retry. Response is identical each time.

A browser POSTs an order create, network dies, browser retries. If you don't handle idempotency, order is created twice. 

**Solution:** Use idempotency keys (request ID in header, deduplicate on backend).

---

### 4.1.4 Request/Response Schemas

Contracts define the exact shape of data flowing in and out.

#### Response Format (Always Consistent)

```json
{
  "status": "success|error",
  "data": { /* actual payload */ },
  "error": { /* if status === error */ },
  "metadata": { /* pagination, timing, etc. */ }
}
```

#### Example Successful GET

```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "customer_id": "660e8400-e29b-41d4-a716-446655440000",
    "total": 149.99,
    "status": "shipped",
    "items": [
      {
        "sku": "LAPTOP-001",
        "quantity": 1,
        "unit_price": 149.99
      }
    ],
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-16T14:22:00Z"
  },
  "metadata": {
    "request_id": "req_123abc",
    "timestamp": "2024-01-16T14:22:15Z"
  }
}
```

#### Error Response (Same Structure)

```json
{
  "status": "error",
  "data": null,
  "error": {
    "code": "INSUFFICIENT_INVENTORY",
    "message": "Not enough stock for SKU LAPTOP-001",
    "details": {
      "sku": "LAPTOP-001",
      "requested": 5,
      "available": 2
    }
  },
  "metadata": {
    "request_id": "req_123abc",
    "timestamp": "2024-01-16T14:22:15Z"
  }
}
```

#### Key Principles

1. **Consistent envelope** - Every response has same structure (data/error/metadata)
2. **Request ID in response** - Trace every request for debugging: `X-Request-ID` header
3. **Timestamps ISO 8601** - `2024-01-16T14:22:15Z` (UTC, machine-parseable)
4. **Null never in responses** - If field missing, omit it entirely (don't send null)
5. **Error codes are enums** - `INSUFFICIENT_INVENTORY` not `not_enough_stock` (consistent with frontend error handling)

---

### 4.1.5 Error Response Format & Catalog

Errors are part of your contract. Document every error code that can occur.

#### Error Code Taxonomy

**4xx = Client error** (frontend/client app did something wrong)
```
├─ 400 = Bad Request (validation failed)
├─ 401 = Unauthorized (not authenticated)
├─ 403 = Forbidden (authenticated but no permission)
├─ 404 = Not Found (resource doesn't exist)
├─ 409 = Conflict (state violation, can't complete request)
└─ 429 = Too Many Requests (rate limited)
```

**5xx = Server error** (something went wrong on backend)
```
├─ 500 = Internal Server Error (unexpected crash)
├─ 503 = Service Unavailable (temporary outage)
└─ 504 = Gateway Timeout (backend took too long)
```

#### Error Code Catalog (in `openapi.yaml`)

```yaml
components:
  schemas:
    ErrorCode:
      type: string
      enum:
        # 400 - Bad Request / Validation
        - INVALID_INPUT
        - MISSING_REQUIRED_FIELD
        - INVALID_EMAIL_FORMAT
        - INVALID_PHONE_FORMAT
        - INVALID_DATE_FORMAT
        - VALUE_OUT_OF_RANGE
        
        # 401 - Authentication
        - UNAUTHORIZED
        - INVALID_CREDENTIALS
        - TOKEN_EXPIRED
        - TOKEN_INVALID
        - TOKEN_MISSING
        
        # 403 - Authorization
        - FORBIDDEN
        - INSUFFICIENT_PERMISSIONS
        - RESOURCE_NOT_OWNED_BY_USER
        
        # 404 - Not Found
        - NOT_FOUND
        - ORDER_NOT_FOUND
        - CUSTOMER_NOT_FOUND
        - SKU_NOT_FOUND
        
        # 409 - Conflict / State Violation
        - INSUFFICIENT_INVENTORY
        - DUPLICATE_EMAIL
        - ORDER_ALREADY_SHIPPED
        - PAYMENT_DECLINED
        - CONCURRENCY_CONFLICT
        
        # 429 - Rate Limit
        - RATE_LIMITED
        - TOO_MANY_REQUESTS
        
        # 500 - Server Error
        - INTERNAL_ERROR
        - DATABASE_ERROR
        - EXTERNAL_SERVICE_ERROR
    
    DetailedError:
      type: object
      required: [code, message, http_status]
      properties:
        code:
          $ref: '#/components/schemas/ErrorCode'
        message:
          type: string
          description: Human-readable explanation
        http_status:
          type: integer
          example: 400
        details:
          type: object
          description: Context-specific details
          example:
            field: email
            reason: must be unique
            current_value: user@example.com
        request_id:
          type: string
          description: For support tickets
        timestamp:
          type: string
          format: date-time
```

#### Error Responses in Practice

**Validation Error (400):**

```json
{
  "status": "error",
  "error": {
    "code": "INVALID_INPUT",
    "message": "Validation failed for order creation",
    "http_status": 400,
    "details": {
      "fields": {
        "total": "must be greater than 0",
        "shipping_address.zip_code": "invalid format"
      }
    },
    "request_id": "req_abc123",
    "timestamp": "2024-01-16T14:22:15Z"
  }
}
```

**Inventory Conflict (409):**

```json
{
  "status": "error",
  "error": {
    "code": "INSUFFICIENT_INVENTORY",
    "message": "Cannot create order: not enough stock",
    "http_status": 409,
    "details": {
      "sku": "LAPTOP-001",
      "requested": 5,
      "available": 2,
      "recommendation": "Order 2 items now, or wait for restock on 2024-01-20"
    },
    "request_id": "req_abc123",
    "timestamp": "2024-01-16T14:22:15Z"
  }
}
```

**Rate Limit (429):**

```json
{
  "status": "error",
  "error": {
    "code": "RATE_LIMITED",
    "message": "API rate limit exceeded",
    "http_status": 429,
    "details": {
      "limit": 100,
      "window_seconds": 60,
      "retry_after_seconds": 23
    },
    "request_id": "req_abc123",
    "timestamp": "2024-01-16T14:22:15Z"
  }
}
```

---

## 4.2 ENDPOINT DESIGN

### 4.2.1 Resource Identification & CRUD Mapping

Every resource you model in the database gets REST endpoints.

#### Database Models → API Resources

| Database Model | API Endpoint | CRUD Operations |
|---------------|--------------|-----------------|
| **users** | `/api/v1/users` | POST (register) |
| | `/api/v1/users/{id}` | GET (profile) |
| | `/api/v1/users/{id}` | PATCH (update) |
| **orders** | `/api/v1/orders` | POST (create) |
| | `/api/v1/orders` | GET (list) |
| | `/api/v1/orders/{id}` | GET (details) |
| | `/api/v1/orders/{id}` | PATCH (modify) |
| | `/api/v1/orders/{id}` | DELETE (cancel) |
| **inventory** | `/api/v1/inventory` | GET (list) |
| | `/api/v1/inventory/{sku}` | GET (by SKU) |
| | `/api/v1/inventory/{sku}` | PATCH (update qty) |
| **payments** | `/api/v1/payments` | POST (create charge) |
| | `/api/v1/payments/{id}` | GET (status) |
| | `/api/v1/payments/{id}/refund` | POST (issue refund) |

#### Key Principle

**Don't create an endpoint for every database operation.**

**Example:** User role change

- ❌ Separate endpoint: `POST /api/v1/users/{id}/roles` → Confusing (is this add role or set role?)
- ✅ PATCH existing resource: `PATCH /api/v1/users/{id}` with `{"role": "admin"}` → Clear

#### CRUD Mapping Rules

| Operation | HTTP Method | Endpoint | Idempotent | Status Code |
|-----------|------------|----------|------------|-------------|
| Create | POST | `/api/v1/orders` | No | 201 |
| Read (one) | GET | `/api/v1/orders/{id}` | Yes | 200 |
| Read (list) | GET | `/api/v1/orders` | Yes | 200 |
| Update (full) | PUT | `/api/v1/orders/{id}` | Yes | 200 |
| Update (partial) | PATCH | `/api/v1/orders/{id}` | Yes | 200 |
| Delete | DELETE | `/api/v1/orders/{id}` | Yes | 204 |

---

### 4.2.2 Query Parameters: Filtering, Sorting, Pagination

These are the most-abused aspects of API design. Get them right.

#### Filtering

**Example:** `GET /api/v1/orders`

```
?status=shipped                                  → orders with status = shipped
?status=shipped,processing                       → orders with status IN (shipped, processing)
?created_after=2024-01-01                        → orders created on or after 2024-01-01
?created_before=2024-01-31                       → orders created before 2024-01-31
?customer_id=123                                 → orders for customer 123
?min_total=100&max_total=500                     → total price between 100-500
?search=laptop                                   → Full-text search in order items
```

**Rules:**

- Single value: `?status=shipped`
- Multiple values: `?status=shipped,processing` (comma-separated, not repeated params)
- Range queries: `?min_price=10&max_price=100`
- Dates: ISO 8601 format `2024-01-15T10:30:00Z`
- Text search: `?search=query` (backend does full-text search if applicable)

#### Sorting

**Example:** `GET /api/v1/orders`

```
?sort=created_at                  → Ascending by created_at
?sort=-created_at                 → Descending (minus prefix)
?sort=created_at,-total           → Multiple fields (created ascending, total descending)
```

**Rules:**

- Default sort should be consistent (typically `-created_at`, newest first)
- Allow only whitelisted fields (don't let user sort by password field)
- Maximum 3 sort fields (prevent performance degradation)

#### Pagination

**Example:** `GET /api/v1/orders`

```
?limit=20&offset=0    → First 20 items
?limit=20&offset=20   → Items 21-40
?limit=20&offset=40   → Items 41-60
```

**Rules:**

- Default limit: 20 items per page (not too big, not too small)
- Max limit: 100 items (prevent accidental scanning entire database)
- Min limit: 1 item
- Default offset: 0 (start from beginning)

**Response includes metadata:**

```json
{
  "status": "success",
  "data": [ /* 20 items */ ],
  "metadata": {
    "total": 15847,         // Total items matching filter
    "limit": 20,
    "offset": 0,
    "has_more": true,       // Are there more items after this page?
    "next_offset": 20,      // Offset for next page
    "pages": 793            // Total pages
  }
}
```

#### Complete Example

**Request:**
```
GET /api/v1/orders?status=shipped,processing&created_after=2024-01-01&min_total=100&sort=-created_at&limit=50&offset=0
```

**Response:**
```json
{
  "status": "success",
  "data": [ /* 50 orders */ ],
  "metadata": {
    "total": 5234,
    "limit": 50,
    "offset": 0,
    "has_more": true,
    "next_offset": 50
  }
}
```

---

### 4.2.3 Request Validation Rules

Validate at the API boundary. Invalid input should never reach database logic.

#### Validation Layers

```
HTTP Request
    ↓
SCHEMA VALIDATION (Zod/Joi)
├─ Required fields present?
├─ Field types correct? (string, number, array)
├─ Field lengths valid? (email < 254 chars)
├─ Field values in enum? (status IN [pending, shipped, ...])
└─ Return 400 if fails
    ↓
BUSINESS LOGIC VALIDATION
├─ Does customer exist?
├─ Is inventory sufficient?
├─ Is payment method valid?
└─ Return 400 or 409 if fails
    ↓
DATABASE LAYER
├─ Apply constraints (unique indexes, foreign keys)
└─ Return 409 Conflict if constraint violated
    ↓
SUCCESS (200/201)
```

#### Validation Rules in Schema

```yaml
components:
  schemas:
    CreateOrderRequest:
      type: object
      required: [items, shipping_address]  # Required fields
      properties:
        items:
          type: array
          minItems: 1        # At least 1 item
          maxItems: 100      # At most 100 items
          items:
            type: object
            required: [sku, quantity]
            properties:
              sku:
                type: string
                minLength: 3       # At least 3 characters
                maxLength: 20      # At most 20 characters
                pattern: '^[A-Z0-9-]+$'  # Alphanumeric + dash only
              quantity:
                type: integer
                minimum: 1         # At least 1
                maximum: 1000      # At most 1000 per item
        shipping_address:
          type: object
          required: [street, city, state, zip]
          properties:
            street:
              type: string
              minLength: 5
              maxLength: 255
            city:
              type: string
              minLength: 2
              maxLength: 100
            state:
              type: string
              minLength: 2
              maxLength: 2
              pattern: '^[A-Z]{2}$'  # US state code
            zip:
              type: string
              pattern: '^\d{5}(-\d{4})?$'  # ZIP or ZIP+4
```

#### Business Logic Validation (After Schema Passes)

```javascript
// POST /api/v1/orders - Create order
async function createOrder(req) {
  // Schema validation passed at middleware level
  const { items, shipping_address } = req.body;
  
  // Business logic validation
  
  // Check 1: SKUs exist and have stock
  for (const item of items) {
    const inventory = await db.inventory.findUnique({ 
      where: { sku: item.sku } 
    });
    
    if (!inventory) {
      return error(404, 'NOT_FOUND', `SKU ${item.sku} doesn't exist`);
    }
    
    if (inventory.quantity < item.quantity) {
      return error(409, 'INSUFFICIENT_INVENTORY',
        `Only ${inventory.quantity} available for ${item.sku}`);
    }
  }
  
  // Check 2: Shipping address is valid (ZIP code format)
  if (!isValidUSZip(shipping_address.zip)) {
    return error(400, 'INVALID_INPUT', 'Invalid ZIP code format');
  }
  
  // Check 3: Verify payment method if included
  if (req.body.payment_method_id) {
    const valid = await stripe.paymentMethods.retrieve(
      req.body.payment_method_id
    );
    if (!valid) {
      return error(400, 'INVALID_INPUT', 'Payment method not found');
    }
  }
  
  // All validations passed, create order
  const order = await db.order.create({ /* ... */ });
  return success(201, order);
}
```

---

## 4.3 API VERSIONING

You will break the API. Plan for it.

### 4.3.1 Versioning Strategy: URL vs Header

#### Strategy 1: URL Versioning (Recommended)

```
GET /api/v1/orders  → Version 1
GET /api/v2/orders  → Version 2 (newer, different response)
```

**Pros:**
- Clear in URLs and logs
- Different code paths per version (no feature flags)
- Can retire old versions explicitly
- No ambiguity

**Cons:**
- URL duplication (v1, v2, v3 code paths side-by-side)
- More disk space

#### Strategy 2: Header Versioning (Less Recommended)

```
GET /api/orders
Header: X-API-Version: 1
```

**Pros:**
- Cleaner URLs
- One code path

**Cons:**
- Easy to miss in logs
- Clients might not set header consistently
- Harder to debug

> **My recommendation:** Use URL versioning. Clean, explicit, no ambiguity.

#### Version Lifecycle

```
Version 1.0
├─ Release date: 2024-01-01
├─ Current: ACTIVE (full support)
├─ Deadline: 2024-12-31 (EOL in 12 months)
└─ Deprecation notice in responses:
    Header: Deprecation: true
    Header: Sunset: Sun, 31 Dec 2024 23:59:59 GMT
    Header: Link: </api/v2>; rel="successor-version"

Version 2.0
├─ Release date: 2024-06-01
├─ Current: ACTIVE (full support)
├─ Deadline: 2025-06-01 (EOL in 12 months from v2 launch)
└─ Deprecation notice:
    Header: Deprecation: true
    Header: Sunset: Sun, 01 Jun 2025 23:59:59 GMT
```

---

### 4.3.2 Backward Compatibility Rules

#### Rule 1: Never Delete Fields from Response

Old clients expect `customer_name` in order response. If you remove it, old app crashes.

**Solution:** Always include fields, even if deprecated.

```javascript
// v1 response
{
  "customer_name": "John Doe",
  "total": 99.99
}

// v2 response (still include customer_name for backward compat)
{
  "customer_name": "John Doe",  // Kept for old clients
  "customer_id": "123",         // New field
  "total": 99.99
}
```

#### Rule 2: Additive Changes Only in Minor Versions

- `v1.0 → v1.1`: Can add optional fields, can't remove/rename existing ones
- `v1.x → v2.0`: Can make breaking changes (removed fields, renamed fields, etc.)

```
v1.0: { id, email, created_at }
v1.1: { id, email, created_at, updated_at }  ✓ Backward compatible
v2.0: { id, user_email, created_timestamp }  ✓ Breaking change acceptable
```

#### Rule 3: New Endpoints in Same Version

If you add a new endpoint `/api/v1/orders/{id}/returns`, it's still v1. Only bump major version if you break existing endpoints.

---

## 4.4 SECURITY DESIGN

### 4.4.1 Authentication Mechanism

**Authentication** = Who are you? (identity)  
**Authorization** = Can you do that? (permissions)

**Chosen mechanism:** JWT (JSON Web Tokens)

#### Why JWT?

- **Stateless** - Don't need session database for every request
- **Scalable** across multiple servers (no sticky sessions)
- **Standard** - Every API uses JWT
- **Mobile-friendly** - Works without cookies
- **Supports refresh tokens** - Separate short-lived + long-lived tokens

#### JWT Structure

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6Ikpva
G4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.SflKxwRJSMeKKF2QT4fwp
MeJf36POk6yJV_adQssw5c

HEADER.PAYLOAD.SIGNATURE
```

- **Header:** `{ "alg": "HS256", "typ": "JWT" }`
- **Payload:** `{ "sub": "user123", "email": "user@example.com", "roles": ["admin"], "exp": 1700000000 }`
- **Signature:** `HMAC-SHA256(base64url(header) + "." + base64url(payload), SECRET_KEY)`

#### Login Flow

**1. User submits email + password**

```
POST /api/v1/auth/login
{ "email": "user@example.com", "password": "secure123" }
```

**2. Backend:**

```
├─ Find user by email in database
├─ Compare submitted password with stored hash (bcrypt)
├─ If mismatch → return 401 Unauthorized
├─ If match → create tokens:
│  ├─ Access token: JWT valid for 15 minutes
│  ├─ Refresh token: JWT valid for 30 days (secure HTTP-only cookie)
└─ Return tokens
```

**3. Response:**

```
Status: 200 OK
Body: { 
  "access_token": "eyJ...", 
  "user": { "id": "123", "email": "..." } 
}
Set-Cookie: refresh_token=eyJ...; Path=/; HttpOnly; Secure; SameSite=Strict
```

**4. Subsequent requests:**

Frontend includes:
```
Header: Authorization: Bearer eyJ...  (15-minute access token)
```

**5. When access token expires:**

```
POST /api/v1/auth/refresh
Cookie: refresh_token=eyJ...  (auto-included by browser)

Response: { "access_token": "new eyJ..." }
Set-Cookie: refresh_token=new eyJ...; ...
```

**6. On logout:**

```
DELETE /api/v1/auth/logout
Response: Clear refresh_token cookie
```

#### Access Token Payload Example

```json
{
  "sub": "user-123",                                    // Subject (user ID)
  "email": "john@example.com",
  "roles": ["user", "admin"],                           // User roles
  "permissions": ["orders:read", "orders:write", "users:write"],
  "iat": 1700000000,                                    // Issued at
  "exp": 1700000900,                                    // Expiration (15 min later)
  "iss": "https://api.company.com",                     // Issuer
  "aud": ["web", "mobile"]                              // Audience (who can use this)
}
```

#### Refresh Token Payload

```json
{
  "sub": "user-123",
  "type": "refresh",                                    // Mark as refresh token (not access)
  "iat": 1700000000,
  "exp": 1730000000,                                    // 30 days later
  "iss": "https://api.company.com"
}
```

#### Token Validation Middleware

```javascript
// Every protected endpoint runs this
app.use(authenticateJWT);

async function authenticateJWT(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({
      error: { code: 'UNAUTHORIZED', message: 'Missing token' }
    });
  }
  
  try {
    // Verify signature + expiration (NO database call)
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({
        error: { code: 'TOKEN_EXPIRED', message: 'Token expired, use refresh' }
      });
    }
    return res.status(401).json({
      error: { code: 'TOKEN_INVALID', message: 'Invalid token' }
    });
  }
}
```

---

### 4.4.2 Authorization Model: RBAC

**RBAC** = Role-Based Access Control

Users have roles. Roles have permissions. Permissions control what can be done.

#### Model

```
User
├─ id
├─ email
└─ roles: [admin, user, moderator]

Role
├─ admin
│  └─ permissions: [users:read, users:write, orders:read, orders:write, ...]
├─ user
│  └─ permissions: [orders:read, orders:write (own), profile:read, ...]
└─ moderator
   └─ permissions: [orders:read, users:read, reports:read, ...]

Permission
├─ users:read     → Can view user list
├─ users:write    → Can create/update users
├─ orders:read    → Can view orders
├─ orders:write   → Can create orders
├─ admin:access   → Can access admin panel
└─ ...
```

#### Checking Permissions

```javascript
// Middleware to check if user has permission
function authorize(requiredPermission: string) {
  return (req, res, next) => {
    const userPermissions = req.user.permissions;  // From JWT token
    
    if (!userPermissions.includes(requiredPermission)) {
      return res.status(403).json({
        error: {
          code: 'FORBIDDEN',
          message: `Missing permission: ${requiredPermission}`
        }
      });
    }
    
    next();
  };
}

// Apply to endpoints
app.get('/api/v1/users', authenticate, authorize('users:read'), listUsers);
app.post('/api/v1/orders', authenticate, authorize('orders:write'), createOrder);
app.delete('/api/v1/users/:id', authenticate, authorize('admin:access'), deleteUser);
```

#### Resource-Level Authorization (Most Important)

A user should only see/modify their own data.

```javascript
app.get('/api/v1/orders/:order_id', authenticate, async (req, res) => {
  const order = await db.order.findUnique({ 
    where: { id: req.params.order_id } 
  });
  
  // Check: Does this order belong to requesting user?
  if (order.customer_id !== req.user.sub) {
    return res.status(403).json({
      error: { code: 'FORBIDDEN', message: 'Order not owned by user' }
    });
  }
  
  return res.json({ status: 'success', data: order });
});
```

---

### 4.4.3 Rate Limiting

**Why:** Prevent abuse (brute force attacks, DDoS), ensure fair resource use.

**Strategy:** Token bucket algorithm

- Each user has a bucket with N tokens
- Each request consumes 1 token
- Tokens refill at rate of M tokens/minute

#### Example

```
├─ Limit: 100 requests per minute
├─ Burst: Allow up to 100 requests at once (full bucket)
├─ Refill: 100 tokens / 60 seconds = 1.67 tokens/second

User 1:
├─ Makes 100 requests in 1 second → Rate limited (quota exceeded)
├─ Waits 30 seconds
├─ ~50 tokens refilled, can make 50 more requests

User 2:
├─ Makes 50 requests over 60 seconds
├─ Within limit, no rate limiting
```

#### Implementation in Redis

```javascript
import Redis from 'ioredis';
const redis = new Redis(process.env.REDIS_URL);

async function checkRateLimit(
  userId: string, 
  limit: number = 100, 
  windowSeconds: number = 60
) {
  const key = `rate_limit:${userId}`;
  
  // Increment counter
  const current = await redis.incr(key);
  
  // First request: Set expiration
  if (current === 1) {
    await redis.expire(key, windowSeconds);
  }
  
  // Check if exceeded
  if (current > limit) {
    const ttl = await redis.ttl(key);
    return {
      allowed: false,
      limit,
      remaining: 0,
      resetIn: ttl
    };
  }
  
  return {
    allowed: true,
    limit,
    remaining: limit - current,
    resetIn: await redis.ttl(key)
  };
}

// Apply to all routes
app.use(async (req, res, next) => {
  const userId = req.user?.sub || req.ip;  // Use user ID or IP if not authenticated
  const { allowed, remaining, resetIn } = await checkRateLimit(userId);
  
  // Add rate limit headers to response
  res.setHeader('X-RateLimit-Limit', 100);
  res.setHeader('X-RateLimit-Remaining', remaining);
  res.setHeader('X-RateLimit-Reset', Date.now() + resetIn * 1000);
  
  if (!allowed) {
    return res.status(429).json({
      error: {
        code: 'RATE_LIMITED',
        message: 'Too many requests',
        details: { retry_after: resetIn }
      }
    });
  }
  
  next();
});
```

#### Different Limits Per Endpoint

```
Regular endpoints: 100 requests/minute
├─ GET /api/v1/orders: 100/min
├─ GET /api/v1/products: 100/min

Expensive endpoints: 10 requests/minute
├─ POST /api/v1/reports/generate: 10/min (heavy computation)
├─ POST /api/v1/bulk-import: 10/min (database intensive)

Login endpoint: 5 requests/minute (brute force protection)
├─ POST /api/v1/auth/login: 5/min per IP

Public endpoints: 1000 requests/minute
├─ GET /api/v1/public/products: 1000/min
```

---

### 4.4.4 CORS Configuration

**CORS** = Cross-Origin Resource Sharing

Your API is at `api.company.com`. Your frontend is at `app.company.com`. Same company, different subdomain. Browser blocks it unless you explicitly allow it.

#### Minimal CORS Config

```javascript
import cors from 'cors';

app.use(cors({
  origin: process.env.FRONTEND_URL,        // Only this domain can access API
  credentials: true,                       // Allow cookies/auth headers
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  exposedHeaders: ['X-RateLimit-Remaining', 'X-Request-ID'],
  maxAge: 3600                             // Preflight cache 1 hour
}));
```

#### For Multiple Origins (Staging, Production)

```javascript
const allowedOrigins = [
  'https://app.company.com',           // Production
  'https://staging-app.company.com',   // Staging
  'http://localhost:3000'              // Local dev
];

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('CORS not allowed'));
    }
  },
  credentials: true
}));
```

---

### 4.4.5 Security Headers

HTTP security headers prevent common attacks.

```javascript
import helmet from 'helmet';

app.use(helmet());  // Sets all security headers by default

// Or configure manually:
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'", "https://api.company.com"]
    }
  },
  hsts: {
    maxAge: 31536000,           // 1 year
    includeSubDomains: true,
    preload: true
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  noSniff: true,
  xssFilter: true
}));
```

#### What Each Header Does

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
└─ Force HTTPS, never allow HTTP

Content-Security-Policy: default-src 'self'
└─ Only load resources from own domain (prevent XSS, injection attacks)

X-Content-Type-Options: nosniff
└─ Don't guess content type, trust the Content-Type header

X-Frame-Options: DENY
└─ Don't allow embedding in iframes (clickjacking prevention)

X-XSS-Protection: 1; mode=block
└─ Enable XSS protection in browser
```

---

## 4.5 API DOCUMENTATION

### 4.5.1 Interactive Documentation (Swagger UI)

Swagger UI generates interactive docs directly from OpenAPI spec.

```javascript
import swaggerJsdoc from 'swagger-jsdoc';
import swaggerUi from 'swagger-ui-express';

// Read openapi.yaml
const spec = swaggerJsdoc({
  definition: {
    openapi: '3.0.0',
    info: { title: 'API', version: '1.0.0' },
    servers: [{ url: 'http://localhost:3000/api/v1' }]
  },
  apis: ['./routes/*.ts']
});

// Serve at /api/docs
app.use('/api/docs', swaggerUi.serve, swaggerUi.setup(spec));
```

Users visit `https://api.company.com/api/docs`:

- See all endpoints listed
- Try requests directly (click "Try it out")
- See request/response examples
- Copy curl commands for testing

---

### 4.5.2 Code Examples

Every endpoint should have examples in 2+ languages.

#### In OpenAPI Spec

```yaml
paths:
  /orders:
    post:
      summary: Create order
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrderRequest'
            examples:
              simple:
                value:
                  items:
                    - sku: "LAPTOP-001"
                      quantity: 1
                  shipping_address:
                    street: "123 Main St"
                    city: "San Francisco"
                    state: "CA"
                    zip: "94102"
      responses:
        '201':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
              examples:
                success:
                  value:
                    id: "550e8400-e29b-41d4-a716-446655440000"
                    customer_id: "660e8400-e29b-41d4-a716-446655440000"
                    total: 1299.99
                    status: "pending"
```

#### In Documentation Site

**cURL:**

```bash
curl -X POST https://api.company.com/v1/orders \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "items": [{"sku": "LAPTOP-001", "quantity": 1}],
    "shipping_address": {"street": "123 Main", "city": "SF", ...}
  }'
```

**JavaScript/TypeScript:**

```javascript
import fetch from 'node-fetch';

const response = await fetch('https://api.company.com/v1/orders', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    items: [{ sku: 'LAPTOP-001', quantity: 1 }],
    shipping_address: { ... }
  })
});

const order = await response.json();
console.log(order.data.id);
```

**Python:**

```python
import requests

response = requests.post(
    'https://api.company.com/v1/orders',
    headers={'Authorization': f'Bearer {token}'},
    json={
        'items': [{'sku': 'LAPTOP-001', 'quantity': 1}],
        'shipping_address': {...}
    }
)

order = response.json()['data']
print(order['id'])
```

---

### 4.5.3 Error Code Catalog

Document every error code your API can return, with explanation and how to handle it.

#### INVALID_INPUT (400)

```
├─ What: Validation failed (missing fields, wrong types, constraints violated)
├─ Example: { "message": "missing required field: email" }
├─ How to handle: Show validation errors to user, let them correct and retry
├─ Retry: No (user needs to fix input)
```

#### INSUFFICIENT_INVENTORY (409)

```
├─ What: Not enough stock for order
├─ Example: { "message": "only 2 available for SKU LAPTOP-001, requested 5" }
├─ How to handle: Show available qty to user, suggest ordering less or waiting for restock
├─ Retry: No (inventory hasn't changed)
```

#### RATE_LIMITED (429)

```
├─ What: Too many requests in short time
├─ Example: { "message": "100 requests/min limit exceeded" }
├─ How to handle: Wait for retry_after seconds, then retry
├─ Retry: Yes (exponential backoff)
```

#### INTERNAL_ERROR (500)

```
├─ What: Unexpected server error (bug, crash, etc.)
├─ Example: { "message": "internal server error" }
├─ How to handle: Show "something went wrong" to user, log request_id for support
├─ Retry: Yes (error might be temporary)
```

---

### 4.5.4 Authentication Flow Documentation

Document how to authenticate with code examples.

#### 1. Registration (Optional)

```
POST /api/v1/auth/register
{
  "email": "user@example.com",
  "password": "secure123",
  "name": "John Doe"
}

Response: { "user": {...}, "access_token": "...", "refresh_token": "..." }
```

#### 2. Login

```
POST /api/v1/auth/login
{
  "email": "user@example.com",
  "password": "secure123"
}

Response:
{
  "access_token": "eyJ...",
  "user": {
    "id": "123",
    "email": "user@example.com",
    "roles": ["user"]
  }
}
Header: Set-Cookie: refresh_token=eyJ...; HttpOnly; Secure; Path=/
```

#### 3. Use Token

```
GET /api/v1/orders
Header: Authorization: Bearer eyJ...

Response: { "data": [...] }
```

#### 4. Token Expires (15 min)

```
GET /api/v1/orders
Header: Authorization: Bearer (expired)

Response: 401 Unauthorized, "token_expired"
```

#### 5. Refresh Token

```
POST /api/v1/auth/refresh
Cookie: refresh_token=eyJ... (auto-sent by browser)

Response: { "access_token": "new eyJ..." }
```

#### 6. Logout

```
DELETE /api/v1/auth/logout

Response: Clear refresh_token cookie
```

---

## 4.6 API DESIGN REVIEW GATES

### Gate 1: Frontend Team Contract Review

Before backend implementation, get frontend sign-off on API contract.

#### Checklist

- [ ] Endpoints make sense for frontend workflows
- [ ] Response shapes match frontend model
- [ ] Query parameters sufficient for filtering/sorting needs
- [ ] Error codes are actionable (frontend can show meaningful messages)
- [ ] Pagination strategy works for infinite scroll / table views
- [ ] Rate limits won't break normal usage
- [ ] Documentation is clear and complete

**Meeting:** 1 hour, backend + frontend leads, sign-off in pull request

---

### Gate 2: Security Review

Before merging spec, security team reviews authentication/authorization.

#### Checklist

- [ ] JWT secrets properly rotated (not hardcoded)
- [ ] Token expiration times reasonable (15 min access, 30 day refresh)
- [ ] RBAC model covers all resource types
- [ ] Resource-level authorization checks in place
- [ ] Rate limiting prevents brute force (login: 5 req/min)
- [ ] CORS only allows trusted origins
- [ ] Security headers set (HSTS, CSP, X-Frame-Options)
- [ ] No sensitive data in JWT payload (no passwords, SSN, etc.)
- [ ] Password requirements documented (min length 8, no common patterns)
- [ ] Password reset flows don't leak user existence
- [ ] Credentials never logged or cached

---

## 4.6.1 Deliverables

### Complete OpenAPI Specification (`openapi.yaml`)

- All endpoints defined with request/response schemas
- All error codes documented
- Security schemes defined
- Examples for key endpoints

### API Documentation Site (Swagger UI deployed)

- Interactive endpoint explorer
- Try-it-out feature (sandbox)
- Code examples (cURL, JavaScript, Python)
- Authentication flow documentation

### Postman Collection (for QA/testing)

- All endpoints importable into Postman
- Environment variables for token, base URL
- Pre-request scripts for auth
- Tests for happy path + error cases

### Security Document

- Authentication mechanism explained
- Authorization model (RBAC)
- Rate limits per endpoint
- CORS policy
- Security headers

---

## SUMMARY: WHAT YOU'VE DEFINED

By end of this phase, you have:

- ✅ **OpenAPI spec** - Contract that frontend and backend both follow
- ✅ **Resource model** - What endpoints exist, how they relate
- ✅ **Error taxonomy** - Every error code, what causes it, how to handle it
- ✅ **Authentication** - How users prove identity (JWT)
- ✅ **Authorization** - What users can do (RBAC)
- ✅ **Rate limiting** - Fair use limits per endpoint
- ✅ **Documentation** - Interactive, examples, error codes
- ✅ **Security gates** - Frontend + security review completed

You now have a contract. Backend and frontend can work in parallel. Anyone joining the team reads the spec and understands the entire API without reading code.

**Cost of getting this wrong:** Rework everything later.

**Cost of getting this right:** 3-4 days of design work. Saves 3-4 weeks of rework.

> **20-year veteran's take:** The best technical teams I've seen spend 20% of time on design, 80% on implementation. Bad teams do it backwards.

---

## Footnotes

1. **Pattern `^[A-Z0-9-]+$`** - Alphanumeric (A-Z, 0-9) and dash (-) only
