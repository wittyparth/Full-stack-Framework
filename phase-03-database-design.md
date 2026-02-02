# PHASE 3: DATABASE DESIGN

## Overview

Database design is where architecture meets reality. Poor schema design causes cascading problems: slow queries impact API response times, missing indexes create bottlenecks, denormalization causes data inconsistency, and migration nightmares block deployments. This phase transforms the architecture into a bulletproof schema that scales to 50,000+ concurrent users.

Your database is the source of truth for ALL business logic. Get this right, and scaling becomes straightforward. Get it wrong, and you're rewriting migrations at 2am during production incidents.

---

## 3.1 DATA MODELING

### 3.1.1 Entity Identification

**What to do:**
- Identify all real-world entities that need persistent storage
- Map them to business domains (Orders, Inventory, Users, Payments)
- Determine relationships between entities
- Assess cardinality (1-to-1, 1-to-many, many-to-many)
- Document natural vs surrogate keys

**Core entities for our system:**

```
USERS & AUTHENTICATION:
├─ users
│  ├─ id (PK)
│  ├─ email (UK - unique key)
│  ├─ password_hash (bcrypt)
│  ├─ first_name, last_name
│  ├─ role (admin, manager, employee, customer)
│  ├─ phone
│  ├─ created_at
│  ├─ updated_at
│  └─ deleted_at (soft delete)
│
├─ user_addresses
│  ├─ id (PK)
│  ├─ user_id (FK)
│  ├─ type (billing, shipping, return)
│  ├─ address_line_1
│  ├─ address_line_2
│  ├─ city, state, country, postal_code
│  ├─ is_default
│  └─ created_at
│
├─ user_payment_methods
│  ├─ id (PK)
│  ├─ user_id (FK)
│  ├─ stripe_payment_method_id (external ID, never store full CC)
│  ├─ card_last_4
│  ├─ card_brand (Visa, Mastercard, etc)
│  ├─ exp_month, exp_year
│  ├─ is_default
│  └─ created_at

ORDERS & ORDER FULFILLMENT:
├─ orders
│  ├─ id (PK)
│  ├─ customer_id (FK to users)
│  ├─ order_number (UK - human readable)
│  ├─ status (pending, confirmed, shipped, delivered, cancelled)
│  ├─ subtotal (items only)
│  ├─ shipping_cost
│  ├─ tax
│  ├─ total (denormalized, calculated = subtotal + shipping + tax)
│  ├─ shipping_address_id (FK)
│  ├─ billing_address_id (FK)
│  ├─ created_at
│  ├─ shipped_at
│  ├─ delivered_at
│  └─ cancelled_at
│
├─ order_items
│  ├─ id (PK)
│  ├─ order_id (FK)
│  ├─ product_id (FK)
│  ├─ quantity
│  ├─ unit_price (snapshot at order time, never changes)
│  ├─ line_total (denormalized = quantity * unit_price)
│  └─ created_at
│
├─ payments
│  ├─ id (PK)
│  ├─ order_id (FK, UK with payment_method - prevent duplicate charges)
│  ├─ payment_method_id (FK)
│  ├─ amount (cents, always integer)
│  ├─ status (pending, succeeded, failed, refunded, disputed)
│  ├─ stripe_charge_id (external reference)
│  ├─ failure_reason (if failed)
│  ├─ attempted_at
│  └─ created_at
│
├─ shipments
│  ├─ id (PK)
│  ├─ order_id (FK)
│  ├─ carrier (usps, fedex, ups)
│  ├─ tracking_number (UK)
│  ├─ label_url
│  ├─ estimated_delivery
│  ├─ shipped_at
│  ├─ delivered_at
│  └─ created_at

INVENTORY:
├─ products
│  ├─ id (PK)
│  ├─ sku (UK - stock keeping unit)
│  ├─ name
│  ├─ description
│  ├─ category
│  ├─ price (cents, always integer)
│  ├─ cost (for margin calculation)
│  ├─ image_url
│  ├─ active (soft delete via is_active flag)
│  ├─ created_at
│  └─ updated_at
│
├─ inventory
│  ├─ id (PK)
│  ├─ product_id (FK, UK - one row per product)
│  ├─ quantity_on_hand
│  ├─ quantity_reserved (allocated to pending orders)
│  ├─ quantity_available (calculated = on_hand - reserved, denormalized)
│  ├─ reorder_point (when to re-order)
│  ├─ warehouse_location
│  ├─ updated_at
│  └─ last_counted_at
│
├─ inventory_transactions
│  ├─ id (PK)
│  ├─ product_id (FK)
│  ├─ order_id (FK, nullable - some transactions not order-related)
│  ├─ type (purchase, sale, adjustment, count_variance, return)
│  ├─ quantity_change (positive or negative)
│  ├─ notes (reason for adjustment)
│  ├─ created_by (user_id who initiated)
│  └─ created_at

AUDITING & COMPLIANCE:
├─ audit_logs
│  ├─ id (PK)
│  ├─ user_id (FK, nullable - system actions have no user)
│  ├─ entity_type (order, inventory, user, payment)
│  ├─ entity_id
│  ├─ action (create, update, delete, view)
│  ├─ old_values (JSON of changed fields before)
│  ├─ new_values (JSON of changed fields after)
│  ├─ ip_address
│  ├─ user_agent
│  ├─ created_at
│  └─ INDEX: (user_id, created_at) for "what did user do"
│         (entity_type, entity_id, created_at) for "history of entity"
```

**Entity relationship cardinality:**

```
users ──────────────── 1:M ────────────────── orders
   (customer)

users ──────────────── 1:M ────────────────── user_addresses

users ──────────────── 1:M ────────────────── user_payment_methods

orders ─────────────── 1:M ────────────────── order_items

products ───────────── 1:M ────────────────── order_items

products ───────────── 1:1 ────────────────── inventory

orders ─────────────── 1:M ────────────────── payments

orders ─────────────── 1:M ────────────────── shipments

user_payment_methods ── 1:M ────────────────── payments

products ───────────── 1:M ────────────────── inventory_transactions

orders ─────────────── 1:M ────────────────── inventory_transactions
```

**Why these entities:**

- **users**: Authentication, order tracking, permissions
- **user_addresses**: Multiple addresses per user (shipping ≠ billing)
- **user_payment_methods**: Multiple cards, never store CC directly
- **orders**: Core transaction record, immutable once created
- **order_items**: Line items, references product but stores snapshot price
- **payments**: Separate from orders (order can have multiple payment attempts)
- **shipments**: Tracking info, independent lifecycle from order
- **products**: Catalog, shared across orders
- **inventory**: Current stock, not in products table (changes frequently)
- **inventory_transactions**: Audit trail of every stock change
- **audit_logs**: Compliance, debugging, security (required for SLA)

---

### 3.1.2 Entity-Relationship Diagram (ERD)

**Visual representation:**

```
                      ┌─────────────────┐
                      │      USERS      │
                      ├─────────────────┤
                      │ id (PK)         │
                      │ email (UK)      │
                      │ password_hash   │
                      │ role            │
                      │ created_at      │
                      └────┬────────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
       1:M │             1:M │            1:M │
          │                │                │
    ┌─────▼──────┐  ┌──────▼────────┐ ┌────▼──────────┐
    │  ADDRESSES  │  │ORDER_PAYMENTS │ │ AUDIT_LOGS    │
    │             │  │   METHODS      │ │               │
    └─────────────┘  └────────────────┘ └───────────────┘

                      ┌──────────────┐
                      │    ORDERS    │
                      ├──────────────┤
                      │ id (PK)      │
                      │customer_id   │──────────→ users.id
                      │total         │
                      │status        │
                      │created_at    │
                      └──┬──────────┬─┘
                         │          │
                      1:M │        1:M │
                         │          │
         ┌────────────────▼┐  ┌─────▼──────────┐
         │  ORDER_ITEMS    │  │   PAYMENTS     │
         ├────────────────┤  ├────────────────┤
         │ id (PK)        │  │ id (PK)        │
         │ order_id (FK)  │  │ order_id (FK)  │
         │ product_id (FK)├→─┤ amount         │
         │ quantity       │  │ status         │
         │ unit_price     │  │ stripe_id      │
         └────────────────┘  └────────────────┘
                │
                │ 1:M (references)
                │
         ┌──────▼─────────┐
         │   PRODUCTS     │
         ├────────────────┤
         │ id (PK)        │
         │ sku (UK)       │
         │ name           │
         │ price          │
         │ created_at     │
         └────────────────┘
                │
                │ 1:1
                │
         ┌──────▼─────────┐
         │  INVENTORY     │
         ├────────────────┤
         │ id (PK)        │
         │ product_id(FK) │
         │ quantity_onhand│
         │ updated_at     │
         └────────────────┘
                │
                │ 1:M
                │
      ┌─────────▼───────────┐
      │INVENTORY_TRANS      │
      ├─────────────────────┤
      │ id (PK)             │
      │ product_id (FK)     │
      │ type                │
      │ quantity_change     │
      │ created_at          │
      └─────────────────────┘
```

---

### 3.1.3 Attribute Definition & Data Types

**What to do:**
- Define exact data type for each attribute
- Specify constraints (nullable, defaults, ranges)
- Plan for future extensibility
- Consider storage efficiency

**Attribute specifications:**

```
USERS TABLE EXAMPLE (detailed):

Column          │ Type        │ Constraints      │ Rationale
────────────────┼─────────────┼──────────────────┼──────────────────────────
id              │ UUID        │ PK, NOT NULL     │ Distributed systems friendly
email           │ VARCHAR(255)│ UK, NOT NULL     │ Email is natural key, immutable
password_hash   │ VARCHAR(255)│ NOT NULL         │ Bcrypt output is exactly 60 chars
first_name      │ VARCHAR(100)│ NOT NULL         │ Reasonable name length
last_name       │ VARCHAR(100)│ NOT NULL         │ Reasonable name length
phone           │ VARCHAR(20) │ NULL             │ Optional, formatted consistently
role            │ ENUM        │ NOT NULL,        │ Fixed set: admin, manager, employee
                │             │ DEFAULT=customer │
email_verified  │ BOOLEAN     │ NOT NULL,        │ Track unverified signups
                │             │ DEFAULT=false    │
created_at      │ TIMESTAMP   │ NOT NULL,        │ ISO 8601, UTC only
                │             │ DEFAULT=NOW()    │
updated_at      │ TIMESTAMP   │ NOT NULL,        │ Updated on any change
                │             │ DEFAULT=NOW()    │
deleted_at      │ TIMESTAMP   │ NULL             │ Soft delete, null = active

ORDERS TABLE EXAMPLE:

Column          │ Type        │ Constraints      │ Rationale
────────────────┼─────────────┼──────────────────┼──────────────────────────
id              │ UUID        │ PK, NOT NULL     │ Prevent guessing sequential IDs
customer_id     │ UUID        │ FK(users),       │ Never null - must have customer
                │             │ NOT NULL         │
order_number    │ VARCHAR(50) │ UK, NOT NULL     │ Human readable: ORD-2024-0001
status          │ ENUM        │ NOT NULL,        │ pending → confirmed → shipped → delivered
                │             │ DEFAULT=pending  │ Immutable status transitions
subtotal        │ BIGINT      │ NOT NULL,        │ Cents, avoids floating point
                │             │ CHECK > 0        │
shipping_cost   │ BIGINT      │ NOT NULL,        │ 0 if free shipping
                │             │ DEFAULT=0        │
tax             │ BIGINT      │ NOT NULL,        │ Calculated from state + total
                │             │ DEFAULT=0        │
total           │ BIGINT      │ NOT NULL,        │ Denormalized: subtotal+ship+tax
                │             │ CHECK > 0        │
created_at      │ TIMESTAMP   │ NOT NULL,        │ When order placed
                │             │ DEFAULT=NOW()    │
shipped_at      │ TIMESTAMP   │ NULL             │ When actually shipped (not estimated)
delivered_at    │ TIMESTAMP   │ NULL             │ When customer received
cancelled_at    │ TIMESTAMP   │ NULL             │ Soft delete via cancelled_at

INVENTORY TABLE:

Column          │ Type        │ Constraints      │ Rationale
────────────────┼─────────────┼──────────────────┼──────────────────────────
id              │ UUID        │ PK, NOT NULL     │ Allow shard key in future
product_id      │ UUID        │ FK(products),    │ 1:1 relationship, never null
                │             │ UK, NOT NULL     │
quantity_onhand │ INTEGER     │ NOT NULL,        │ Current stock, ≥ 0
                │             │ CHECK >= 0       │
quantity_reserved│ INTEGER    │ NOT NULL,        │ Allocated to pending orders
                │             │ DEFAULT=0        │
quantity_avail  │ INTEGER     │ NOT NULL,        │ Denormalized: onhand - reserved
                │             │ GENERATED ALWAYS │
                │             │ (onhand-reserved)│
reorder_point   │ INTEGER     │ NOT NULL,        │ Alert when < this
updated_at      │ TIMESTAMP   │ NOT NULL,        │ Changes frequently
                │             │ DEFAULT=NOW()    │

KEY DECISIONS:
├─ BIGINT for money: Never use DECIMAL for amounts in distributed systems
│  ├─ Store as cents (99.99 → 9999 as BIGINT)
│  ├─ No floating point errors
│  ├─ Integer math is fast on all databases
│  └─ Supports currency conversion correctly
│
├─ ENUM for status fields: Fixed set of values
│  ├─ Database enforces valid transitions
│  ├─ Prevents invalid states (order_status='invalid')
│  ├─ Efficient storage (1 byte vs VARCHAR)
│  └─ Never add statuses without migration
│
├─ UUID for PKs: Better than auto-increment
│  ├─ No sequential patterns (security)
│  ├─ Works with distributed data
│  ├─ Prevent ID enumeration attacks
│  └─ Slightly larger (16 bytes vs 8) but worth it
│
├─ TIMESTAMP NOT NULL DEFAULT NOW(): Every table gets this
│  ├─ Audit trail
│  ├─ Query recent changes
│  ├─ Always UTC
│  └─ Immutable once set
│
└─ Soft deletes (deleted_at TIMESTAMP NULL):
   ├─ Never physically delete
   ├─ Historical data preserved
   ├─ Compliance/audit requirements
   └─ Add WHERE deleted_at IS NULL to every query
```

---

### 3.1.4 Relationship Mapping & Cardinality

**Critical rule: Get cardinality RIGHT, or everything breaks.**

**Cardinality analysis for each relationship:**

```
1. USERS → ORDERS (1:M)
   ├─ Cardinality: One user has many orders
   ├─ Implemented: orders.customer_id → users.id
   ├─ Queries:
   │  ├─ "Get all orders for user X" → SELECT * FROM orders WHERE customer_id = X
   │  ├─ "Get user's most recent order" → SELECT * FROM orders WHERE customer_id = X ORDER BY created_at DESC LIMIT 1
   │  └─ Frequent: Cache heavily (user's orders list)
   │
   └─ Schema implications:
      ├─ Index on orders(customer_id, created_at)
      ├─ Partition by customer_id at scale (Year 2)
      └─ Cascade delete: NO (keep historical data)

2. ORDERS → ORDER_ITEMS (1:M)
   ├─ Cardinality: One order has multiple items (typical 1-10)
   ├─ Implemented: order_items.order_id → orders.id
   ├─ Queries:
   │  ├─ "Get all items in order X" → SELECT * FROM order_items WHERE order_id = X
   │  ├─ "Calculate order total" → SUM(quantity * unit_price) GROUP BY order_id
   │  └─ Always loaded together (denormalize total to orders)
   │
   └─ Schema implications:
      ├─ Index on order_items(order_id)
      ├─ Cascade delete: YES (deleting order deletes items)
      └─ Store unit_price in order_items (never changes)

3. ORDER_ITEMS → PRODUCTS (Many:1)
   ├─ Cardinality: Many orders reference same product
   ├─ Implemented: order_items.product_id → products.id
   ├─ Queries:
   │  ├─ "What products are in this order?" → JOIN order_items ← products
   │  ├─ "How many orders for product X?" → COUNT(*) FROM order_items WHERE product_id = X
   │  └─ Critical: Don't modify products after order (store snapshot)
   │
   └─ Schema implications:
      ├─ Index on order_items(product_id)
      ├─ Cascade delete: NO (products live independent of orders)
      └─ unit_price in order_items is immutable (never references products.price)

4. PRODUCTS → INVENTORY (1:1)
   ├─ Cardinality: Each product has exactly one inventory record
   ├─ Implemented: inventory.product_id → products.id (UNIQUE constraint)
   ├─ Queries:
   │  ├─ "Check if product in stock" → SELECT quantity_available FROM inventory WHERE product_id = X
   │  ├─ "Check before order creation" → Redis first (cache miss → DB)
   │  └─ Most frequent query (optimize heavily)
   │
   └─ Schema implications:
      ├─ Unique constraint: UNIQUE(product_id)
      ├─ Always load inventory with product
      ├─ Cache in Redis (key: inventory:{product_id})
      └─ Delete inventory only if product truly deleted (rare)

5. ORDERS → PAYMENTS (1:M)
   ├─ Cardinality: One order can have multiple payment attempts
   ├─ Use case: Card declined → retry → success (2+ payments)
   ├─ Implemented: payments.order_id → orders.id
   ├─ Queries:
   │  ├─ "Get payment for order X" → SELECT * FROM payments WHERE order_id = X AND status = 'succeeded'
   │  ├─ "Get last payment attempt" → SELECT * FROM payments WHERE order_id = X ORDER BY created_at DESC LIMIT 1
   │  └─ Track all attempts (failures for debugging)
   │
   └─ Schema implications:
      ├─ Unique constraint: UNIQUE(order_id, payment_method_id) prevent double-charge
      ├─ Never delete (keep failed attempts for audit)
      ├─ Index on payments(order_id, status)
      └─ Reconciliation: COUNT payments WHERE status='succeeded'

6. USERS → AUDIT_LOGS (1:M)
   ├─ Cardinality: User creates many audit log entries
   ├─ Implemented: audit_logs.user_id → users.id (nullable for system actions)
   ├─ Queries:
   │  ├─ "Show what user X did" → SELECT * FROM audit_logs WHERE user_id = X ORDER BY created_at DESC
   │  ├─ "Show entity history" → SELECT * FROM audit_logs WHERE entity_type='order' AND entity_id=X
   │  └─ Compliance: Never delete (keep forever, archive after 1 year)
   │
   └─ Schema implications:
      ├─ Composite index: (user_id, created_at)
      ├─ Composite index: (entity_type, entity_id, created_at)
      ├─ Partition by month (created_at) at scale
      └─ Archive to cold storage (S3) after 1 year
```

---

### 3.1.5 Data Validation Rules

**What to do:**
- Define constraints at database level (not just application)
- Establish business rules that must be enforced
- Plan for edge cases (nullability, defaults)
- Document why each rule exists

**Validation rules by entity:**

```
USERS TABLE:
├─ email: NOT NULL, UNIQUE, matches regex ^[^@]+@[^@]+\.[^@]+$
│  └─ Why: Email is login mechanism, must be unique
│
├─ password_hash: NOT NULL, length >= 60 (bcrypt)
│  └─ Why: Authentication security, never store plaintext
│
├─ role: NOT NULL, DEFAULT 'customer', IN ('admin', 'manager', 'employee', 'customer')
│  └─ Why: Database enforces valid roles, no application bugs
│
├─ created_at: NOT NULL, DEFAULT CURRENT_TIMESTAMP, <= now()
│  └─ Why: Audit trail, immutable once set
│
└─ email_verified: NOT NULL, DEFAULT false
   └─ Why: Don't allow login until verified (2FA if high security)

ORDERS TABLE:
├─ customer_id: NOT NULL, FK(users.id)
│  └─ Why: Every order must have customer, cascade delete would lose data
│
├─ status: NOT NULL, DEFAULT 'pending', IN ('pending', 'confirmed', 'shipped', 'delivered', 'cancelled')
│  ├─ Valid transitions (enforce in app, document here):
│  │  ├─ pending → confirmed (payment succeeded)
│  │  ├─ confirmed → shipped (warehouse shipped)
│  │  ├─ shipped → delivered (carrier delivered)
│  │  ├─ Any → cancelled (before shipment)
│  │  └─ NO retroactive changes (no status = 'pending' after shipped)
│  └─ Why: Prevents invalid state transitions
│
├─ total: NOT NULL, > 0, CHECK (total = subtotal + shipping_cost + tax)
│  ├─ Constraint: Denormalized total must equal sum of parts
│  └─ Why: Catches calculation errors at database level
│
├─ shipping_cost: NOT NULL, DEFAULT 0, >= 0
│  └─ Why: Free shipping is valid, negative cost is bug
│
├─ created_at: NOT NULL, DEFAULT NOW(), <= NOW()
│  └─ Why: Order time must be in past (no future orders)
│
└─ Order timeline invariants:
   ├─ shipped_at >= created_at (if set)
   ├─ delivered_at >= shipped_at (if set)
   ├─ cancelled_at >= created_at (if set)
   └─ Can't have delivered_at AND cancelled_at both set (business rule: choose one)

ORDER_ITEMS TABLE:
├─ order_id: NOT NULL, FK(orders.id), ON DELETE CASCADE
│  └─ Why: Item belongs to order, delete order deletes items
│
├─ product_id: NOT NULL, FK(products.id), ON DELETE RESTRICT
│  └─ Why: Item references product (historical reference, no delete)
│
├─ quantity: NOT NULL, > 0, CHECK (quantity > 0)
│  ├─ Can't have 0 quantity item
│  └─ Why: Use DELETE instead of quantity=0
│
├─ unit_price: NOT NULL, > 0, immutable after insert
│  ├─ What if product price changes? unit_price stays same
│  └─ Why: Order price locked at order time, never changes
│
└─ Composite unique constraint: UNIQUE(order_id, product_id)
   └─ Why: Can't order same product twice in same order (update quantity instead)

INVENTORY TABLE:
├─ product_id: NOT NULL, UNIQUE, FK(products.id)
│  └─ Why: Each product has exactly one inventory record
│
├─ quantity_onhand: NOT NULL, >= 0, CHECK (quantity_onhand >= 0)
│  └─ Why: Can't have negative stock (bugs should be caught)
│
├─ quantity_reserved: NOT NULL, >= 0, DEFAULT 0
│  └─ Why: Pending orders reserve inventory
│
├─ quantity_available: GENERATED ALWAYS AS (quantity_onhand - quantity_reserved)
│  ├─ Never store this, calculate always
│  └─ Why: Avoids sync issues between two fields
│
├─ reorder_point: NOT NULL, > 0
│  └─ Why: Stock level triggers re-order alert
│
└─ Constraint: quantity_available >= 0 (or alert)
   └─ Why: If available goes negative, something's wrong

PAYMENTS TABLE:
├─ order_id: NOT NULL, FK(orders.id)
│  └─ Why: Payment must belong to order
│
├─ payment_method_id: NOT NULL, FK(user_payment_methods.id)
│  └─ Why: Can't charge without payment method
│
├─ amount: NOT NULL, > 0, CHECK (amount > 0), <= order.total + 100 (cents)
│  ├─ Amount must be positive
│  ├─ Amount shouldn't exceed order total by much (100 cents = $1 overage allowed)
│  └─ Why: Catches obvious errors (negative payment, overcharge)
│
├─ status: NOT NULL, DEFAULT 'pending', IN ('pending', 'succeeded', 'failed', 'refunded')
│  └─ Valid transitions: pending→succeeded, pending→failed, succeeded→refunded
│
├─ stripe_charge_id: VARCHAR(100), UNIQUE (if set)
│  └─ Why: Stripe ID prevents double-charging same transaction
│
├─ Composite unique: UNIQUE(order_id, payment_method_id) WHERE status='succeeded'
│  └─ Why: Can't charge same payment method twice per order
│
└─ created_at must be <= NOW()
   └─ Why: Payment can't be from future

AUDIT_LOGS TABLE:
├─ user_id: UUID, nullable (system actions have no user)
│  └─ Why: Track who did action, null = system action
│
├─ action: NOT NULL, IN ('create', 'update', 'delete', 'view', 'export')
│  └─ Why: Fixed set of auditable actions
│
├─ entity_type: NOT NULL, IN ('user', 'order', 'payment', 'inventory', 'product')
│  └─ Why: Know what was modified
│
├─ old_values, new_values: JSON, both nullable
│  ├─ For CREATE: old_values is null
│  ├─ For DELETE: new_values is null
│  └─ Why: Reconstruct exact change
│
├─ ip_address: VARCHAR(45) (IPv6 support)
│  └─ Why: Track suspicious locations
│
└─ created_at: NOT NULL, DEFAULT NOW(), immutable
   └─ Why: Audit log itself must be immutable
```

**Enforcement strategy:**

```
Layer 1: Database constraints (strongest, always enforced)
├─ NOT NULL constraints
├─ UNIQUE constraints
├─ FOREIGN KEY constraints
├─ CHECK constraints (amount > 0)
├─ DEFAULT values
└─ ENUM types (status IN (...))

Layer 2: Application validation (fast feedback, before DB)
├─ Zod schema validation (request data)
├─ Type checking (TypeScript)
├─ Business logic rules (is user allowed to do this?)
└─ Format validation (email regex, phone format)

Layer 3: Triggers (rarely needed, use sparingly)
├─ Example: Update inventory.updated_at when quantity changes
├─ Example: Prevent order status change if payment not succeeded
└─ Caveat: Triggers are debugging nightmare, keep simple

Recommended approach:
├─ Database constraints: Enforce everything possible here
├─ Application validation: User-facing errors + security
├─ Triggers: Minimal (only audit trail automatic updates)
└─ Tests: Verify constraints via unit tests
```

---

## 3.2 SCHEMA DESIGN

### 3.2.1 Table Structure & Column Specifications

**Complete schema definition in Prisma format (source of truth):**

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ============================================================================
// USERS & AUTHENTICATION
// ============================================================================

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  passwordHash String
  firstName String   @db.VarChar(100)
  lastName  String   @db.VarChar(100)
  phone     String?  @db.VarChar(20)
  role      UserRole @default(CUSTOMER)
  emailVerified Boolean @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  deletedAt DateTime?

  // Relations
  orders Order[]
  addresses UserAddress[]
  paymentMethods UserPaymentMethod[]
  auditLogs AuditLog[]
  
  // Indexes
  @@index([email])
  @@index([createdAt])
}

enum UserRole {
  ADMIN
  MANAGER
  EMPLOYEE
  CUSTOMER
}

model UserAddress {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  
  type      AddressType
  line1     String   @db.VarChar(255)
  line2     String?  @db.VarChar(255)
  city      String   @db.VarChar(100)
  state     String   @db.VarChar(50)
  country   String   @db.VarChar(100)
  postalCode String  @db.VarChar(20)
  isDefault Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  ordersAsShipping Order[] @relation("shippingAddress")
  ordersAsBilling Order[] @relation("billingAddress")

  @@index([userId])
  @@index([isDefault])
}

enum AddressType {
  SHIPPING
  BILLING
  RETURN
}

model UserPaymentMethod {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  
  stripePaymentMethodId String  @unique
  cardLast4 String    @db.VarChar(4)
  cardBrand String    @db.VarChar(50)  // Visa, Mastercard, Amex
  expMonth  Int
  expYear   Int
  isDefault Boolean   @default(false)
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  deletedAt DateTime?

  // Relations
  payments Payment[]

  @@index([userId])
  @@index([isDefault])
  @@unique([userId, stripePaymentMethodId])
}

// ============================================================================
// ORDERS & FULFILLMENT
// ============================================================================

model Order {
  id        String   @id @default(cuid())
  customerId String
  customer  User     @relation(fields: [customerId], references: [id])
  
  orderNumber String  @unique @db.VarChar(50)
  status    OrderStatus @default(PENDING)
  
  // Amounts (in cents, always BIGINT)
  subtotal  BigInt
  shippingCost BigInt @default(0)
  tax       BigInt @default(0)
  total     BigInt
  
  // Addresses
  shippingAddressId String
  shippingAddress UserAddress @relation("shippingAddress", fields: [shippingAddressId], references: [id])
  
  billingAddressId String
  billingAddress UserAddress @relation("billingAddress", fields: [billingAddressId], references: [id])
  
  // Timeline
  createdAt DateTime  @default(now())
  shippedAt DateTime?
  deliveredAt DateTime?
  cancelledAt DateTime?
  updatedAt DateTime @updatedAt

  // Relations
  items     OrderItem[]
  payments  Payment[]
  shipments Shipment[]
  auditLogs AuditLog[]

  // Constraints
  @@check("total = subtotal + shippingCost + tax")
  @@check("total > 0")
  @@check("subtotal > 0")
  @@check("shippingCost >= 0")
  @@check("tax >= 0")
  @@check("shippedAt IS NULL OR shippedAt >= createdAt")
  @@check("deliveredAt IS NULL OR deliveredAt >= shippedAt")
  @@check("cancelledAt IS NULL OR cancelledAt >= createdAt")
  
  @@index([customerId])
  @@index([status])
  @@index([createdAt])
  @@index([orderNumber])
  @@index([customerId, createdAt])
}

enum OrderStatus {
  PENDING
  CONFIRMED
  SHIPPED
  DELIVERED
  CANCELLED
}

model OrderItem {
  id        String   @id @default(cuid())
  orderId   String
  order     Order    @relation(fields: [orderId], references: [id], onDelete: Cascade)
  
  productId String
  product   Product  @relation(fields: [productId], references: [id])
  
  quantity  Int
  unitPrice BigInt  // Snapshot at order time, never changes
  lineTotal BigInt  // Calculated: quantity * unitPrice
  
  createdAt DateTime @default(now())

  @@unique([orderId, productId])
  @@check("quantity > 0")
  @@check("unitPrice > 0")
  @@check("lineTotal > 0")
  @@index([orderId])
  @@index([productId])
}

model Payment {
  id        String   @id @default(cuid())
  orderId   String
  order     Order    @relation(fields: [orderId], references: [id])
  
  paymentMethodId String
  paymentMethod UserPaymentMethod @relation(fields: [paymentMethodId], references: [id])
  
  amount    BigInt
  status    PaymentStatus @default(PENDING)
  stripeChargeId String?  @unique
  failureReason String?
  
  attemptedAt DateTime @default(now())
  createdAt   DateTime @default(now())

  @@unique([orderId, paymentMethodId])
  @@check("amount > 0")
  @@check("amount <= 99999999")  // $999,999.99 cap
  @@index([orderId])
  @@index([status])
  @@index([stripeChargeId])
}

enum PaymentStatus {
  PENDING
  SUCCEEDED
  FAILED
  REFUNDED
}

model Shipment {
  id        String   @id @default(cuid())
  orderId   String   @unique
  order     Order    @relation(fields: [orderId], references: [id])
  
  carrier   Carrier
  trackingNumber String @unique @db.VarChar(100)
  labelUrl  String
  estimatedDelivery DateTime?
  
  shippedAt DateTime @default(now())
  deliveredAt DateTime?
  createdAt DateTime @default(now())

  @@index([orderId])
  @@index([trackingNumber])
}

enum Carrier {
  USPS
  FEDEX
  UPS
}

// ============================================================================
// PRODUCTS & INVENTORY
// ============================================================================

model Product {
  id        String   @id @default(cuid())
  sku       String   @unique
  name      String
  description String?
  category  String   @db.VarChar(100)
  
  // Amounts (in cents)
  price     BigInt   // Retail price
  cost      BigInt?  // Cost for margin calculation
  
  imageUrl  String?
  isActive  Boolean  @default(true)  // Soft delete
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  orderItems OrderItem[]
  inventory  Inventory?
  transactions InventoryTransaction[]
  auditLogs  AuditLog[]

  @@index([sku])
  @@index([category])
  @@index([isActive])
  @@index([createdAt])
}

model Inventory {
  id        String   @id @default(cuid())
  productId String   @unique
  product   Product  @relation(fields: [productId], references: [id])
  
  quantityOnHand Int
  quantityReserved Int @default(0)
  quantityAvailable Int // Generated: quantityOnHand - quantityReserved
  
  reorderPoint Int
  warehouseLocation String?
  
  updatedAt DateTime @updatedAt
  lastCountedAt DateTime?

  // Relations
  transactions InventoryTransaction[]

  @@check("quantityOnHand >= 0")
  @@check("quantityReserved >= 0")
  @@check("reorderPoint > 0")
  @@index([quantityAvailable])
}

model InventoryTransaction {
  id        String   @id @default(cuid())
  productId String
  product   Product  @relation(fields: [productId], references: [id])
  
  orderId   String?  // Nullable, some transactions not order-related
  order     Order?   @relation(fields: [orderId], references: [id])
  
  type      TransactionType
  quantityChange Int  // Positive or negative
  notes     String?
  createdBy String
  createdAt DateTime @default(now())

  @@index([productId])
  @@index([orderId])
  @@index([type])
  @@index([createdAt])
}

enum TransactionType {
  PURCHASE
  SALE
  ADJUSTMENT
  COUNT_VARIANCE
  RETURN
}

// ============================================================================
// AUDITING
// ============================================================================

model AuditLog {
  id        String   @id @default(cuid())
  userId    String?  // Nullable for system actions
  user      User?    @relation(fields: [userId], references: [id])
  
  entityType AuditEntityType
  entityId  String
  action    AuditAction
  
  oldValues Json?   // Previous values
  newValues Json?   // New values
  
  ipAddress String?
  userAgent String?
  
  createdAt DateTime @default(now())

  @@index([userId])
  @@index([createdAt])
  @@index([entityType, entityId])
  @@index([userId, createdAt])
}

enum AuditEntityType {
  USER
  ORDER
  PAYMENT
  PRODUCT
  INVENTORY
}

enum AuditAction {
  CREATE
  UPDATE
  DELETE
  VIEW
  EXPORT
}
```

---

### 3.2.2 Primary Key & Foreign Key Strategy

**Primary Key (PK) Design:**

```
Decision: Use UUIDs (UUID v4) instead of auto-increment INT

Why UUIDs:
├─ Security: No sequential ID enumeration (can't guess next order ID)
├─ Distributed: Works across multiple databases (sharding-ready)
├─ Immutable: Generated client-side, never changes
├─ Database-agnostic: Same ID format in PostgreSQL, MongoDB, DynamoDB
└─ Scalability: Prepared for global distribution

Trade-off:
├─ Size: 16 bytes vs 8 bytes for BIGINT (2x larger)
├─ Speed: UUID generation slightly slower than auto-increment
├─ Indexing: Slightly more disk space for indexes
└─ Worth it: Security + future scalability > small storage cost

Alternative rejected: Snowflake IDs
├─ Use case: Distributed ID generation (Twitter-like scale)
├─ Our case: Single database initially, UUID sufficient
└─ Revisit at Year 2 if sharding needed

Implementation:
├─ Prisma: @default(cuid()) generates IDs automatically
├─ Format: 25-character string (CUID, like uuid but shorter)
├─ Or: @default(uuid()) for standard UUID format
└─ Recommendation: Use CUID (smaller URLs, same security)
```

**Foreign Key (FK) Strategy:**

```
Decision: Always use explicit FK constraints

Why enforce FKs at database level:
├─ Referential integrity: Can't create order without customer
├─ Cascade delete: Delete user → automatically delete orders
├─ Automatic: Database prevents invalid states, app can't bypass
├─ Audit trail: Schema documents what references what
└─ Performance: Database can optimize joins, knows relationship

On Delete / On Update Strategy:

1. CASCADE (for dependent entities)
   ├─ orders.customer_id → users.id (NO CASCADE, keep history)
   ├─ order_items.order_id → orders.id (CASCADE: delete items when order deleted)
   ├─ inventory.product_id → products.id (NO CASCADE, keep as reference)
   └─ Rule: Only CASCADE if child can't exist without parent

2. RESTRICT (for reference data)
   ├─ order_items.product_id → products.id (RESTRICT: can't delete product with orders)
   ├─ inventory_transactions.product_id → products.id (RESTRICT)
   └─ Rule: If deletion would lose data, RESTRICT and make user delete references first

3. SET NULL (rarely used)
   ├─ Use case: Optional parent (department could be deleted, employee survives)
   ├─ Our system: Never used (all FKs are NOT NULL)
   └─ Don't use: Loses information, bad audit trail

4. NO ACTION (same as RESTRICT, with different timing)
   ├─ Database waits until after foreign key check
   ├─ Practically same as RESTRICT
   └─ Use RESTRICT (clearer intention)

Foreign Key Constraints in Prisma:
├─ @relation(fields: [customerId], references: [id])
├─ Automatically creates FK constraint in database
├─ Cascade on delete: onDelete: Cascade
├─ Other options: Restrict, SetNull, NoAction
└─ Default (no option): RESTRICT

Foreign Keys Across Multiple Relationships:
├─ Problem: Product referenced from order_items AND product table
├─ Solution: Use @relation("name") to disambiguate
├─ Example:
│  ├─ OrderItem references Product (for current info)
│  ├─ Order item stores historical unit_price (never changes)
│  └─ No FK constraint on unit_price (it's immutable)
│
└─ Rule: Store historical data separately, never try to update it

Critical: Never Denormalize by Breaking FKs
├─ ❌ WRONG: Copy customer email to orders table
├─ ❌ WRONG: Copy product price to order_items instead of storing unit_price
├─ ❌ WRONG: Store user role in orders (user_role should be looked up from users table)
├─ ✅ RIGHT: Store references (IDs), join when needed
├─ ✅ RIGHT: Store immutable snapshots (unit_price at order time)
└─ Rule: Denormalize only calculated/cache fields, never live references
```

---

### 3.2.3 Index Planning & Performance

**What to do:**
- Identify every query pattern
- Create indexes for WHERE, ORDER BY, JOIN clauses
- Avoid index bloat (too many indexes = slower writes)
- Plan for growth (what queries will be slow at 1M rows?)

**Index strategy for high-performance queries:**

```
Indexing Decision Tree:

For every table, ask:
1. What's the most frequent query?
2. What's the largest query result set?
3. What's the hardest query (most JOINs)?
4. What grows fastest (ORDER BY, pagination)?

USERS TABLE - Primary Queries:
├─ Query 1: "Get user by email"
│  ├─ SELECT * FROM users WHERE email = ?
│  ├─ Index needed: CREATE INDEX users_email ON users(email)
│  ├─ Type: UNIQUE (email is unique key, must be enforced)
│  └─ Frequency: EVERY login + password reset + account lookup
│
├─ Query 2: "Get users created today"
│  ├─ SELECT * FROM users WHERE created_at > NOW() - INTERVAL 1 day
│  ├─ Index needed: CREATE INDEX users_created_at ON users(created_at)
│  └─ Frequency: Daily (analytics, reports)
│
└─ Index Summary:
   ├─ users(email) - UNIQUE
   ├─ users(created_at)
   ├─ users(role) - if filtering by role (admin dashboard)
   └─ Total: 3-4 indexes on USERS table

ORDERS TABLE - Primary Queries:
├─ Query 1: "Get user's orders"
│  ├─ SELECT * FROM orders WHERE customer_id = ? ORDER BY created_at DESC
│  ├─ Index needed: CREATE INDEX orders_customer_created ON orders(customer_id, created_at)
│  ├─ Type: Composite (customer_id + created_at for sorting)
│  └─ Frequency: VERY HIGH (user dashboard, every session)
│
├─ Query 2: "Get orders by status"
│  ├─ SELECT * FROM orders WHERE status = 'SHIPPED'
│  ├─ Index needed: CREATE INDEX orders_status ON orders(status)
│  └─ Frequency: HIGH (admin dashboard, monitoring)
│
├─ Query 3: "Get order by order_number"
│  ├─ SELECT * FROM orders WHERE order_number = ?
│  ├─ Index needed: CREATE UNIQUE INDEX orders_order_number ON orders(order_number)
│  ├─ Type: UNIQUE (order_number is unique)
│  └─ Frequency: VERY HIGH (tracking lookups)
│
├─ Query 4: "Get all orders created in last hour"
│  ├─ SELECT COUNT(*) FROM orders WHERE created_at > NOW() - INTERVAL 1 hour
│  ├─ Index needed: Already covered by orders(customer_id, created_at)
│  └─ Frequency: MEDIUM (monitoring, alerts)
│
└─ Index Summary:
   ├─ orders(customer_id, created_at) - COMPOSITE
   ├─ orders(status)
   ├─ orders_order_number UNIQUE
   ├─ Created_at usually implicit (many queries filter by it)
   └─ Total: 3-4 indexes on ORDERS table

ORDER_ITEMS TABLE - Primary Queries:
├─ Query 1: "Get all items in order"
│  ├─ SELECT * FROM order_items WHERE order_id = ?
│  ├─ Index needed: CREATE INDEX order_items_order_id ON order_items(order_id)
│  └─ Frequency: VERY HIGH (almost every order lookup)
│
├─ Query 2: "Get all orders for product"
│  ├─ SELECT COUNT(*) FROM order_items WHERE product_id = ?
│  ├─ Index needed: CREATE INDEX order_items_product_id ON order_items(product_id)
│  └─ Frequency: MEDIUM (product analytics)
│
└─ Index Summary:
   ├─ order_items(order_id)
   ├─ order_items(product_id)
   └─ Total: 2 indexes on ORDER_ITEMS table

INVENTORY TABLE - Primary Queries:
├─ Query 1: "Check stock before order"
│  ├─ SELECT quantity_available FROM inventory WHERE product_id = ?
│  ├─ Index needed: UNIQUE(product_id) - already PKs
│  ├─ Type: Primary key lookup (fastest)
│  └─ Frequency: EXTREME (every order creation)
│
├─ Query 2: "Get low stock products"
│  ├─ SELECT * FROM inventory WHERE quantity_available < reorder_point
│  ├─ Index needed: CREATE INDEX inventory_quantity_available ON inventory(quantity_available)
│  └─ Frequency: MEDIUM (daily, stock alerts)
│
└─ Index Strategy:
   ├─ Inventory is HOT table (very frequent access)
   ├─ Cache in Redis: KEY inventory:{product_id}
   ├─ Only query DB on cache miss
   ├─ Update cache after every inventory change
   └─ Database index less critical (served from cache)

PAYMENTS TABLE - Primary Queries:
├─ Query 1: "Get order's successful payment"
│  ├─ SELECT * FROM payments WHERE order_id = ? AND status = 'SUCCEEDED'
│  ├─ Index needed: CREATE INDEX payments_order_status ON payments(order_id, status)
│  └─ Frequency: HIGH (payment confirmation, reconciliation)
│
├─ Query 2: "Find failed payments"
│  ├─ SELECT * FROM payments WHERE status = 'FAILED' ORDER BY created_at DESC
│  ├─ Index needed: CREATE INDEX payments_status_created ON payments(status, created_at)
│  └─ Frequency: MEDIUM (daily payment reconciliation)
│
└─ Index Summary:
   ├─ payments(order_id, status)
   ├─ payments(status, created_at)
   ├─ payments(stripe_charge_id) UNIQUE
   └─ Total: 3 indexes on PAYMENTS table

AUDIT_LOGS TABLE - Special Considerations:
├─ Growth: Millions of rows (one per action)
├─ Queries:
│  ├─ "Show what user_id=X did" → Index: (user_id, created_at)
│  ├─ "Show history of entity_id=Y" → Index: (entity_type, entity_id, created_at)
│  └─ "Get logs in time range" → Index: created_at
│
├─ Strategy:
│  ├─ PARTITION by month (created_at) at scale
│  ├─ Archive old partitions to cold storage (S3)
│  ├─ Never delete (compliance requirement)
│  └─ Example: audit_logs_2024_01, audit_logs_2024_02, ...
│
└─ Index Summary:
   ├─ audit_logs(user_id, created_at)
   ├─ audit_logs(entity_type, entity_id, created_at)
   └─ Total: 2 indexes (plus partition pruning)

TOTAL INDEXES ACROSS SCHEMA:
├─ USERS: 3-4 indexes
├─ ORDERS: 3-4 indexes
├─ ORDER_ITEMS: 2 indexes
├─ INVENTORY: 1 index (+ Redis cache)
├─ PAYMENTS: 3 indexes
├─ PRODUCTS: 4 indexes
├─ AUDIT_LOGS: 2 indexes (+ partitioning)
└─ TOTAL: ~20-22 indexes across entire schema

Index Maintenance:
├─ Monitor: ANALYZE TABLE (weekly)
├─ Rebuild: REINDEX (monthly, only if needed)
├─ Unused indexes: SELECT pg_stat_user_indexes to find unused ones
├─ Too many writes: If write load > read load, consider removing indexes
└─ Growth: Monitor index size (they grow with data)
```

**Index Rules to Never Break:**

```
1. Index WHERE clauses
   ├─ If you write "WHERE x = ?", index x
   └─ If you write "WHERE x > ? AND y < ?", index (x, y)

2. Composite indexes follow query column order
   ├─ Query: WHERE customer_id = ? ORDER BY created_at
   ├─ Index: (customer_id, created_at) ← same order
   ├─ Index: (created_at, customer_id) ← WRONG, less optimal

3. UNIQUE constraints are auto-indexed
   ├─ @unique email → automatically indexed
   ├─ @unique([orderId, productId]) → composite index automatically
   └─ Don't create redundant indexes

4. Foreign keys can benefit from indexes
   ├─ JOINs on FK columns use indexes
   ├─ Example: JOIN orders o ON o.customer_id = u.id
   ├─ Index needed: orders(customer_id)
   └─ Prisma creates these automatically (usually)

5. Never over-index
   ├─ Every index slows down INSERT/UPDATE/DELETE
   ├─ If table has 10+ indexes, something's wrong
   ├─ Measure impact: Enable slow query log, see actual usage
   └─ Remove unused indexes (they waste disk space)

6. Partial indexes for large tables
   ├─ Only index active rows: WHERE deleted_at IS NULL
   ├─ Only index relevant status: WHERE status IN ('PENDING', 'CONFIRMED')
   ├─ Example: CREATE INDEX orders_active ON orders(customer_id) WHERE deleted_at IS NULL
   └─ Reduces index size significantly

7. Multi-column indexes are different from separate indexes
   ├─ ❌ WRONG: Index A separately, Index B separately
   ├─ ✅ RIGHT: Composite index (A, B) for query using both
   ├─ Exception: If queries filter on B alone, need separate index on B
   └─ Rule: Composite indexes can't be used to find just the second column
```

---

### 3.2.4 Normalization vs Denormalization

**When to normalize, when to denormalize:**

```
NORMALIZATION (Normal Form Analysis):

First Normal Form (1NF): Atomic values only
├─ Every column contains single value, not list
├─ ❌ WRONG: order.item_ids = '1,2,3' (comma-separated)
├─ ✅ RIGHT: order_items table with multiple rows
└─ Our schema: Already in 1NF

Second Normal Form (2NF): No partial dependencies
├─ Non-key attributes depend on full PK, not part of it
├─ ❌ WRONG: (order_id, product_id) → product_name (depends on product_id alone)
├─ ✅ RIGHT: Store product_id in order_items, join to products for name
└─ Our schema: Mostly 2NF

Third Normal Form (3NF): No transitive dependencies
├─ Non-key attributes depend only on PK, not on other non-key attributes
├─ ❌ WRONG: orders(customer_id, customer_email) - email depends on customer_id
├─ ✅ RIGHT: Look up customer_email from users table when needed
└─ Our schema: Follows 3NF (minimal redundancy)

Boyce-Codd Normal Form (BCNF): No anomalies
├─ Every determinant is a candidate key
├─ Practical for most business apps (stricter than 3NF)
├─ Trade-off: More JOINs required, but data integrity guaranteed
└─ Our schema: BCNF for most tables

DENORMALIZATION (Strategic Redundancy):

When to denormalize (ONLY after profiling shows bottleneck):

1. Total Price in Orders Table
   ├─ Normalized: total = SUM(order_items.line_total)
   ├─ Denormalized: orders.total (stored, not calculated)
   ├─ Why: Users want order total instantly (no JOIN + SUM)
   ├─ Sync strategy: UPDATE orders SET total = ... whenever order_items change
   ├─ Validation: CHECK (total = subtotal + shipping_cost + tax)
   └─ Risk: Sync failures (code that updates items but forgets total)

2. Inventory Quantities in Inventory Table
   ├─ Quantity_available = quantity_onhand - quantity_reserved
   ├─ Generated: GENERATED ALWAYS AS (quantity_onhand - quantity_reserved)
   ├─ Why: Cache this calculation (never calculate it)
   ├─ Sync strategy: Database generates automatically
   └─ Risk: None (database guarantees consistency)

3. Cache Redis Keys for Frequently Accessed Data
   ├─ Normalized DB: User needs to JOIN 5 tables to get info
   ├─ Denormalized cache: Redis stores pre-joined data
   ├─ Example: cache:user:{id} = {user object with addresses, payment methods}
   ├─ Sync strategy: Invalidate cache when any related table changes
   └─ Risk: Cache inconsistency (code updates DB but forgets cache)

4. Historical Snapshots (unit_price in order_items)
   ├─ Why: Product price changes, but order_items.unit_price never changes
   ├─ This is NOT normalization violation (it's a snapshot)
   ├─ Different from: "Copy current price to avoid JOIN"
   └─ Historical data is valid denormalization

Denormalization Guidelines (Follow Strictly):

✅ Denormalize when:
├─ You've profiled the query and it's a bottleneck
├─ The calculation is expensive (SUM, COUNT, complex logic)
├─ Data changes infrequently (price changes less often than queries)
├─ You have a clear sync mechanism (trigger, app code, cache)
└─ You've added validation (CHECK constraints) to catch sync failures

❌ Never denormalize:
├─ "Just in case" (without profiling)
├─ Live references (customer_email should stay in users table)
├─ Complex calculations (if formula changes, update is nightmare)
├─ When you can cache instead (Redis is better than DB denormalization)
└─ Without validation (sync failures cause data corruption)

Our Schema Denormalization Strategy:

1. Order.total ← DENORMALIZED (required for performance)
   ├─ Validation: CHECK (total = subtotal + shipping_cost + tax)
   ├─ Update trigger: Fires whenever subtotal, shipping_cost, or tax change
   ├─ Never manually set: Always calculated
   └─ Risk mitigation: Nightly audit to verify total consistency

2. Inventory.quantity_available ← DENORMALIZED (safe)
   ├─ Type: GENERATED ALWAYS (database guarantees consistency)
   ├─ Never manually updated
   ├─ Used only for: Reads (checking stock available)
   └─ Zero risk: Database maintains it

3. Unit_price in order_items ← NOT DENORMALIZATION (historical snapshot)
   ├─ Immutable after order created
   ├─ Never updated
   ├─ Different from products.price
   └─ This is correct design

4. NO customer_email in orders ← Stays normalized
   ├─ ❌ Don't copy: orders.customer_email
   ├─ ✅ Look up: JOIN orders o INNER JOIN users u ON u.id = o.customer_id
   ├─ Why: Email can change, would need sync
   └─ Performance: Redis cache if needed

5. NO product_name in order_items ← Stays normalized
   ├─ ❌ Don't copy: order_items.product_name
   ├─ ✅ Look up: JOIN order_items oi INNER JOIN products p ON p.id = oi.product_id
   ├─ Why: Product name changes (historical accuracy important)
   └─ Performance: Products table is small, JOIN is fast
```

---

## 3.3 DATA INTEGRITY

### 3.3.1 Constraint Definitions

**Database-level constraints that prevent invalid data:**

```
CONSTRAINT TYPES:

1. NOT NULL Constraints
   ├─ Definition: Column must have value (can't be NULL)
   ├─ Examples:
   │  ├─ users.email NOT NULL
   │  ├─ orders.customer_id NOT NULL
   │  └─ products.name NOT NULL
   ├─ When to use:
   │  ├─ Always for required business data
   │  ├─ Never for optional data (use NULL for missing)
   │  └─ Think: "Does this entity exist without this attribute?"
   └─ Test:
      ├─ INSERT INTO users(name) VALUES ('John') -- error if email NOT NULL
      └─ Good: Application can't accidentally create incomplete records

2. UNIQUE Constraints
   ├─ Definition: Column value must be unique across table
   ├─ Examples:
   │  ├─ users.email UNIQUE
   │  ├─ orders.order_number UNIQUE
   │  ├─ products.sku UNIQUE
   │  └─ Composite: UNIQUE(user_id, product_id) - no duplicate reviews per user
   ├─ NULL handling:
   │  ├─ Multiple NULLs allowed (NULL != NULL in databases)
   │  ├─ Use UNIQUE WHERE column IS NOT NULL for conditional uniqueness
   │  └─ Example: UNIQUE(deleted_at, email) - allows multiple NULLs (active users)
   └─ Test:
      ├─ INSERT INTO users(email) VALUES ('john@example.com')
      ├─ INSERT INTO users(email) VALUES ('john@example.com') -- error
      └─ Good: Prevents duplicate emails, database enforces

3. PRIMARY KEY Constraints
   ├─ Definition: Unique identifier for each row (UNIQUE + NOT NULL)
   ├─ Examples:
   │  ├─ id UUID PRIMARY KEY
   │  ├─ Composite: PRIMARY KEY(user_id, product_id) for junction tables
   ├─ Important:
   │  ├─ Only ONE PK per table (Composite PKs are multiple columns)
   │  ├─ Database indexes PK automatically
   │  ├─ FK constraints reference PK
   │  └─ Every table MUST have a PK (no exceptions)
   └─ Our schema:
      ├─ Single PKs: id UUID for all tables (100% recommended)
      ├─ Never composite PKs (keep IDs simple)
      └─ Natural keys (email, sku) are UNIQUE, not PK

4. FOREIGN KEY Constraints
   ├─ Definition: Column value must reference existing row in another table
   ├─ Examples:
   │  ├─ orders.customer_id REFERENCES users(id)
   │  ├─ order_items.product_id REFERENCES products(id)
   │  └─ payments.order_id REFERENCES orders(id)
   ├─ Actions on deletion:
   │  ├─ CASCADE: Delete parent → child deleted automatically
   │  ├─ RESTRICT: Delete parent → error if children exist
   │  ├─ SET NULL: Delete parent → child FK set to NULL
   │  └─ NO ACTION: Same as RESTRICT (timing difference)
   └─ Our strategy:
      ├─ orders.customer_id → users.id (RESTRICT: keep order history)
      ├─ order_items.order_id → orders.id (CASCADE: delete items with order)
      ├─ inventory.product_id → products.id (RESTRICT: keep for reporting)
      └─ payments.order_id → orders.id (RESTRICT: audit trail)

5. CHECK Constraints
   ├─ Definition: Value must satisfy boolean expression
   ├─ Examples:
   │  ├─ CHECK (total > 0) - order total must be positive
   │  ├─ CHECK (quantity >= 0) - stock can't be negative
   │  ├─ CHECK (exp_month BETWEEN 1 AND 12) - valid month
   │  ├─ CHECK (total = subtotal + shipping_cost + tax) - calculated field validation
   │  └─ CHECK (delivered_at >= shipped_at) - timeline must be valid
   ├─ Composite checks:
   │  ├─ CHECK (status IN ('pending', 'confirmed', 'shipped'))
   │  ├─ CHECK (email LIKE '%@%.%') - email format
   │  └─ CHECK (created_at <= NOW()) - can't create in future
   └─ Test:
      ├─ INSERT INTO orders(total) VALUES (-100) -- error
      └─ Good: Database prevents invalid state (not just app)

6. DEFAULT Values
   ├─ Definition: Value assigned if not provided in INSERT
   ├─ Examples:
   │  ├─ status DEFAULT 'pending' - new orders are pending
   │  ├─ created_at DEFAULT NOW() - timestamp current time
   │  ├─ is_active DEFAULT true - products active by default
   │  └─ quantity_reserved DEFAULT 0 - inventory starts empty
   ├─ Functions:
   │  ├─ NOW() - current timestamp
   │  ├─ CURRENT_TIMESTAMP - same as NOW()
   │  ├─ CURRENT_DATE - today's date
   │  ├─ uuid_generate_v4() - random UUID
   │  └─ gen_random_uuid() - PostgreSQL specific
   └─ Test:
      ├─ INSERT INTO orders(customer_id, total) VALUES (uuid, 1000)
      ├─ SELECT status FROM orders -- returns 'pending' (DEFAULT applied)
      └─ Good: Can't accidentally create incomplete orders

7. ENUM Constraints
   ├─ Definition: Column can only hold one of predefined values
   ├─ Examples:
   │  ├─ status ENUM('pending', 'confirmed', 'shipped')
   │  ├─ role ENUM('admin', 'manager', 'employee', 'customer')
   │  ├─ carrier ENUM('usps', 'fedex', 'ups')
   │  └─ action ENUM('create', 'update', 'delete', 'view')
   ├─ Advantages:
   │  ├─ Database enforces valid values (typos caught)
   │  ├─ Storage efficient (1-2 bytes vs VARCHAR)
   │  ├─ Query optimization (comparison is integer, not string)
   │  └─ Documentation (valid values explicit in schema)
   └─ Caution:
      ├─ Adding new enum value requires ALTER TABLE (migration)
      ├─ Removing value requires UPDATE all rows first
      ├─ Easier to use CHECK (status IN (...)) if values change frequently
      └─ Our approach: ENUM for stable values, CHECK for changing ones
```

---

### 3.3.2 Referential Integrity & Cascade Rules

**How to handle related data when parent changes:**

```
CASCADE DELETE (Delete parent → delete children automatically)

When to use CASCADE:
├─ Dependent entities: Can't exist without parent
├─ Examples:
│  ├─ Delete order → DELETE order_items (items depend on order)
│  ├─ Delete invoice → DELETE invoice_items (items depend on invoice)
│  └─ Delete shipment → DELETE tracking_events (events depend on shipment)
├─ Safe if: Child entity has no other use
└─ Test:
   ├─ DELETE FROM orders WHERE id = ? -- order_items automatically deleted
   ├─ SELECT * FROM order_items WHERE order_id = ? -- returns 0 rows
   └─ Good: No orphaned items

RESTRICT DELETE (Delete parent → error if children exist)

When to use RESTRICT:
├─ Referenced entities: Might be referenced by other things
├─ Examples:
│  ├─ Delete user → ERROR if user has orders (keep history)
│  ├─ Delete product → ERROR if product in orders (keep for reporting)
│  ├─ Delete payment method → ERROR if used in payments (audit trail)
│  └─ Delete inventory → ERROR if inventory_transactions reference it
├─ Process: Delete children first, then parent
│  ├─ SELECT COUNT(*) FROM orders WHERE customer_id = ? -- check count
│  ├─ If count > 0: Can't delete user, show error
│  └─ Or: Delete orders first (if safe), then delete user
└─ Test:
   ├─ DELETE FROM users WHERE id = ? -- ERROR if customer has orders
   ├─ Delete orders first, then user succeeds
   └─ Good: Prevents data loss from accidental deletion

SET NULL DELETE (Delete parent → child FK set to NULL)

When to use SET NULL:
├─ Optional parent: Parent reference is optional
├─ Examples:
│  ├─ Delete department → employees.department_id = NULL
│  ├─ Delete manager → employees.manager_id = NULL
│  └─ Delete territory → accounts.territory_id = NULL
├─ Caution: Loses information (which department did employee belong to?)
└─ Our system: DON'T USE
   ├─ Keep all FKs NOT NULL (every entity must belong to something)
   └─ Use RESTRICT to enforce referential integrity

SOFT DELETES (Never physically delete)

Strategy for all business entities:
├─ Add deleted_at TIMESTAMP NULL to every table
├─ UPDATE deleted_at = NOW() instead of DELETE
├─ Add WHERE deleted_at IS NULL to every query
├─ Examples:
│  ├─ users(id, email, deleted_at)
│  ├─ products(id, sku, deleted_at)
│  ├─ orders(id, customer_id, deleted_at)
│  └─ payments(id, order_id, deleted_at)
├─ Benefits:
│  ├─ Historical data preserved (analytics, auditing, debugging)
│  ├─ Recover deleted data (set deleted_at = NULL)
│  ├─ Never lose information (compliance requirement)
│  ├─ Audit trail complete (know when things deleted)
│  └─ Foreign key constraint not affected (deleted FK still valid)
├─ Queries MUST filter:
│  ├─ ❌ WRONG: SELECT * FROM users WHERE id = ?
│  ├─ ✅ RIGHT: SELECT * FROM users WHERE id = ? AND deleted_at IS NULL
│  ├─ ❌ WRONG: SELECT * FROM orders WHERE customer_id = ?
│  └─ ✅ RIGHT: SELECT * FROM orders WHERE customer_id = ? AND deleted_at IS NULL
└─ Database view for convenience:
   ├─ CREATE VIEW users_active AS SELECT * FROM users WHERE deleted_at IS NULL
   ├─ Then: SELECT * FROM users_active WHERE id = ? (simpler queries)
   └─ Or: Use Prisma filters (automatic deleted_at filtering)

CASCADE DELETE + Soft Deletes (Best of Both Worlds):

Use case: Order with items
├─ Order has foreign key constraint to Customer (RESTRICT)
├─ Order_items has foreign key constraint to Order (CASCADE)
├─ Question: What happens when delete Order?
├─ Answer:
│  ├─ Instead of DELETE: UPDATE orders SET deleted_at = NOW()
│  ├─ Soft delete: No CASCADE triggered (because DELETE never executed)
│  ├─ Order_items stay intact (not deleted)
│  ├─ Data preserved for auditing
│  └─ But if you REALLY delete order: CASCADE deletes items
├─ Process:
│  ├─ For normal operations: Use soft delete (UPDATE deleted_at)
│  ├─ For hard delete (rare): Database CASCADE delete child rows
│  └─ Queries filter deleted_at IS NULL automatically
└─ Recommendation: Soft delete everything, CASCADE delete is backup

Our Referential Integrity Strategy:

┌─────────────────────────────────────────────────────────────┐
│ ENTITY RELATIONSHIP    │ DELETE BEHAVIOR    │ RATIONALE        │
├─────────────────────────────────────────────────────────────┤
│ User → Order           │ RESTRICT           │ Keep history     │
│ Order → OrderItem      │ CASCADE            │ Item needs order │
│ Product → OrderItem    │ RESTRICT           │ Keep for report  │
│ OrderItem → Product    │ RESTRICT           │ Don't orphan item│
│ Order → Payment        │ RESTRICT           │ Keep audit trail │
│ User → UserAddress     │ CASCADE            │ Address needs usr│
│ User → PaymentMethod   │ CASCADE            │ Card needs user  │
│ Product → Inventory    │ RESTRICT           │ Keep stock data  │
│ Product → InventoryTx  │ RESTRICT           │ Keep history     │
└─────────────────────────────────────────────────────────────┘

But all use soft deletes (UPDATE deleted_at IS NULL):
├─ Physical DELETE rare (only for data cleanup)
├─ CASCADE only executes if physical DELETE used
├─ Normal operations: Soft delete preserves everything
└─ This is production best practice
```

---

## 3.4 MIGRATION STRATEGY

### 3.4.1 Migration Tool Selection & Approach

**Using Prisma Migrate (recommended for our stack):**

```
WHY PRISMA MIGRATE:

Advantages:
├─ Single source of truth: Prisma schema.prisma
├─ Type-safe: TypeScript checks before migration
├─ Automated migrations: prisma migrate dev (auto-creates .sql files)
├─ Rollback support: Undo migrations with prisma migrate resolve
├─ Shadow database: Test migrations safely before running
├─ Reset capability: prisma migrate reset (dev only)
└─ Seamless with Prisma ORM: No separate query language

Alternatives considered:
├─ Knex: Manual SQL migrations (more control, more work)
├─ Alembic: Python tool (don't use non-JS tool with JS stack)
├─ Sequelize: Built-in migrations (less type-safe than Prisma)
└─ Flyway: Cross-platform (overkill for our needs)

DECISION: Use Prisma Migrate exclusively

Prisma Migrate Workflow:

1. Update schema.prisma with new model or field
   ├─ Example: Add email_verified to users
   └─ File: prisma/schema.prisma

2. Create migration
   ├─ Command: npx prisma migrate dev --name add_email_verified_to_users
   ├─ Creates: prisma/migrations/20240115_add_email_verified_to_users/migration.sql
   ├─ Applies: Migration automatically runs
   └─ Tracks: prisma_migrations table records applied migrations

3. Migration file review (ALWAYS review auto-generated SQL)
   ├─ File path: prisma/migrations/20240115_add_email_verified_to_users/migration.sql
   ├─ Example content:
   │  ├─ ALTER TABLE users ADD COLUMN email_verified BOOLEAN NOT NULL DEFAULT false;
   │  └─ Optional: Add indexes, constraints
   ├─ Good: Migration is reversible (add column can be removed)
   ├─ Bad: Data transformations are not reversible (require two-step migration)
   └─ Review: Ensure no data loss

4. Commit to Git
   ├─ All migration files tracked in Git
   ├─ Team sees exact schema changes
   ├─ CI/CD can validate migrations
   └─ Rollback: Revert Git commit, re-run migrations

5. Deploy to production
   ├─ Run: npx prisma migrate deploy (in production)
   ├─ Applies: Only unapplied migrations
   ├─ Idempotent: Safe to run multiple times
   └─ Faster: No down-time (if migration is additive)

Example: Adding a Required Field with Default

Step 1: Naive approach (BAD - can fail)
├─ schema.prisma: Add new field as NOT NULL (no DEFAULT)
├─ Problem: Existing rows don't have value, can't add NOT NULL column
├─ Error: "Existing columns are not compatible with NOT NULL"
└─ Result: Migration fails

Step 2: Correct approach (GOOD - two-step)
├─ Migration 1: Add column with DEFAULT, nullable
│  ├─ ALTER TABLE users ADD COLUMN email_verified BOOLEAN DEFAULT false;
│  ├─ Runs: Instantly (just adds column, no data change)
│  └─ Data: Existing rows get value false
│
├─ Migration 2: Make it NOT NULL (after data is populated)
│  ├─ ALTER TABLE users ALTER COLUMN email_verified SET NOT NULL;
│  ├─ Runs: Instantly (all rows already have value)
│  └─ Data: No rows with NULL, safe to make NOT NULL
│
└─ Result: Zero downtime, no data loss

Step 3: Schema.prisma reflects final state
├─ model User {
│  │ email_verified Boolean @default(false)
│  └─ }
└─ Migration will auto-generate if you made both changes above
```

---

### 3.4.2 Zero-Downtime Migration Strategy

**Critical: Database changes must not block users:**

```
Categories of Migrations:

1. SAFE MIGRATIONS (Can run during business hours)
├─ Add nullable column
│  ├─ ALTER TABLE users ADD COLUMN phone VARCHAR(20);
│  ├─ Doesn't require data migration
│  ├─ Existing rows can have NULL
│  ├─ Application can be updated immediately
│  └─ Time: < 1 second (even for large tables)
│
├─ Add column with DEFAULT
│  ├─ ALTER TABLE users ADD COLUMN email_verified BOOLEAN DEFAULT false;
│  ├─ Existing rows get DEFAULT value instantly
│  ├─ Query unaffected
│  └─ Time: < 1 second
│
├─ Create new table
│  ├─ CREATE TABLE temp_data (id UUID PRIMARY KEY, ...);
│  ├─ No impact on existing tables
│  ├─ Can be created anytime
│  └─ Time: Seconds (even if table has millions of rows)
│
├─ Add index
│  ├─ CREATE INDEX idx_users_email ON users(email);
│  ├─ Runs in background (CONCURRENTLY)
│  ├─ Doesn't lock table
│  ├─ Queries still work during indexing
│  └─ Time: Minutes for 100M rows (but non-blocking)
│
├─ Add NOT NULL with DEFAULT to existing column
│  ├─ Must be two-step:
│  │  ├─ Step 1: Add DEFAULT (already has data)
│  │  └─ Step 2: Add NOT NULL (now safe)
│  ├─ Combined: ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP DEFAULT NULL;
│  └─ Time: < 1 second
│
└─ Rename column (optional, PostgreSQL supports)
   ├─ ALTER TABLE users RENAME COLUMN phone TO phone_number;
   ├─ Doesn't require data migration
   ├─ Update application code
   └─ Time: Instant (just metadata change)

2. RISKY MIGRATIONS (Requires careful planning)
├─ Add NOT NULL column without DEFAULT to existing data
│  ├─ ❌ WRONG: ALTER TABLE users ADD COLUMN password NOT NULL;
│  ├─ ERROR: Existing rows don't have value
│  ├─ Solution: Two-step migration above
│  └─ Time: 0ms + migration setup time
│
├─ Remove or rename column used by application
│  ├─ Risk: Old app still running, tries to use deleted column → ERROR
│  ├─ Solution: Blue-green deployment (see deployment section)
│  ├─ Step 1: Deploy new code (doesn't use column)
│  ├─ Step 2: Migrate database (remove column)
│  ├─ Step 3: Confirm no errors
│  └─ Rollback: Restore column, deploy old code
│
├─ Add UNIQUE constraint to column with duplicates
│  ├─ ❌ WRONG: ALTER TABLE users ADD UNIQUE(email); (fails if duplicates exist)
│  ├─ Solution: Clean data first, then add constraint
│  ├─ Step 1: Identify duplicates: SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;
│  ├─ Step 2: Merge or delete: DELETE FROM users WHERE id IN (select id from ...) -- keep one
│  ├─ Step 3: Add constraint: ALTER TABLE users ADD UNIQUE(email);
│  └─ Time: Depends on data cleanup
│
├─ Add foreign key constraint to column with invalid references
│  ├─ ❌ WRONG: ALTER TABLE orders ADD CONSTRAINT fk_customer FOREIGN KEY (customer_id) REFERENCES users(id); (fails if some customer_ids don't exist)
│  ├─ Solution: Clean data first
│  ├─ Step 1: Find orphans: SELECT DISTINCT customer_id FROM orders WHERE customer_id NOT IN (SELECT id FROM users);
│  ├─ Step 2: Fix: DELETE FROM orders WHERE customer_id NOT IN (SELECT id FROM users);
│  ├─ Step 3: Add FK: ALTER TABLE orders ADD FOREIGN KEY (customer_id) REFERENCES users(id);
│  └─ Time: Depends on data cleanup
│
└─ Change column type (INTEGER → VARCHAR)
   ├─ ❌ WRONG: ALTER TABLE products MODIFY price VARCHAR(50);
   ├─ Risk: String comparison ≠ number comparison
   ├─ Solution: Create new column, copy data, switch (multi-step)
   ├─ Step 1: Add new column: ALTER TABLE products ADD COLUMN price_new BIGINT;
   ├─ Step 2: Copy data: UPDATE products SET price_new = CAST(price AS BIGINT);
   ├─ Step 3: Update app code: Use price_new instead of price
   ├─ Step 4: Deploy code
   ├─ Step 5: Swap columns: ALTER TABLE products DROP COLUMN price; ALTER TABLE products RENAME COLUMN price_new TO price;
   └─ Time: Migration + code update delay (1-2 hours total)

3. IMPOSSIBLE MIGRATIONS (Requires downtime)
├─ Drop required column while application still uses it
│  ├─ Even with careful planning: Code must be updated first
│  ├─ Solution: Blue-green deployment (zero downtime with proper orchestration)
│  └─ Time: < 1 minute with blue-green
│
├─ Change primary key (rare, only for major refactoring)
│  ├─ Affects ALL foreign keys
│  ├─ Solution: Create new table with new PK, copy data, switch (hours of work)
│  └─ Time: 2-4 hours + testing
│
├─ Rename heavily-used table
│  ├─ All foreign keys affected
│  ├─ All application code affected
│  ├─ Solution: Create new table, migrate data over time (weeks)
│  └─ Time: Days to weeks
│
└─ Split one table into multiple tables
   ├─ Requires migration logic (normalize data)
   ├─ Must update all queries
   ├─ Complex transformation
   └─ Time: Days to weeks

STRATEGY FOR SAFE MIGRATIONS:

1. Additive only (in primary migration)
   ├─ Add columns, tables, indexes only
   ├─ Never remove or change in same release
   ├─ This enables rollback (if code breaks, revert app + ignore migrations)
   └─ Example: Prepare column in migration, use in next deployment

2. Two deployments for breaking changes
   ├─ Deployment 1: Deploy code that handles old AND new columns
   ├─ Deployment 2 (1+ hour later): Remove old columns
   ├─ Rollback: If code breaks, revert before reaching step 2
   └─ This prevents "migration deployed but app fails" scenario

3. Test migrations on shadow database
   ├─ Command: npx prisma migrate dev (uses shadow DB by default)
   ├─ Tests: All migrations on clean database
   ├─ Validates: No errors in migration SQL
   ├─ Time: Minutes
   └─ Run: Before each migration deployment

4. Measure migration impact
   ├─ Time: How long does migration take?
   ├─ Locks: Does it lock table? (bad for live systems)
   ├─ Data: Does it modify existing data?
   ├─ Rollback: Can it be reversed easily?
   └─ Decision: Schedule accordingly (off-hours if > 5 minutes)

Our Migration Rules:

✅ Always allowed:
├─ Add nullable column
├─ Add column with DEFAULT
├─ Add index (with CONCURRENTLY)
├─ Create new table
├─ Add foreign key to new table
└─ Add unique constraint to column without duplicates

⚠️ Allowed with caution:
├─ Add NOT NULL column (must have DEFAULT)
├─ Drop index (verify not used first)
├─ Rename column (update code simultaneously)
├─ Add foreign key to existing column (verify data first)
└─ Add unique constraint (check for duplicates first)

❌ Never allowed in single deployment:
├─ Add required column to existing table without DEFAULT
├─ Drop column that application uses
├─ Drop table
├─ Change column type
├─ Remove foreign key constraint
└─ Change primary key
```

---

## 3.5 DATABASE OPERATIONS PLANNING

### 3.5.1 Backup & Disaster Recovery Strategy

**Never lose data - implement backups before going live:**

```
BACKUP REQUIREMENTS:

Tier 1: Automated Backups (RDS built-in)
├─ Frequency: Automated daily (automated every 6 hours by default)
├─ Retention: 35 days (keep for 1 month minimum)
├─ Location: AWS-managed (multiple AZs automatically)
├─ Trigger: Automatic (no configuration needed)
├─ Restoration: Point-in-time recover (PITR) within 35 days
├─ RPO (Recovery Point Objective): 6 hours (data loss if DB fails mid-day)
├─ RTO (Recovery Time Objective): 10 minutes (take 10 min to restore)
└─ Cost: Included in RDS pricing

Configuration:
├─ AWS RDS Console:
│  ├─ Automated backups: Enabled
│  ├─ Backup retention period: 35 days
│  ├─ Backup window: 02:00 UTC (off-peak hours)
│  ├─ Multi-AZ: Enabled (automatic failover to standby)
│  └─ Copy automated backups to another region: Enabled (DR)
└─ Command: No command needed (automatic)

Tier 2: Manual Snapshots (Periodic, long-term)
├─ Frequency: Daily snapshot (takes 5 minutes)
├─ Retention: 90 days (keep for 3 months for compliance)
├─ Location: AWS-managed + replicate to another region
├─ Trigger: Daily job (scheduled task)
├─ Restoration: Full restore (faster than PITR for old dates)
├─ Use case: Compliance, long-term archive
└─ Cost: S3 storage (~$0.023 per GB-month)

Tier 3: Logical Backups (Application-consistent)
├─ Frequency: Daily at 03:00 UTC
├─ Method: pg_dump (PostgreSQL dump)
├─ Format: Custom format (compressed, faster restore)
├─ Storage: S3 (long-term, durable)
├─ Retention: 1 year (regulatory compliance)
├─ Restoration: From logical backup (slower but data-safe)
└─ Cost: S3 storage

Tier 4: Replication (Hot standby for failover)
├─ Type: RDS Read Replica (synchronous or asynchronous)
├─ Frequency: Real-time (continuous replication)
├─ Location: Different AZ (automatic failover if primary fails)
├─ Lag: < 100ms (asynchronous, acceptable for our scale)
├─ Failover time: < 1 minute (automatic)
├─ Use: High availability (not primary backup)
└─ Cost: Additional instance (100% of primary instance cost)

BACKUP STRATEGY IMPLEMENTATION:

Step 1: Enable RDS Automated Backups (5 minutes)
├─ AWS RDS console
├─ DB instances → select database
├─ Backup & restore section
├─ Enable automated backups: Yes
├─ Retention: 35 days
├─ Save

Step 2: Enable Multi-AZ (for automatic failover)
├─ AWS RDS console
├─ Deployment & security section
├─ Multi-AZ deployment: Yes
├─ Creates standby replica in different AZ
├─ Automatic failover on failure
├─ Cost: +100% (two instances)
└─ Worth it: Reduces RTO from 10 min → < 1 min

Step 3: Create Daily Logical Backup (automated)
├─ AWS Lambda function:
│  ├─ Schedule: cron(0 3 * * ? *) - 03:00 UTC daily
│  ├─ Code:
│  │  ├─ pg_dump -h {RDS endpoint} -U postgres {database} | gzip > backup.sql.gz
│  │  ├─ aws s3 cp backup.sql.gz s3://my-backups/$(date +%Y%m%d_%H%M%S).sql.gz
│  │  └─ Cleanup: Delete backups > 1 year old
│  └─ IAM role: Read RDS password from Secrets Manager, write to S3
│
├─ Backup size: ~500MB compressed for 50GB DB (1% of original)
├─ Time: ~10 minutes per backup
└─ Storage cost: $0.023 per GB-month × 365 days = ~$4.20/year per backup

Step 4: Test Restore Procedure (monthly)
├─ Restore from backup to test instance
├─ Verify: Data integrity
├─ Time: Full restore takes ~30 minutes
├─ Process:
│  ├─ 1. Create new RDS instance
│  ├─ 2. Create DB from S3 snapshot
│  ├─ 3. Run sanity checks: SELECT COUNT(*) from each table
│  ├─ 4. Verify recent data: SELECT * from orders where created_at > NOW() - INTERVAL 1 day
│  ├─ 5. Delete test instance (cleanup)
│  └─ 6. Document restore time + any issues
└─ Frequency: Monthly (or before major deployment)

DISASTER RECOVERY PLAN:

Scenario 1: Database instance fails (hardware error)
├─ Time to detect: < 1 minute (CloudWatch alert)
├─ Action: Enable Multi-AZ failover (automatic within 1 minute)
├─ Result: Standby replica becomes primary
├─ RTO: < 2 minutes (restart app servers to connect to new primary)
├─ RPO: 0 (synchronous replication with Multi-AZ)
└─ Cost: None (already paid for Multi-AZ)

Scenario 2: Accidental data deletion (dropped table)
├─ Detection: Monitoring shows missing rows
├─ Time to detect: Minutes (need monitoring alert for row count anomalies)
├─ Action: Restore from PITR to point before deletion
│  ├─ Command: CreateDBInstanceFromDBSnapshot in AWS
│  ├─ Time: ~30 minutes for 50GB database
│  ├─ Process: Create snapshot → restore to new instance → verify → failover
│  └─ RTO: 30-45 minutes
├─ RPO: Up to 6 hours (since last automated backup)
├─ Prevention: Soft deletes (deleted_at timestamp), never DROP
└─ Cost: ~$0.50 for restore

Scenario 3: Ransomware / Malicious deletion
├─ Detection: Sudden DROP of multiple tables
├─ Action: Restore from backup predating attack
│  ├─ Use logical backup if known when attack occurred
│  ├─ Use snapshot from 1+ day ago
│  ├─ Verify no malicious code in app servers
│  └─ RTO: 2-4 hours (restore + verification)
├─ RPO: Up to 24 hours (restore from previous day's snapshot)
├─ Prevention:
│  ├─ Database access controls (least privilege)
│  ├─ Immutable snapshots (can't delete old backups)
│  ├─ Separate AWS account for backups (can't access from app account)
│  └─ Encryption (ransomware can't read encrypted backups)
└─ Cost: None (already has backups)

Scenario 4: Regional failure (AWS region down)
├─ Detection: All services in region unavailable
├─ Action: Failover to secondary region
│  ├─ Prerequisites: Cross-region read replica configured
│  ├─ Promote: Read replica to standalone primary
│  ├─ Time: ~1 minute (just promote)
│  ├─ DNS: Update Route 53 to point to secondary region
│  ├─ Time: ~1 minute (DNS update)
│  └─ Total RTO: 2-3 minutes
├─ RPO: < 1 second (read replica replicates synchronously)
├─ Cost: Double (primary + secondary region instances)
└─ Worth it? Depends on SLA requirements (not for MVP)

Backup Monitoring & Alerts:

```yaml
Alerts:
  - Automated backup failed
    └─ Action: Check CloudWatch logs, retry manually
  
  - Backup older than 35 days
    └─ Action: Extend retention period
  
  - Database instance fails
    └─ Action: Multi-AZ automatic failover (monitor completion)
  
  - Unusual row count drop
    └─ Action: Check audit logs, investigate deletion
  
  - High CPU on database
    └─ Action: Investigate slow queries, scale if needed

Dashboard (CloudWatch):
  - Backup count (should be daily)
  - Latest backup timestamp (should be < 24 hours old)
  - Instance status (should be green)
  - Replication lag (should be < 100ms)
  - Failed queries (should be 0)
```

---

## 3.6 DELIVERABLES & GATES

### Final Deliverables for Phase 3:

```
Deliverable 1: Database Schema Document
├─ Content:
│  ├─ Entity-relationship diagram (ERD)
│  ├─ Table descriptions (purpose, size estimate)
│  ├─ Column specifications (type, constraints, indexes)
│  ├─ Relationship mapping (1:1, 1:M, etc)
│  └─ Query patterns (most common queries + performance assumptions)
├─ Format: Markdown + visual diagrams
└─ Review: Architecture team, Backend lead

Deliverable 2: Prisma Schema File
├─ File: prisma/schema.prisma
├─ Content: Complete, production-ready schema
├─ Validation:
│  ├─ All tables defined
│  ├─ All relationships properly configured
│  ├─ All indexes created
│  ├─ All constraints enforced
│  └─ No TypeScript errors
├─ Testing: Generate Prisma client, verify no errors
└─ Location: Git repository, accessible to team

Deliverable 3: Migration Scripts
├─ Location: prisma/migrations/{timestamp}_{name}/migration.sql
├─ Content: All DDL statements
├─ Checklist:
│  ├─ No breaking changes to existing columns
│  ├─ All new columns have defaults or are nullable
│  ├─ Foreign keys are RESTRICT (don't lose data)
│  ├─ Indexes created
│  ├─ Comments added (why this migration)
│  └─ Rollback procedure documented
├─ Testing: Run on shadow database, verify success
└─ Version control: All migrations tracked in Git

Deliverable 4: Backup & Recovery Procedures
├─ Document: docs/disaster-recovery.md
├─ Content:
│  ├─ Backup frequency and location
│  ├─ Recovery procedures (step-by-step)
│  ├─ RTO/RPO targets
│  ├─ Failover procedures
│  ├─ Test results (successful restore from each backup type)
│  └─ Contact escalation (who to call if disaster)
├─ Testing:
│  ├─ Monthly restore test from production backup
│  ├─ Verify data integrity
│  ├─ Document restore time
│  └─ Sign-off by ops team
└─ Frequency: Review/test monthly

Deliverable 5: Data Migration Plan (if applicable)
├─ Applies to: Migrating from legacy system
├─ Content:
│  ├─ Legacy data analysis (what data exists)
│  ├─ Mapping (legacy schema → new schema)
│  ├─ Transformation rules (how to convert data)
│  ├─ Validation plan (verify migration success)
│  ├─ Rollback procedure (revert if issues found)
│  └─ Cutover plan (when to switch to new data)
├─ Scripts: Migration code (SQL or Python)
└─ Testing: Dry run on test data set

Deliverable 6: Performance Baseline
├─ Measurement:
│  ├─ Query: SELECT COUNT(*) FROM {each table}
│  ├─ Index: EXPLAIN ANALYZE for top 10 queries
│  ├─ Storage: How much disk space used
│  └─ Growth: Projected size at Year 1 scale
├─ Results: Documented in performance-baseline.md
└─ Use: Comparison point for future optimization

Deliverable 7: Database Setup Documentation
├─ File: docs/database-setup.md
├─ Content:
│  ├─ Local development: How to set up locally
│  ├─ Environment: PostgreSQL version, Docker setup
│  ├─ Seed data: How to populate test data
│  ├─ Reset: How to reset database during development
│  └─ Troubleshooting: Common issues + solutions
├─ Examples:
│  ├─ docker-compose.yml (database + adminer UI)
│  ├─ seed.ts script (populate test data)
│  ├─ .env.example (configuration)
│  └─ migrations README (how they work)
└─ Audience: All developers (should be able to setup in 10 minutes)

### Phase 3 Approval Gates:

Gate 1: Schema Design Review ✓
├─ Reviewers: Backend lead, database expert (if available)
├─ Criteria:
│  ├─ All entities identified (no missing tables)
│  ├─ Relationships are correctly modeled
│  ├─ Normalization appropriate (no unnecessary redundancy)
│  ├─ PKs and FKs properly designed
│  ├─ Constraints enforce business rules
│  └─ Soft deletes planned where needed
├─ Approval: Sign-off that schema matches requirements
└─ Time: 1-2 hours review time

Gate 2: Performance Validation ✓
├─ Verification:
│  ├─ All frequently-queried columns are indexed
│  ├─ Query plans use indexes (EXPLAIN ANALYZE confirms)
│  ├─ No obvious N+1 query problems
│  ├─ Denormalization justified (measured & documented)
│  └─ Capacity projections align with growth targets
├─ Success: Top 10 queries all < 100ms
└─ Sign-off: Performance acceptable for MVP

Gate 3: Migration Testing ✓
├─ Test: Run all migrations on fresh PostgreSQL instance
├─ Verify:
│  ├─ All migrations apply without error
│  ├─ Schema matches Prisma schema after migration
│  ├─ Foreign key constraints work
│  ├─ Default values applied correctly
│  └─ Indexes created and function properly
├─ Test: Reverse migrations (if supported)
└─ Sign-off: Migrations are production-ready

Gate 4: Backup Validation ✓
├─ Test: Automated backup job runs successfully
├─ Verify:
│  ├─ Backup file created
│  ├─ Backup is valid (can be restored)
│  ├─ Restore procedure documented
│  ├─ RTO/RPO targets documented
│  └─ Team trained on recovery procedures
├─ Test: Restore from backup to separate instance
│  ├─ Verify data integrity after restore
│  ├─ Confirm row counts match source
│  └─ Document restore time
└─ Sign-off: Backups and recovery procedures ready

Gate 5: Developer Environment Setup ✓
├─ Test: Fresh developer can setup database in < 10 minutes
├─ Verification:
│  ├─ Docker compose file works
│  ├─ Seed data script populates correctly
│  ├─ Prisma client generates without errors
│  ├─ Can query data from local instance
│  └─ Reset script works
├─ Documentation: Step-by-step guide available
└─ Sign-off: Dev team can work independently

Gate 6: Security Review ✓
├─ Checklist:
│  ├─ No hardcoded passwords in schema
│  ├─ Secrets management configured (Prisma uses env vars)
│  ├─ Connection encrypted (TLS to RDS)
│  ├─ Access control configured (database user limited permissions)
│  ├─ Backup encryption enabled
│  ├─ Sensitive data encrypted at rest (if required)
│  └─ Audit logging planned for sensitive tables
├─ Review: Security team or architect
└─ Sign-off: Security requirements met

### Phase 3 Sign-Off:

```
PHASE 3 SIGN-OFF CHECKLIST:

[ ] Schema Design Review - Approved
[ ] Performance Validation - All queries < 100ms
[ ] Migration Testing - All migrations pass
[ ] Backup Validation - Restore test successful
[ ] Developer Setup - Fresh developer can setup in 10 min
[ ] Security Review - All security checks pass
[ ] Documentation Complete - Available in docs/ folder
[ ] Prisma Schema Valid - No TypeScript errors

Approval signatures:
├─ Backend Lead: _____________________ Date: ______
├─ Database Expert: __________________ Date: ______
├─ Security Lead: _____________________ Date: ______
└─ Project Manager: __________________ Date: ______

Next Phase: Ready for API Design (Phase 4)
```

---

## SUMMARY

Phase 3 transforms the architectural decisions into a concrete, tested database design. The schema is the foundation - get it right here, and implementation flows smoothly. Get it wrong, and migrations become nightmares.

Key outcomes:
✅ Production-ready schema (PostgreSQL)
✅ Zero-downtime migrations planned
✅ Backups & disaster recovery configured
✅ Performance targets achievable
✅ Developer workflow enabled
✅ Security & compliance addressed

This doc is referenced by:
- Backend developers (implementing queries)
- DevOps (deploying migrations)
- Security team (data protection)
- QA (test data & edge cases)
- Future maintainers (understanding design decisions)
