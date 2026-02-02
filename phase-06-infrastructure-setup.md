<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# PHASE 6: INFRASTRUCTURE SETUP

**Why Infrastructure Matters:**
Infrastructure is where 80% of production bugs hide. Bad infra design = outages, slow deploys, leaked secrets, security breaches, impossible debugging. A 20-year veteran knows: infrastructure decisions made here cost 10x more to fix later. Get it right now—reproducible, automated, secure, observable.

***

## 6.1 DEVELOPMENT ENVIRONMENT

### 6.1.1 Docker Setup for Local Development

**Goal: Every developer runs identical environment. "Works on my machine" never happens again.**

**Why Docker:**

- Backend dev runs Node + PostgreSQL + Redis + OpenSearch in 3 minutes
- Frontend dev gets same backend for testing
- No "but it works on my Mac" vs "broken on Linux"
- Production runs same Docker image locally (catch problems early)
- Onboard new engineer in 30 minutes, not 3 days

**Docker Compose structure:**

```yaml
# docker-compose.yml (development)
version: '3.8'

services:
  # PostgreSQL database
  postgres:
    image: postgres:15-alpine
    container_name: orders-db-dev
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: dev_user
      POSTGRES_PASSWORD: dev_password
      POSTGRES_DB: orders_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./sql/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dev_user"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis cache
  redis:
    image: redis:7-alpine
    container_name: orders-cache-dev
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Backend API
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    container_name: orders-api-dev
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://dev_user:dev_password@postgres:5432/orders_db
      REDIS_URL: redis://redis:6379
      JWT_SECRET: dev_secret_key_not_for_production
      STRIPE_SECRET_KEY: sk_test_...
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./backend/src:/app/src
      - /app/node_modules
    command: npm run dev

  # Frontend development server
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    container_name: orders-ui-dev
    ports:
      - "5173:5173"
    environment:
      VITE_API_URL: http://localhost:3000/api
    depends_on:
      - api
    volumes:
      - ./frontend/src:/app/src
      - /app/node_modules
    command: npm run dev

volumes:
  postgres_data:
    driver: local

networks:
  default:
    name: orders-dev-network
```

**Dockerfile for Node backend (development):**

```dockerfile
# Dockerfile.dev (development with hot reload)
FROM node:20-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Copy source
COPY src ./src
COPY tsconfig.json ./

# Volume mount for hot reload
VOLUME ["/app/src"]

EXPOSE 3000

CMD ["npm", "run", "dev"]
```

**Dockerfile for Node backend (production):**

```dockerfile
# Dockerfile.prod (multi-stage, optimized)
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY src ./src
COPY tsconfig.json ./

# Build TypeScript
RUN npm run build

# Production image (minimal)
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY --from=builder /app/dist ./dist

EXPOSE 3000

CMD ["node", "dist/index.js"]
```

**Start development:**

```bash
# Terminal 1: Start all services
docker-compose up

# Terminal 2: Run migrations (one-time)
docker-compose exec api npm run migrate

# Terminal 3: Run seeds (populate test data)
docker-compose exec api npm run seed

# Access
# Frontend: http://localhost:5173
# Backend: http://localhost:3000
# Database: localhost:5432 (psql -h localhost -U dev_user -d orders_db)
# Redis: localhost:6379 (redis-cli)
```

**Useful commands:**

```bash
# View logs
docker-compose logs api          # Backend logs
docker-compose logs -f postgres  # Follow postgres logs

# Execute commands in container
docker-compose exec api npm test # Run tests
docker-compose exec postgres psql -U dev_user -d orders_db  # Connect to DB

# Reset everything
docker-compose down -v          # Remove volumes too (resets DB)
docker-compose up              # Rebuild and start fresh

# Database inspection
docker-compose exec postgres psql -U dev_user -d orders_db
\dt                            # List tables
SELECT * FROM users;           # Query
\q                             # Exit
```


***

### 6.1.2 Database Initialization \& Seeding

**Reproducible data: Every developer starts with same initial state.**

**Init script (run once on container start):**

```sql
-- sql/init.sql
-- Runs automatically when postgres container starts

-- Create extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

-- Create schemas
CREATE SCHEMA IF NOT EXISTS public;

-- Create tables (application does this via migrations)
-- This init script just creates extensions + databases

-- Create dedicated user for app (not root)
CREATE USER app_user WITH PASSWORD 'app_password';
GRANT CONNECT ON DATABASE orders_db TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT CREATE ON SCHEMA public TO app_user;
```

**Database migrations (version control for schema):**

```typescript
// backend/db/migrations/001_create_users.ts
import { Knex } from "knex";

export async function up(knex: Knex): Promise<void> {
  return knex.schema.createTable("users", (table) => {
    table.uuid("id").primary().defaultTo(knex.raw("uuid_generate_v4()"));
    table.string("email").unique().notNullable();
    table.string("password_hash").notNullable();
    table.enum("role", ["user", "admin"]).defaultTo("user");
    table.timestamps(true, true);
    table.index("email");
  });
}

export async function down(knex: Knex): Promise<void> {
  return knex.schema.dropTable("users");
}
```

**Run migrations:**

```bash
# Terminal
npm run migrate:latest  # Run all pending migrations
npm run migrate:up     # Run next migration
npm run migrate:down   # Rollback last migration
npm run migrate:reset  # Drop all tables, run from start
```

**Seed script (populate test data):**

```typescript
// backend/db/seeds/01_seed_users.ts
import { Knex } from "knex";
import bcrypt from "bcrypt";

export async function seed(knex: Knex): Promise<void> {
  // Clear existing data
  await knex("orders").del();
  await knex("users").del();

  // Insert test users
  await knex("users").insert([
    {
      id: "11111111-1111-1111-1111-111111111111",
      email: "admin@example.com",
      password_hash: await bcrypt.hash("admin123", 10),
      role: "admin",
    },
    {
      id: "22222222-2222-2222-2222-222222222222",
      email: "user@example.com",
      password_hash: await bcrypt.hash("user123", 10),
      role: "user",
    },
  ]);

  // Insert test orders
  await knex("orders").insert([
    {
      id: "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
      customer_id: "22222222-2222-2222-2222-222222222222",
      total: 99.99,
      status: "pending",
    },
    {
      id: "bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb",
      customer_id: "22222222-2222-2222-2222-222222222222",
      total: 149.99,
      status: "shipped",
    },
  ]);
}
```

**Run seeds:**

```bash
npm run seed  # Populate with test data
```


***

### 6.1.3 Mock API Server Setup

**Frontend can develop without waiting for backend. Use mock API.**

**Mock server with MSW (Mock Service Worker):**

```typescript
// frontend/src/mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  // GET /api/orders
  http.get('*/api/v1/orders', () => {
    return HttpResponse.json({
      status: 'success',
      data: [
        {
          id: 'ORD-001',
          customer_id: 'CUST-001',
          total: 99.99,
          status: 'pending',
          created_at: '2024-01-15T10:30:00Z'
        },
        {
          id: 'ORD-002',
          customer_id: 'CUST-001',
          total: 149.99,
          status: 'shipped',
          created_at: '2024-01-14T14:22:00Z'
        }
      ],
      metadata: {
        total: 2,
        limit: 20,
        offset: 0
      }
    });
  }),

  // POST /api/orders
  http.post('*/api/v1/orders', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json(
      {
        status: 'success',
        data: {
          id: `ORD-${Math.random().toString().slice(2, 6)}`,
          customer_id: body.customer_id,
          total: body.total,
          status: 'pending',
          created_at: new Date().toISOString()
        }
      },
      { status: 201 }
    );
  }),

  // POST /api/auth/login
  http.post('*/api/v1/auth/login', () => {
    return HttpResponse.json({
      status: 'success',
      data: {
        access_token: 'mock_jwt_token_here',
        user: {
          id: 'USER-001',
          email: 'user@example.com',
          role: 'user'
        }
      }
    });
  })
];
```

**Setup MSW:**

```typescript
// frontend/src/mocks/setup.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

// Only setup mock server in development, not production
export const server = setupServer(...handlers);

if (process.env.NODE_ENV === 'development') {
  server.listen({ onUnhandledRequest: 'warn' });
}
```

**Use in tests:**

```typescript
// frontend/src/__tests__/OrdersList.test.tsx
import { render, screen } from '@testing-library/react';
import { server } from '../mocks/setup';
import { OrdersList } from '../components/OrdersList';

describe('OrdersList', () => {
  it('renders orders from mock API', async () => {
    render(<OrdersList />);
    
    // Wait for requests to complete
    await screen.findByText('ORD-001');
    
    expect(screen.getByText('$99.99')).toBeInTheDocument();
    expect(screen.getByText('$149.99')).toBeInTheDocument();
  });

  it('shows error when API fails', async () => {
    // Override specific handler for this test
    server.use(
      http.get('*/api/v1/orders', () => {
        return HttpResponse.error();
      })
    );

    render(<OrdersList />);
    
    await screen.findByText(/error/i);
  });
});
```

**Toggle between real and mock:**

```typescript
// frontend/src/api/client.ts
const API_BASE = process.env.REACT_APP_API_URL || 'http://localhost:3000';

const USE_MOCK_API = process.env.REACT_APP_USE_MOCK_API === 'true';

if (USE_MOCK_API) {
  // Import mock handlers
  await import('../mocks/setup');
}

export const apiClient = axios.create({
  baseURL: API_BASE
});
```

**Run with mock:**

```bash
# Terminal
REACT_APP_USE_MOCK_API=true npm run dev
```


***

### 6.1.4 Hot Reload Configuration

**Code change → App reloads in browser instantly (state preserved).**

**Vite hot reload (frontend):**

```typescript
// frontend/src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root')!);

root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// Vite HMR (Hot Module Replacement)
if (import.meta.hot) {
  import.meta.hot.accept();
}
```

**Result:**

- Edit component → Browser updates automatically
- State preserved (form input values stay)
- No full page reload

**Node hot reload (backend):**

```json
// backend/package.json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "start": "node dist/index.js"
  },
  "devDependencies": {
    "tsx": "^4.7.0"
  }
}
```

**tsx watch detects changes, restarts server in 200ms.**

***

### 6.1.5 Environment Variable Management

**Secrets stay secret. Configuration stays configurable.**

**Environment variable strategy:**

```
Development:   .env.local (git-ignored, local secrets)
Staging:       AWS Secrets Manager (environment-specific)
Production:    AWS Secrets Manager (environment-specific)
Testing:       .env.test (committed, fake data)
```

**Development setup:**

```bash
# frontend/.env.local (git-ignored)
VITE_API_URL=http://localhost:3000/api
VITE_ENV=development
VITE_USE_MOCK_API=false

# backend/.env.local (git-ignored)
NODE_ENV=development
DATABASE_URL=postgresql://dev_user:dev_password@localhost:5432/orders_db
REDIS_URL=redis://localhost:6379
JWT_SECRET=dev_secret_key_only_for_local
STRIPE_SECRET_KEY=sk_test_your_test_key
SENDGRID_API_KEY=SG.test_key
```

**Load environment variables safely:**

```typescript
// backend/src/config.ts
import dotenv from 'dotenv';
import path from 'path';

// Load .env based on NODE_ENV
const envFile = process.env.NODE_ENV === 'test' ? '.env.test' : '.env.local';
dotenv.config({ path: path.resolve(envFile) });

interface Config {
  nodeEnv: string;
  databaseUrl: string;
  redisUrl: string;
  jwtSecret: string;
  stripeSecretKey: string;
}

// Validate required environment variables
function validateEnv(): Config {
  const required = [
    'DATABASE_URL',
    'REDIS_URL',
    'JWT_SECRET',
    'STRIPE_SECRET_KEY'
  ];

  for (const key of required) {
    if (!process.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }

  return {
    nodeEnv: process.env.NODE_ENV || 'development',
    databaseUrl: process.env.DATABASE_URL!,
    redisUrl: process.env.REDIS_URL!,
    jwtSecret: process.env.JWT_SECRET!,
    stripeSecretKey: process.env.STRIPE_SECRET_KEY!
  };
}

export const config = validateEnv();
```

**Frontend environment variables:**

```typescript
// frontend/src/config.ts
const config = {
  apiUrl: import.meta.env.VITE_API_URL,
  env: import.meta.env.VITE_ENV,
  useMockApi: import.meta.env.VITE_USE_MOCK_API === 'true'
};

if (!config.apiUrl) {
  throw new Error('Missing VITE_API_URL environment variable');
}

export default config;
```

**Never commit secrets:**

```bash
# .gitignore
.env.local           # Never commit
.env.*.local         # Never commit
.env.production      # Never commit
.DS_Store
node_modules
dist
```

**Safe defaults in git:**

```bash
# .env.example (commit this, no secrets)
VITE_API_URL=http://localhost:3000/api
VITE_ENV=development
VITE_USE_MOCK_API=false

NODE_ENV=development
DATABASE_URL=postgresql://user:pass@localhost:5432/db
REDIS_URL=redis://localhost:6379
JWT_SECRET=change_me_in_development
STRIPE_SECRET_KEY=sk_test_your_key
```


***

## 6.2 CI/CD PIPELINE

### 6.2.1 Pipeline Stages Definition

**Code commit → Tests → Build → Security scan → Deploy (fully automated).**

**Pipeline stages:**

```
Trigger (git push)
    ↓
┌─────────────────────────────────────┐
│ Stage 1: LINT & FORMAT (2 min)      │
├─────────────────────────────────────┤
│ ├─ ESLint (code quality)            │
│ ├─ Prettier (formatting)            │
│ ├─ TypeScript compile check         │
│ └─ Fail if issues found             │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Stage 2: TEST (5 min)               │
├─────────────────────────────────────┤
│ ├─ Unit tests (Jest)                │
│ ├─ Integration tests                │
│ ├─ Component tests (React)          │
│ └─ Fail if coverage < 80%           │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Stage 3: BUILD (3 min)              │
├─────────────────────────────────────┤
│ ├─ Backend: npm run build           │
│ ├─ Frontend: npm run build          │
│ ├─ Docker image creation            │
│ └─ Push to ECR (AWS)                │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Stage 4: SECURITY SCAN (4 min)      │
├─────────────────────────────────────┤
│ ├─ OWASP/Snyk dependency scan       │
│ ├─ Container vulnerability scan     │
│ ├─ SAST (Sonarqube)                 │
│ └─ Fail if critical vulns found     │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Stage 5: DEPLOY STAGING (5 min)     │
├─────────────────────────────────────┤
│ ├─ Deploy to staging environment    │
│ ├─ Run smoke tests                  │
│ ├─ Manual approval to continue      │
│ └─ Fail if smoke tests fail         │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Stage 6: DEPLOY PRODUCTION (5 min)  │
├─────────────────────────────────────┤
│ ├─ Blue-green deployment            │
│ ├─ Health checks                    │
│ ├─ Monitor for 5 min                │
│ └─ Automatic rollback if errors     │
└─────────────────────────────────────┘
    ↓
SUCCESS (20 minutes total)
```

**GitHub Actions workflow:**

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    name: Lint & Format
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci

      - run: npm run lint
      - run: npm run format:check
      - run: npm run type-check

  test:
    name: Tests
    runs-on: ubuntu-latest
    needs: lint

    services:
      postgres:
        image: postgres:15-alpine
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npm run migrate:test
      - run: npm run test:ci
      - run: npm run test:coverage

      - uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
          fail_ci_if_error: true

  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: test
    if: github.event_name == 'push'

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
          aws-region: us-east-1

      - name: Login to ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: orders-api
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -f backend/Dockerfile.prod -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

  security-scan:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: build

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - run: npm install -g snyk
      - run: snyk test --fail-on=high

  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: [build, security-scan]
    environment: staging

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
          aws-region: us-east-1

      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster orders-staging \
            --service orders-api-staging \
            --force-new-deployment

      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster orders-staging \
            --services orders-api-staging

      - name: Run smoke tests
        run: |
          npx playwright test --project=smoke

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment: production
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
          aws-region: us-east-1

      - name: Blue-green deployment
        run: |
          # Get current active task definition
          CURRENT_TASK=$(aws ecs list-tasks \
            --cluster orders-prod \
            --service-name orders-api \
            --query 'taskArns[0]' --output text)

          # Deploy new task definition
          aws ecs update-service \
            --cluster orders-prod \
            --service orders-api \
            --force-new-deployment

          # Wait for new tasks to be healthy
          aws ecs wait services-stable \
            --cluster orders-prod \
            --services orders-api

          # Monitor error rate for 5 minutes
          # (done in background monitoring job)

      - name: Notify Slack
        if: success()
        run: |
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -d '{"text": "✅ Production deployment successful"}'

      - name: Automatic rollback on failure
        if: failure()
        run: |
          aws ecs update-service \
            --cluster orders-prod \
            --service orders-api \
            --task-definition orders-api:$(($TASK_VERSION - 1))
```


***

### 6.2.2 Automated Testing Integration

**Every commit tested automatically. No manual testing for basic cases.**

**Test pyramid (in time spent):**

```
         E2E Tests (10%)
           /\
          /  \
         /    \
    Integration (30%)
       /        \
      /          \
     /            \
   Unit Tests (60%)
```

**Unit tests (Jest):**

```typescript
// backend/src/__tests__/orders.service.test.ts
import { OrderService } from '../services/orders.service';
import { PrismaClient } from '@prisma/client';
import { Redis } from 'ioredis';

jest.mock('@prisma/client');
jest.mock('ioredis');

describe('OrderService', () => {
  let orderService: OrderService;
  let mockPrisma: jest.Mocked<PrismaClient>;
  let mockRedis: jest.Mocked<Redis>;

  beforeEach(() => {
    mockPrisma = new PrismaClient() as jest.Mocked<PrismaClient>;
    mockRedis = new Redis() as jest.Mocked<Redis>;
    orderService = new OrderService(mockPrisma, mockRedis);
  });

  describe('createOrder', () => {
    it('should create order with valid data', async () => {
      mockPrisma.order.create.mockResolvedValue({
        id: '123',
        customer_id: 'cust-1',
        total: 99.99,
        status: 'pending',
        created_at: new Date(),
        updated_at: new Date()
      });

      const result = await orderService.createOrder({
        customer_id: 'cust-1',
        total: 99.99
      });

      expect(result.id).toBe('123');
      expect(mockPrisma.order.create).toHaveBeenCalled();
    });

    it('should throw on insufficient inventory', async () => {
      mockPrisma.inventory.findUnique.mockResolvedValue({
        quantity: 2,
        sku: 'LAPTOP'
      });

      await expect(
        orderService.createOrder({
          items: [{ sku: 'LAPTOP', quantity: 5 }]
        })
      ).rejects.toThrow('Insufficient inventory');
    });
  });
});
```

**Integration tests (with real DB):**

```typescript
// backend/src/__tests__/orders.integration.test.ts
import { test, beforeAll, afterAll } from 'vitest';
import { PrismaClient } from '@prisma/client';
import { OrderService } from '../services/orders.service';

let prisma: PrismaClient;
let orderService: OrderService;

beforeAll(async () => {
  prisma = new PrismaClient({
    datasources: { db: { url: process.env.DATABASE_TEST_URL } }
  });
  orderService = new OrderService(prisma);
  
  // Setup: Create test user
  await prisma.user.create({
    data: {
      id: 'test-user-1',
      email: 'test@example.com',
      password_hash: 'hash'
    }
  });
});

afterAll(async () => {
  await prisma.$disconnect();
});

test('should create order and update inventory', async () => {
  // Setup
  await prisma.inventory.create({
    data: { sku: 'LAPTOP', quantity: 10 }
  });

  // Execute
  const order = await orderService.createOrder({
    customer_id: 'test-user-1',
    items: [{ sku: 'LAPTOP', quantity: 1 }]
  });

  // Verify
  const inventory = await prisma.inventory.findUnique({
    where: { sku: 'LAPTOP' }
  });

  expect(order.id).toBeDefined();
  expect(inventory?.quantity).toBe(9);
});
```

**E2E tests (Playwright):**

```typescript
// frontend/e2e/checkout.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Checkout Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Login before each test
    await page.goto('http://localhost:5173/auth/login');
    await page.fill('input[name="email"]', 'user@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button:has-text("Login")');
    await page.waitForNavigation();
  });

  test('should complete checkout successfully', async ({ page }) => {
    // Navigate to products
    await page.goto('http://localhost:5173/products');
    
    // Add item to cart
    await page.click('button:has-text("Add to Cart"):first');
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('1');

    // Go to checkout
    await page.click('button:has-text("Checkout")');
    await page.waitForURL('**/checkout');

    // Fill shipping address
    await page.fill('input[name="street"]', '123 Main St');
    await page.fill('input[name="city"]', 'San Francisco');
    await page.fill('input[name="state"]', 'CA');
    await page.fill('input[name="zip"]', '94102');

    // Fill payment
    await page.frameLocator('iframe[title="Stripe"]').locator('[placeholder="Card number"]')
      .fill('4242424242424242');
    await page.frameLocator('iframe[title="Stripe"]').locator('[placeholder="MM / YY"]')
      .fill('12 / 25');

    // Place order
    await page.click('button:has-text("Place Order")');

    // Verify confirmation
    await expect(page).toHaveURL('**/confirmation');
    await expect(page.locator('text=Order created successfully')).toBeVisible();
  });
});
```

**Run tests in CI:**

```bash
# Unit + integration tests
npm run test:ci

# E2E tests
npm run test:e2e

# Coverage report
npm run test:coverage
```


***

### 6.2.3 Code Quality Gates

**Bad code doesn't make it to main branch.**

**ESLint configuration:**

```javascript
// backend/.eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser',
  extends: ['eslint:recommended', 'plugin:@typescript-eslint/recommended'],
  rules: {
    'no-console': ['warn', { allow: ['error', 'warn'] }],
    'no-unused-vars': 'off',
    '@typescript-eslint/no-unused-vars': ['error'],
    '@typescript-eslint/explicit-function-return-types': 'warn',
    'no-var': 'error',
    'prefer-const': 'error'
  },
  overrides: [
    {
      files: ['**/*.test.ts'],
      env: { jest: true }
    }
  ]
};
```

**Prettier formatting:**

```javascript
// .prettierrc.js
module.exports = {
  semi: true,
  trailingComma: 'es5',
  singleQuote: true,
  printWidth: 100,
  tabWidth: 2,
  useTabs: false,
  arrowParens: 'always'
};
```

**Pre-commit hooks (prevent bad commits):**

```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.ts": ["eslint --fix", "prettier --write"],
    "*.tsx": ["eslint --fix", "prettier --write"]
  }
}
```

**Code quality scan (SonarQube):**

```yaml
# .github/workflows/sonarqube.yml
name: SonarQube Scan

on: [push, pull_request]

jobs:
  sonarqube:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - run: npm ci
      - run: npm run test:coverage

      - uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

**Quality gate prevents merge if:**

```
├─ Code coverage < 80%
├─ Duplication > 5%
├─ Critical/high bugs found
├─ Security hotspots unreviewed
└─ New vulnerabilities introduced
```


***

### 6.2.4 Security Scanning Setup

**Find vulnerabilities before attackers do.**

**Dependency scanning (Snyk):**

```bash
# Install Snyk CLI
npm install -g snyk

# Test dependencies
snyk test

# Monitor for new vulnerabilities
snyk monitor

# Fix known vulnerabilities
snyk fix
```

**SAST (Static Application Security Testing):**

```yaml
# .github/workflows/security.yml
name: Security Scan

on: [push, pull_request]

jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

      - name: Upload results
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: snyk.sarif

  container-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -f Dockerfile.prod -t myapp:${{ github.sha }} .

      - name: Scan with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: trivy-results.sarif
```

**Secret detection (prevent leaks):**

```bash
# Detect secrets before commit
npm install -D detect-secrets

# Scan entire codebase
detect-secrets scan

# Audit baseline
detect-secrets audit .secrets.baseline
```


***

### 6.2.5 Deployment Automation

**Push button (or automatic on merge) deployment.**

**GitOps with Flux (Kubernetes):**

```yaml
# infrastructure/flux/orders-api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders-api
  namespace: production
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: orders-api
  template:
    metadata:
      labels:
        app: orders-api
    spec:
      serviceAccountName: orders-api
      containers:
      - name: api
        image: 123456789.dkr.ecr.us-east-1.amazonaws.com/orders-api:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: orders-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: orders-secrets
              key: redis-url
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - orders-api
              topologyKey: kubernetes.io/hostname
```

**Flux automation:**

```yaml
# infrastructure/flux/orders-api-imagerepository.yaml
apiVersion: image.fluxcd.io/v1beta1
kind: ImageRepository
metadata:
  name: orders-api
spec:
  image: 123456789.dkr.ecr.us-east-1.amazonaws.com/orders-api
  interval: 1m

---
apiVersion: image.fluxcd.io/v1beta1
kind: ImagePolicy
metadata:
  name: orders-api
spec:
  imageRepositoryRef:
    name: orders-api
  policy:
    semver:
      range: '>=1.0.0 <2.0.0'

---
apiVersion: image.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: orders-api
spec:
  sourceRef:
    kind: GitRepository
    name: flux-system
  interval: 1m
  autoDiscoverBranches: true
  update:
    strategy: Setters
    path: ./infrastructure/flux
  commit:
    author:
      name: fluxcdbot
      email: fluxcdbot@example.com
    messageTemplate: 'Automated image update'
  push:
    branch: main
```

**Blue-green deployment (zero downtime):**

```bash
#!/bin/bash
# scripts/deploy.sh

CURRENT_ENV=$(aws ecs describe-services \
  --cluster orders-prod \
  --services orders-api \
  --query 'services[0].taskDefinition' \
  --output text | grep -oP 'orders-api:\K\d+' || echo "0")

NEW_ENV=$((CURRENT_ENV + 1))

# Register new task definition
aws ecs register-task-definition \
  --cli-input-json file://task-definition-v${NEW_ENV}.json

# Update service (creates new tasks, keeps old ones running)
aws ecs update-service \
  --cluster orders-prod \
  --service orders-api \
  --task-definition orders-api:${NEW_ENV}

# Wait for new tasks to be healthy
aws ecs wait services-stable \
  --cluster orders-prod \
  --services orders-api

# Monitor for 5 minutes
sleep 300

# Check error metrics
ERROR_RATE=$(aws cloudwatch get-metric-statistics \
  --namespace AWS/ECS \
  --metric-name Errors \
  --dimensions Name=ServiceName,Value=orders-api \
  --start-time $(date -u -d '5 minutes ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Sum \
  --query 'Datapoints[0].Sum' \
  --output text)

if (( $(echo "$ERROR_RATE > 10" | bc -l) )); then
  # Rollback
  aws ecs update-service \
    --cluster orders-prod \
    --service orders-api \
    --task-definition orders-api:${CURRENT_ENV}
  exit 1
fi

echo "✅ Deployment successful"
```


***

## 6.3 CLOUD INFRASTRUCTURE (IaC)

### 6.3.1 Infrastructure as Code (Terraform)

**All infrastructure defined in code. Version controlled. Reproducible.**

**Directory structure:**

```
infrastructure/
├─ terraform/
│  ├─ main.tf              # Main configuration
│  ├─ variables.tf         # Input variables
│  ├─ outputs.tf           # Output values
│  ├─ vpc.tf               # Network setup
│  ├─ rds.tf               # Database
│  ├─ elasticache.tf       # Redis
│  ├─ ecs.tf               # Container orchestration
│  ├─ alb.tf               # Load balancer
│  ├─ iam.tf               # Permissions
│  ├─ cloudwatch.tf        # Monitoring
│  ├─ terraform.tfvars     # Development values
│  ├─ prod.tfvars          # Production values
│  └─ .terraform.lock.hcl  # Locked versions
└─ kubernetes/             # K8s manifests
```

**Main Terraform configuration:**

```hcl
# infrastructure/terraform/main.tf
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket         = "orders-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Project     = "orders"
      Environment = var.environment
      ManagedBy   = "Terraform"
      CreatedAt   = timestamp()
    }
  }
}

# VPC + Networking
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "orders-${var.environment}"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  enable_vpn_gateway = false
  enable_dns_hostnames = true

  tags = {
    Name = "orders-vpc-${var.environment}"
  }
}

# RDS PostgreSQL
module "rds" {
  source = "terraform-aws-modules/rds/aws"
  version = "6.0.0"

  identifier = "orders-db-${var.environment}"

  engine               = "postgres"
  engine_version       = "15.3"
  family               = "postgres15"
  major_engine_version = "15"
  instance_class       = var.db_instance_class

  allocated_storage     = var.db_allocated_storage
  max_allocated_storage = var.db_max_allocated_storage

  db_name  = "orders_db"
  username = "postgres"
  password = random_password.db_password.result

  multi_az            = var.environment == "prod" ? true : false
  publicly_accessible = false

  backup_retention_period = var.environment == "prod" ? 30 : 7
  backup_window           = "03:00-04:00"
  maintenance_window      = "mon:04:00-mon:05:00"

  skip_final_snapshot = var.environment == "dev" ? true : false

  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = module.vpc.database_subnet_group_name

  enabled_cloudwatch_logs_exports = ["postgresql"]

  tags = {
    Name = "orders-db-${var.environment}"
  }
}

# Random password for RDS
resource "random_password" "db_password" {
  length  = 32
  special = true
}

# Store password in Secrets Manager
resource "aws_secretsmanager_secret" "db_password" {
  name = "orders/db/password-${var.environment}"
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id      = aws_secretsmanager_secret.db_password.id
  secret_string  = random_password.db_password.result
}

# ElastiCache Redis
module "redis" {
  source = "terraform-aws-modules/elasticache/aws"
  version = "1.0.0"

  name = "orders-cache-${var.environment}"

  engine               = "redis"
  engine_version       = "7.1"
  node_type            = var.redis_node_type
  num_cache_nodes      = var.environment == "prod" ? 3 : 1
  parameter_group_name = "default.redis7"
  port                 = 6379

  automatic_failover_enabled = var.environment == "prod" ? true : false
  multi_az_enabled          = var.environment == "prod" ? true : false

  at_rest_encryption_enabled = true
  transit_encryption_enabled = true

  security_group_ids = [aws_security_group.redis.id]
  subnet_group_name  = aws_elasticache_subnet_group.redis.name

  log_delivery_configuration = {
    slow_log = {
      cloudwatch_log_group      = aws_cloudwatch_log_group.redis_slow_log.name
      cloudwatch_log_group_arn  = "${aws_cloudwatch_log_group.redis_slow_log.arn}:*"
      enabled                   = true
    }
    engine_log = {
      cloudwatch_log_group      = aws_cloudwatch_log_group.redis_engine_log.name
      cloudwatch_log_group_arn  = "${aws_cloudwatch_log_group.redis_engine_log.arn}:*"
      enabled                   = true
    }
  }

  tags = {
    Name = "orders-cache-${var.environment}"
  }
}

# Output values (for reference)
output "rds_endpoint" {
  value       = module.rds.db_instance_endpoint
  description = "RDS database endpoint"
}

output "redis_endpoint" {
  value       = module.redis.primary_endpoint_address
  description = "Redis primary endpoint"
}

output "vpc_id" {
  value       = module.vpc.vpc_id
  description = "VPC ID"
}
```

**Apply Terraform:**

```bash
# Plan changes (dry run)
terraform plan -var-file=dev.tfvars -out=dev.tfplan

# Review plan
cat dev.tfplan

# Apply changes
terraform apply dev.tfplan

# Output values
terraform output rds_endpoint
terraform output redis_endpoint

# Destroy (careful!)
terraform destroy -var-file=dev.tfvars
```


***

### 6.3.2 Network Configuration

**Secure network isolation. Public/private subnets. Security groups.**

**Network diagram:**

```
┌─────────────────────────────────────────┐
│          VPC 10.0.0.0/16                 │
├─────────────────────────────────────────┤
│                                         │
│  PUBLIC SUBNET (10.0.101.0/24)          │
│  ├─ NAT Gateway                         │
│  ├─ Application Load Balancer (port 443)│
│  └─ Internet Gateway                    │
│                                         │
│  PRIVATE SUBNET (10.0.1.0/24)           │
│  ├─ ECS Tasks (API servers)             │
│  └─ Security Group: Allow ALB only      │
│                                         │
│  PRIVATE SUBNET (10.0.2.0/24)           │
│  ├─ RDS PostgreSQL                      │
│  └─ Security Group: Allow ECS only      │
│                                         │
│  PRIVATE SUBNET (10.0.3.0/24)           │
│  ├─ ElastiCache Redis                   │
│  └─ Security Group: Allow ECS only      │
│                                         │
└─────────────────────────────────────────┘

Traffic flow:
Internet → ALB (public) → ECS Tasks (private) → RDS/Redis (private)
```

**Security groups (Terraform):**

```hcl
# ALB security group (allows internet traffic)
resource "aws_security_group" "alb" {
  name        = "orders-alb-${var.environment}"
  description = "Security group for ALB"
  vpc_id      = module.vpc.vpc_id

  # Allow HTTPS from internet
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allow HTTP for redirect to HTTPS
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allow outbound traffic
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "orders-alb-sg-${var.environment}"
  }
}

# ECS security group (allows ALB only)
resource "aws_security_group" "ecs" {
  name        = "orders-ecs-${var.environment}"
  description = "Security group for ECS tasks"
  vpc_id      = module.vpc.vpc_id

  # Allow traffic from ALB
  ingress {
    from_port       = 3000
    to_port         = 3000
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  # Allow outbound to everything (needed for API calls, updates)
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "orders-ecs-sg-${var.environment}"
  }
}

# RDS security group (allows ECS only)
resource "aws_security_group" "rds" {
  name        = "orders-rds-${var.environment}"
  description = "Security group for RDS"
  vpc_id      = module.vpc.vpc_id

  # Allow PostgreSQL from ECS
  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.ecs.id]
  }

  # No outbound (database doesn't initiate connections)

  tags = {
    Name = "orders-rds-sg-${var.environment}"
  }
}

# Redis security group (allows ECS only)
resource "aws_security_group" "redis" {
  name        = "orders-redis-${var.environment}"
  description = "Security group for Redis"
  vpc_id      = module.vpc.vpc_id

  # Allow Redis from ECS
  ingress {
    from_port       = 6379
    to_port         = 6379
    protocol        = "tcp"
    security_groups = [aws_security_group.ecs.id]
  }

  # No outbound

  tags = {
    Name = "orders-redis-sg-${var.environment}"
  }
}
```


***

### 6.3.3 Load Balancer Configuration

**Distribute traffic across multiple app instances. Health checks. SSL termination.**

```hcl
# Application Load Balancer
resource "aws_lb" "main" {
  name               = "orders-alb-${var.environment}"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = module.vpc.public_subnets

  enable_deletion_protection = var.environment == "prod"
  enable_http2              = true
  enable_cross_zone_load_balancing = true

  tags = {
    Name = "orders-alb-${var.environment}"
  }
}

# Target group (where to send traffic)
resource "aws_lb_target_group" "api" {
  name        = "orders-api-tg-${var.environment}"
  port        = 3000
  protocol    = "HTTP"
  vpc_id      = module.vpc.vpc_id
  target_type = "ip"

  health_check {
    healthy_threshold   = 2
    unhealthy_threshold = 2
    timeout             = 3
    interval            = 30
    path                = "/health"
    matcher             = "200"
  }

  tags = {
    Name = "orders-api-tg-${var.environment}"
  }
}

# HTTPS listener (with SSL certificate)
resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.main.arn
  port              = "443"
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS-1-2-2017-01"
  certificate_arn   = aws_acm_certificate.main.arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.api.arn
  }
}

# HTTP redirect to HTTPS
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type = "redirect"

    redirect {
      port        = "443"
      protocol    = "HTTPS"
      status_code = "HTTP_301"
    }
  }
}

# SSL certificate (auto-renewal)
resource "aws_acm_certificate" "main" {
  domain_name       = "api.company.com"
  validation_method = "DNS"

  lifecycle {
    create_before_destroy = true
  }

  tags = {
    Name = "orders-api-cert-${var.environment}"
  }
}

# Output for CloudFront
output "alb_dns_name" {
  value = aws_lb.main.dns_name
}
```


***

### 6.3.4 Auto-Scaling Policies

**Automatically add/remove instances based on demand.**

```hcl
# ECS Auto Scaling
resource "aws_ecs_service" "api" {
  name            = "orders-api"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.api.arn
  desired_count   = var.ecs_desired_count

  launch_type = "FARGATE"

  load_balancer {
    target_group_arn = aws_lb_target_group.api.arn
    container_name   = "api"
    container_port   = 3000
  }

  network_configuration {
    subnets          = module.vpc.private_subnets
    security_groups  = [aws_security_group.ecs.id]
    assign_public_ip = false
  }

  depends_on = [aws_lb_listener.https]
}

# Auto Scaling Target (tracks the service)
resource "aws_autoscaling_target" "ecs_target" {
  max_capacity       = var.environment == "prod" ? 20 : 5
  min_capacity       = var.environment == "prod" ? 3 : 1
  resource_id        = "service/${aws_ecs_cluster.main.name}/${aws_ecs_service.api.name}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
}

# Scale up on high CPU (> 70%)
resource "aws_autoscaling_policy" "ecs_policy_up" {
  name               = "orders-scale-up-${var.environment}"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_autoscaling_target.ecs_target.resource_id
  scalable_dimension = aws_autoscaling_target.ecs_target.scalable_dimension
  service_namespace  = aws_autoscaling_target.ecs_target.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }

    target_value = 70.0
    scale_out_cooldown = 60
    scale_in_cooldown  = 300
  }
}

# Scale up on high memory (> 80%)
resource "aws_autoscaling_policy" "ecs_policy_memory" {
  name               = "orders-scale-memory-${var.environment}"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_autoscaling_target.ecs_target.resource_id
  scalable_dimension = aws_autoscaling_target.ecs_target.scalable_dimension
  service_namespace  = aws_autoscaling_target.ecs_target.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageMemoryUtilization"
    }

    target_value = 80.0
  }
}

# Target group attribute for deregistration delay
resource "aws_lb_target_group" "api" {
  # ... existing config ...

  deregistration_delay = 30

  stickiness {
    type            = "lb_cookie"
    cookie_duration = 86400
    enabled         = false  # Disable for stateless API
  }
}
```

**Scaling behavior:**

```
Load: 20 req/sec
├─ 1 instance @ 85% CPU
├─ Trigger: CPU > 70%
├─ Action: Launch 1 new instance
├─ Wait: 60 seconds (scale_out_cooldown)
└─ Result: 2 instances @ 42% CPU each

Load: 5 req/sec
├─ 2 instances @ 30% CPU each
├─ Trigger: CPU < 70% for 5 minutes
├─ Action: Terminate 1 instance
├─ Wait: 300 seconds (scale_in_cooldown)
└─ Result: 1 instance @ 60% CPU
```


***

## 6.4 MONITORING \& LOGGING

### 6.4.1 Log Aggregation Setup

**All logs centralized. Searchable. Queryable. Alertable.**

**CloudWatch Logs (AWS native):**

```hcl
# CloudWatch Log Group for backend
resource "aws_cloudwatch_log_group" "api" {
  name              = "/orders/api/${var.environment}"
  retention_in_days = var.environment == "prod" ? 30 : 7

  tags = {
    Name = "orders-api-logs-${var.environment}"
  }
}

# CloudWatch Log Group for database
resource "aws_cloudwatch_log_group" "rds" {
  name              = "/orders/rds/${var.environment}"
  retention_in_days = var.environment == "prod" ? 30 : 7

  tags = {
    Name = "orders-rds-logs-${var.environment}"
  }
}
```

**Backend logging configuration:**

```typescript
// backend/src/logger.ts
import winston from 'winston';

// JSON logs for CloudWatch parsing
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    environment: process.env.NODE_ENV,
    service: 'orders-api',
    version: process.env.APP_VERSION
  },
  transports: [
    // Console for development
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.printf(({ timestamp, level, message, ...meta }) => {
          return `${timestamp} [${level}] ${message} ${
            Object.keys(meta).length ? JSON.stringify(meta, null, 2) : ''
          }`;
        })
      )
    }),
    // CloudWatch for production
    new WinstonCloudWatch({
      logGroupName: `/orders/api/${process.env.NODE_ENV}`,
      logStreamName: `instance-${hostname()}`,
      awsRegion: 'us-east-1',
      messageFormatter: ({ level, message, meta }) => {
        return JSON.stringify({ level, message, meta });
      }
    })
  ]
});

export default logger;
```

**Structured logging in code:**

```typescript
// Log with context
logger.info('Order created', {
  orderId: 'ORD-123',
  customerId: 'CUST-001',
  total: 99.99,
  items: 2,
  duration_ms: 145,
  request_id: req.id
});

// Log errors with stack trace
logger.error('Database connection failed', {
  error: err.message,
  code: err.code,
  host: config.database.host,
  request_id: req.id
});

// Log metrics
logger.info('API metrics', {
  metric_type: 'performance',
  endpoint: 'POST /api/orders',
  response_time_ms: 245,
  status_code: 201,
  db_query_time_ms: 123,
  cache_hit: false,
  request_id: req.id
});
```


***

### 6.4.2 Metrics Collection

**CPU, memory, requests, latency, errors—all measured.**

**CloudWatch Metrics (Terraform):**

```hcl
# Custom metric: Orders created per minute
resource "aws_cloudwatch_metric_alarm" "orders_high" {
  alarm_name          = "orders-creation-rate-high-${var.environment}"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "OrdersCreated"
  namespace           = "Orders"
  period              = 60
  statistic           = "Sum"
  threshold           = 100
  alarm_description   = "Alert when order creation rate exceeds 100/min"
  treat_missing_data  = "notBreaching"

  alarm_actions = [aws_sns_topic.alerts.arn]
}

# ECS metrics
resource "aws_cloudwatch_metric_alarm" "ecs_cpu_high" {
  alarm_name          = "ecs-cpu-high-${var.environment}"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/ECS"
  period              = 300
  statistic           = "Average"
  threshold           = 80
  alarm_description   = "Alert when ECS CPU > 80%"

  dimensions = {
    ServiceName = aws_ecs_service.api.name
    ClusterName = aws_ecs_cluster.main.name
  }

  alarm_actions = [aws_sns_topic.alerts.arn]
}

# RDS metrics
resource "aws_cloudwatch_metric_alarm" "rds_cpu_high" {
  alarm_name          = "rds-cpu-high-${var.environment}"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/RDS"
  period              = 300
  statistic           = "Average"
  threshold           = 80

  dimensions = {
    DBInstanceIdentifier = module.rds.db_instance_id
  }

  alarm_actions = [aws_sns_topic.critical.arn]
}
```

**Application metrics (Node.js):**

```typescript
// backend/src/metrics.ts
import client from 'prom-client';

// Create metrics
export const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request latency',
  labelNames: ['method', 'route', 'status'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

export const ordersCreated = new client.Counter({
  name: 'orders_created_total',
  help: 'Total orders created',
  labelNames: ['status']
});

export const databaseQueryDuration = new client.Histogram({
  name: 'database_query_duration_seconds',
  help: 'Database query latency',
  labelNames: ['operation', 'table'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1]
});

export const cacheHitRate = new client.Gauge({
  name: 'cache_hit_rate',
  help: 'Cache hit rate percentage',
  labelNames: ['cache_name']
});
```

**Collect metrics in middleware:**

```typescript
// Middleware to track request duration
app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration
      .labels(req.method, req.route?.path || req.path, res.statusCode)
      .observe(duration);
  });

  next();
});

// Track order creation
app.post('/api/v1/orders', async (req, res) => {
  try {
    const order = await createOrder(req.body);
    ordersCreated.labels('success').inc();
    res.json(order);
  } catch (error) {
    ordersCreated.labels('error').inc();
    res.status(400).json({ error: error.message });
  }
});

// Expose metrics endpoint for Prometheus
app.get('/metrics', (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(client.register.metrics());
});
```


***

### 6.4.3 Distributed Tracing

**Follow request across services. See where it slows down.**

**Jaeger setup (Terraform):**

```hcl
# Jaeger container for tracing
resource "aws_ecs_task_definition" "jaeger" {
  family                   = "jaeger"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = "256"
  memory                   = "512"

  container_definitions = jsonencode([
    {
      name      = "jaeger"
      image     = "jaegertracing/all-in-one:latest"
      essential = true
      portMappings = [
        {
          containerPort = 6831
          protocol      = "udp"
        },
        {
          containerPort = 16686
          protocol      = "tcp"
        }
      ]
      logConfiguration = {
        logDriver = "awslogs"
        options = {
          "awslogs-group"         = aws_cloudwatch_log_group.jaeger.name
          "awslogs-region"        = var.aws_region
          "awslogs-stream-prefix" = "ecs"
        }
      }
    }
  ])
}

# Jaeger service
resource "aws_ecs_service" "jaeger" {
  name            = "jaeger"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.jaeger.arn
  desired_count   = 1
  launch_type     = "FARGATE"

  network_configuration {
    subnets         = module.vpc.private_subnets
    security_groups = [aws_security_group.ecs.id]
  }
}
```

**Backend tracing:**

```typescript
// backend/src/tracing.ts
import { BasicTracerProvider, ConsoleSpanExporter, SimpleSpanProcessor } from '@opentelemetry/sdk-trace-node';
import { JaegerExporter } from '@opentelemetry/exporter-jaeger-http';
import { registerInstrumentations } from '@opentelemetry/auto-instrumentations-node';

const provider = new BasicTracerProvider();

// Export to Jaeger
const jaegerExporter = new JaegerExporter({
  host: process.env.JAEGER_HOST || 'localhost',
  port: 14268
});

provider.addSpanProcessor(new SimpleSpanProcessor(jaegerExporter));

registerInstrumentations({
  tracerProvider: provider
});

export const tracer = provider.getTracer('orders-api');
```

**Create spans for operations:**

```typescript
// Manual span creation
const span = tracer.startSpan('create-order', {
  attributes: {
    'customer_id': req.user.id,
    'item_count': req.body.items.length
  }
});

try {
  const order = await createOrder(req.body);
  span.addEvent('order_created', { order_id: order.id });
} catch (error) {
  span.recordException(error);
} finally {
  span.end();
}

// Automatic instrumentation for HTTP calls
const response = await fetch('https://api.stripe.com/...');
// Span created automatically for this HTTP call
```

**Jaeger UI:**

```
Visit http://localhost:16686

├─ Find traces by service: orders-api
├─ Filter by operation: POST /api/orders
├─ See request breakdown:
│  ├─ Total: 245ms
│  ├─ Database query: 123ms
│  ├─ Stripe API: 89ms
│  ├─ Redis cache: 2ms
│  └─ Other: 31ms
├─ Identify bottleneck (database query at 50%)
└─ Optimize that first
```


***

### 6.4.4 Alert Rules Definition

**Issues detected automatically. Team notified immediately.**

**Alert rules:**

```hcl
# SNS topic for alerts
resource "aws_sns_topic" "alerts" {
  name = "orders-alerts-${var.environment}"
}

resource "aws_sns_topic_subscription" "alerts_email" {
  topic_arn = aws_sns_topic.alerts.arn
  protocol  = "email"
  endpoint  = var.alert_email
}

resource "aws_sns_topic" "critical" {
  name = "orders-critical-${var.environment}"
}

resource "aws_sns_topic_subscription" "critical_slack" {
  topic_arn = aws_sns_topic.critical.arn
  protocol  = "https"
  endpoint  = var.slack_webhook_url
}

# High error rate alert (> 5% in 5 minutes)
resource "aws_cloudwatch_metric_alarm" "error_rate_high" {
  alarm_name          = "error-rate-high-${var.environment}"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  threshold           = 5.0  # 5%
  alarm_description   = "Alert when error rate > 5%"

  metrics = [
    {
      id          = "error_rate"
      expression  = "errors / requests * 100"
      label       = "Error Rate"
      return_data = true
    },
    {
      id          = "errors"
      metric_name = "Errors"
      namespace   = "AWS/ApplicationELB"
      statistic   = "Sum"
      period      = 300
    },
    {
      id          = "requests"
      metric_name = "RequestCount"
      namespace   = "AWS/ApplicationELB"
      statistic   = "Sum"
      period      = 300
    }
  ]

  alarm_actions = [aws_sns_topic.critical.arn]
}

# Database connection timeout alert
resource "aws_cloudwatch_metric_alarm" "db_timeout" {
  alarm_name          = "db-timeout-${var.environment}"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = 1
  metric_name         = "DatabaseConnectionErrors"
  namespace           = "Orders"
  period              = 300
  statistic           = "Sum"
  threshold           = 5
  alarm_description   = "Alert on database connection errors"
  treat_missing_data  = "notBreaching"

  alarm_actions = [aws_sns_topic.critical.arn]
}

# Deployment failure alert
resource "aws_cloudwatch_event_rule" "ecs_deployment_failure" {
  name        = "ecs-deployment-failure-${var.environment}"
  description = "Alert on ECS deployment failures"

  event_pattern = jsonencode({
    source      = ["aws.ecs"]
    detail-type = ["ECS Service Action"]
    detail = {
      service      = [aws_ecs_service.api.name]
      clusterArn   = [aws_ecs_cluster.main.arn]
      eventName    = ["deploymentStateChanged"]
      eventReason  = ["ECS deployment arn:aws:ecs:... stopped."]
    }
  })
}

resource "aws_cloudwatch_event_target" "ecs_failure_sns" {
  rule      = aws_cloudwatch_event_rule.ecs_deployment_failure.name
  target_id = "SendToSNS"
  arn       = aws_sns_topic.critical.arn
  role_arn  = aws_iam_role.cloudwatch_role.arn
}
```

**Alert thresholds (production):**

```
CRITICAL (page engineer immediately):
├─ Error rate > 5%
├─ Response time p99 > 5s
├─ Database offline
├─ Deployment failed
└─ Any instance unhealthy

WARNING (notify team):
├─ Error rate > 1%
├─ Response time p99 > 1s
├─ CPU > 80%
├─ Memory > 85%
└─ Cache hit rate < 50%

INFO (log only, no notification):
├─ Deployment successful
├─ Scaling triggered
└─ Scheduled backup completed
```


***

### 6.4.5 Dashboard Creation

**Single pane of glass. See system health instantly.**

**Grafana dashboard (JSON):**

```json
{
  "dashboard": {
    "title": "Orders API - Production",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~'5..'}[5m]) / rate(http_requests_total[5m]) * 100"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Response Time (p99)",
        "targets": [
          {
            "expr": "histogram_quantile(0.99, http_request_duration_seconds)"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Database Query Time",
        "targets": [
          {
            "expr": "rate(database_query_duration_seconds_sum[5m]) / rate(database_query_duration_seconds_count[5m])"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Cache Hit Rate",
        "targets": [
          {
            "expr": "cache_hit_rate"
          }
        ],
        "type": "gauge"
      },
      {
        "title": "Active Connections",
        "targets": [
          {
            "expr": "pg_stat_activity_count"
          }
        ],
        "type": "gauge"
      },
      {
        "title": "ECS Task Count",
        "targets": [
          {
            "expr": "ecs_service_running_count"
          }
        ],
        "type": "stat"
      },
      {
        "title": "Recent Errors",
        "targets": [
          {
            "expr": "topk(10, increase(errors_total[5m]))"
          }
        ],
        "type": "table"
      }
    ]
  }
}
```


***

## 6.5 SECRETS MANAGEMENT

### 6.5.1 Secrets Storage Solution

**API keys, passwords, tokens—stored securely. Never in code or logs.**

**AWS Secrets Manager (Terraform):**

```hcl
# Store database password
resource "aws_secretsmanager_secret" "db_password" {
  name                    = "orders/db/password-${var.environment}"
  recovery_window_in_days = 7
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id      = aws_secretsmanager_secret.db_password.id
  secret_string  = random_password.db_password.result
}

# Store JWT secret
resource "aws_secretsmanager_secret" "jwt_secret" {
  name = "orders/jwt/secret-${var.environment}"
}

resource "aws_secretsmanager_secret_version" "jwt_secret" {
  secret_id     = aws_secretsmanager_secret.jwt_secret.id
  secret_string = random_password.jwt_secret.result
}

# Store Stripe API key
resource "aws_secretsmanager_secret" "stripe_key" {
  name = "orders/stripe/secret-key-${var.environment}"
}

# Manual input for Stripe key (don't generate, it's an external API key)
resource "aws_secretsmanager_secret_version" "stripe_key" {
  secret_id     = aws_secretsmanager_secret.stripe_key.id
  secret_string = var.stripe_secret_key
}

# Store all secrets as JSON for easy access
resource "aws_secretsmanager_secret" "app_secrets" {
  name = "orders/app-secrets-${var.environment}"
}

resource "aws_secretsmanager_secret_version" "app_secrets" {
  secret_id = aws_secretsmanager_secret.app_secrets.id
  secret_string = jsonencode({
    database_url  = "postgresql://user:pass@host:5432/db"
    redis_url     = "redis://host:6379"
    jwt_secret    = random_password.jwt_secret.result
    stripe_key    = var.stripe_secret_key
    sendgrid_key  = var.sendgrid_key
  })
}
```

**Load secrets in application:**

```typescript
// backend/src/secrets.ts
import { SecretsManager } from '@aws-sdk/client-secrets-manager';

const sm = new SecretsManager({ region: 'us-east-1' });

let cachedSecrets: Record<string, string> = {};

export async function getSecret(name: string): Promise<string> {
  // Check cache first (avoid repeated API calls)
  if (cachedSecrets[name]) {
    return cachedSecrets[name];
  }

  try {
    const response = await sm.getSecretValue({ SecretId: name });
    const secret = response.SecretString || '';
    
    // Cache for 1 hour
    cachedSecrets[name] = secret;
    setTimeout(() => delete cachedSecrets[name], 3600000);
    
    return secret;
  } catch (error) {
    console.error(`Failed to retrieve secret ${name}:`, error);
    // Fallback to environment variable
    return process.env[name] || '';
  }
}

// Usage:
const jwtSecret = await getSecret('orders/jwt/secret-prod');
const dbPassword = await getSecret('orders/db/password-prod');
```

**Or use via ECS task role:**

```hcl
# ECS task role with Secrets Manager permissions
resource "aws_iam_role" "ecs_task_role" {
  name = "ecs-task-role-${var.environment}"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ecs-tasks.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy" "ecs_secrets_policy" {
  name = "ecs-secrets-policy-${var.environment}"
  role = aws_iam_role.ecs_task_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "secretsmanager:GetSecretValue",
          "secretsmanager:DescribeSecret"
        ]
        Resource = [
          aws_secretsmanager_secret.db_password.arn,
          aws_secretsmanager_secret.jwt_secret.arn,
          aws_secretsmanager_secret.stripe_key.arn
        ]
      }
    ]
  })
}

# Pass secrets to ECS container via environment variables
resource "aws_ecs_task_definition" "api" {
  # ... other config ...
  
  container_definitions = jsonencode([
    {
      name = "api"
      # ... other config ...
      
      secrets = [
        {
          name      = "DATABASE_PASSWORD"
          valueFrom = aws_secretsmanager_secret.db_password.arn
        },
        {
          name      = "JWT_SECRET"
          valueFrom = aws_secretsmanager_secret.jwt_secret.arn
        },
        {
          name      = "STRIPE_SECRET_KEY"
          valueFrom = aws_secretsmanager_secret.stripe_key.arn
        }
      ]
    }
  ])
}
```


***

### 6.5.2 Rotation Procedures

**Secrets rotated regularly. Old secrets invalidated.**

**Automated rotation (Terraform):**

```hcl
# Lambda function to rotate secrets
resource "aws_lambda_function" "rotate_db_password" {
  filename = "lambda_rotate_secret.zip"
  function_name = "rotate-db-password-${var.environment}"
  role = aws_iam_role.lambda_role.arn
  handler = "index.handler"
  timeout = 60
  runtime = "python3.11"

  environment {
    variables = {
      SECRETS_MANAGER_ENDPOINT = "https://secretsmanager.${var.aws_region}.amazonaws.com"
      RDS_ENDPOINT = module.rds.db_instance_endpoint
    }
  }
}

# Trigger rotation every 30 days
resource "aws_secretsmanager_secret_rotation" "db_password" {
  secret_id           = aws_secretsmanager_secret.db_password.id
  rotation_enabled    = true
  rotation_lambda_arn = aws_lambda_function.rotate_db_password.arn
  rotation_rules {
    automatically_after_days = 30
  }
}
```

**Rotation Lambda (Python):**

```python
# lambda_rotate_secret.py
import boto3
import psycopg2
import os

sm = boto3.client('secretsmanager')
rds_endpoint = os.environ['RDS_ENDPOINT']

def lambda_handler(event, context):
    secret_id = event['SecretId']
    
    # Get current secret
    current = sm.get_secret_value(SecretId=secret_id)['SecretString']
    
    # Generate new password
    new_password = sm.get_random_password(PasswordLength=32)['RandomPassword']
    
    # Update in database
    conn = psycopg2.connect(
        host=rds_endpoint,
        database='orders_db',
        user='postgres',
        password=current
    )
    cur = conn.cursor()
    cur.execute(f"ALTER USER postgres WITH PASSWORD %s", (new_password,))
    conn.commit()
    conn.close()
    
    # Store new secret
    sm.put_secret_value(
        SecretId=secret_id,
        SecretString=new_password,
        VersionStages=['AWSCURRENT']
    )
    
    return {'statusCode': 200, 'body': 'Secret rotated'}
```


***

## 6.6 DESIGN REVIEW GATES

### Gate 1: Infrastructure Security Review

**Before deploying to production, security team reviews infrastructure.**

**Checklist:**

Network Security:
☐ Public subnets only have ALB (not application servers)
☐ Private subnets have no internet gateway (NAT only for outbound)
☐ Security groups follow least privilege (minimum required access)
☐ RDS not publicly accessible
☐ Redis not publicly accessible
☐ Network ACLs reviewed

Access Control:
☐ IAM roles follow least privilege
☐ ECS task role has only needed Secrets Manager permissions
☐ No wildcard `*` permissions
☐ Cross-account access (if applicable) properly restricted
☐ Service accounts not using root credentials

Secrets:
☐ No secrets in code, Dockerfile, environment files
☐ All secrets in Secrets Manager
☐ Rotation configured for long-lived secrets
☐ Secret access logged in CloudTrail
☐ Encryption at rest enabled

Encryption:
☐ TLS 1.2+ for all traffic (ALB → ECS, ECS → RDS)
☐ SSL certificate valid and auto-renewed
☐ RDS encryption enabled at rest
☐ Redis encryption enabled (in transit)
☐ S3 buckets (if applicable) encrypted

Monitoring:
☐ CloudTrail enabled (audit all API calls)
☐ VPC Flow Logs enabled (network traffic)
☐ CloudWatch Logs retention configured
☐ ALB access logs stored in S3

Compliance:
☐ Meet regulatory requirements (PCI, HIPAA, GDPR if applicable)
☐ Data retention policies documented
☐ Backup strategy documented
☐ Disaster recovery plan documented

Sign-off:
☐ Security team approved
☐ Infrastructure team approved
☐ Documented in compliance tracker

***

### Gate 2: Disaster Recovery \& Backup Plan Review

**Can you recover from total data loss in < 4 hours?**

**Checklist:**

Backups:
☐ Database backed up daily (automated)
☐ Backups stored in separate region (redundancy)
☐ Backup retention: 30 days (production)
☐ Backup encryption enabled
☐ Backup tested (restore to test environment quarterly)

RTO/RPO targets:
☐ RTO (Recovery Time Objective): < 1 hour
☐ RPO (Recovery Point Objective): < 1 hour
☐ Documented in runbook

Failover:
☐ Multi-AZ RDS enabled (automatic failover)
☐ Multi-AZ ElastiCache enabled
☐ Application can handle region failover (if needed)
☐ DNS failover documented

Runbooks:
☐ Database recovery procedure documented
☐ Secret re-generation procedure documented
☐ SSL certificate re-issue procedure documented
☐ All runbooks in version control, accessible offline

Testing:
☐ Disaster recovery drill scheduled (quarterly)
☐ Last drill completed successfully
☐ Issues from last drill resolved

Sign-off:
☐ Operations team approved
☐ Disaster recovery plan filed

***

## SUMMARY: WHAT YOU'VE BUILT

By end of Phase 6, you have:

✅ **Development Environment**

- Docker Compose with PostgreSQL, Redis, backend, frontend
- Hot reload configured (code changes reflect instantly)
- Mock API server for frontend independence
- Database migrations and seeds (reproducible state)
- Environment variable management (secrets safe)

✅ **CI/CD Pipeline**

- Automated testing (unit, integration, E2E)
- Code quality gates (ESLint, Prettier, TypeScript)
- Security scanning (dependencies, containers, SAST)
- Automated deployment (staging → production)
- Blue-green deployments (zero downtime)

✅ **Cloud Infrastructure**

- VPC with public/private subnets (secure network)
- RDS PostgreSQL (managed, backed up, secure)
- ElastiCache Redis (high availability, encryption)
- Application Load Balancer (SSL, health checks)
- Auto-scaling policies (handle traffic spikes)
- All defined as code (Terraform, reproducible)

✅ **Monitoring \& Logging**

- Centralized logs (CloudWatch, searchable)
- Metrics collection (CPU, memory, requests, latency)
- Distributed tracing (see where requests slow down)
- Alert rules (critical issues detected immediately)
- Dashboards (health visible at a glance)

✅ **Secrets Management**

- Secrets stored securely (AWS Secrets Manager)
- Access controlled (IAM roles)
- Rotated regularly (automated)
- Never logged or exposed

✅ **Security Review Gates**

- Infrastructure security approved
- Disaster recovery plan in place
- Backup/restore tested
- Runbooks documented

***

## CRITICAL INFRASTRUCTURE DECISIONS

**These decisions eliminate production incidents:**

1. **Private subnets for apps** → Attackers can't reach app directly (must go through ALB)
2. **RDS encryption at rest** → Database breach doesn't expose data
3. **JWT instead of session storage** → Scales to millions without bottleneck
4. **CloudWatch logs** → Can debug any production issue post-mortem
5. **Auto-scaling** → Traffic spike doesn't cause outage, just more instances
6. **Blue-green deployments** → New bugs found by monitoring, auto-rollback triggered
7. **Secrets rotation** → Leaked key becomes worthless after 30 days
8. **Distributed tracing** → Know exactly which service/query caused slowdown

**20-year veteran's take:** I've seen billion-dollar outages from infrastructure decisions made in hours. This infrastructure is bulletproof. Now the only outages come from bugs in code, not infrastructure. That's the goal.

***

**NEXT PHASE: Phase 7 (Database Design)**

Infrastructure is ready. Now design the database schema informed by all previous phases.

Frontend → Backend → Infrastructure → **Database** dependency chain complete.

Database design uses patterns learned from API (Phase 4), Infrastructure (Phase 6).
</query>

# PHASE 6: INFRASTRUCTURE SETUP

**Why Infrastructure Matters:**
Infrastructure is where 80% of production bugs hide. Bad infra design = outages, slow deploys, leaked secrets, security breaches, impossible debugging. A 20-year veteran knows: infrastructure decisions made here cost 10x more to fix later. Get it right now—reproducible, automated, secure, observable.

***

## 6.1 DEVELOPMENT ENVIRONMENT

### 6.1.1 Docker Setup for Local Development

**Goal: Every developer runs identical environment. "Works on my machine" never happens again.**

**Why Docker:**

- Backend dev runs Node + PostgreSQL + Redis + OpenSearch in 3 minutes
- Frontend dev gets same backend for testing
- No "but it works on my Mac" vs "broken on Linux"
- Production runs same Docker image locally (catch problems early)
- Onboard new engineer in 30 minutes, not 3 days

**Docker Compose structure:**

```yaml
# docker-compose.yml (development)
version: '3.8'

services:
  # PostgreSQL database
  postgres:
    image: postgres:15-alpine
    container_name: orders-db-dev
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: dev_user
      POSTGRES_PASSWORD: dev_password
      POSTGRES_DB: orders_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./sql/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dev_user"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis cache
  redis:
    image: redis:7-alpine
    container_name: orders-cache-dev
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Backend API
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    container_name: orders-api-dev
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://dev_user:dev_password@postgres:5432/orders_db
      REDIS_URL: redis://redis:6379
      JWT_SECRET: dev_secret_key_not_for_production
      STRIPE_SECRET_KEY: sk_test_...
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./backend/src:/app/src
      - /app/node_modules
    command: npm run dev

  # Frontend development server
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    container_name: orders-ui-dev
    ports:
      - "5173:5173"
    environment:
      VITE_API_URL: http://localhost:3000/api
    depends_on:
      - api
    volumes:
      - ./frontend/src:/app/src
      - /app/node_modules
    command: npm run dev

volumes:
  postgres_data:
    driver: local

networks:
  default:
    name: orders-dev-network
```

**Dockerfile for Node backend (development):**

```dockerfile
# Dockerfile.dev (development with hot reload)
FROM node:20-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Copy source
COPY src ./src
COPY tsconfig.json ./

# Volume mount for hot reload
VOLUME ["/app/src"]

EXPOSE 3000

CMD ["npm", "run", "dev"]
```

**Dockerfile for Node backend (production):**

```dockerfile
# Dockerfile.prod (multi-stage, optimized)
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY src ./src
COPY tsconfig.json ./

# Build TypeScript
RUN npm run build

# Production image (minimal)
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY --from=builder /app/dist ./dist

EXPOSE 3000

CMD ["node", "dist/index.js"]
```

**Start development:**

```bash
# Terminal 1: Start all services
docker-compose up

# Terminal 2: Run migrations (one-time)
docker-compose exec api npm run migrate

# Terminal 3: Run seeds (populate test data)
docker-compose exec api npm run seed

# Access
# Frontend: http://localhost:5173
# Backend: http://localhost:3000
# Database: localhost:5432 (psql -h localhost -U dev_user -d orders_db)
# Redis: localhost:6379 (redis-cli)
```

**Useful commands:**

```bash
# View logs
docker-compose logs api          # Backend logs
docker-compose logs -f postgres  # Follow postgres logs

# Execute commands in container
docker-compose exec api npm test # Run tests
docker-compose exec postgres psql -U dev_user -d orders_db  # Connect to DB

# Reset everything
docker-compose down -v          # Remove volumes too (resets DB)
docker-compose up              # Rebuild and start fresh

# Database inspection
docker-compose exec postgres psql -U dev_user -d orders_db
\dt                            # List tables
SELECT * FROM users;           # Query
\q                             # Exit
```


***

### 6.1.2 Database Initialization \& Seeding

**Reproducible data: Every developer starts with same initial state.**

**Init script (run once on container start):**

```sql
-- sql/init.sql
-- Runs automatically when postgres container starts

-- Create extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

-- Create schemas
CREATE SCHEMA IF NOT EXISTS public;

-- Create tables (application does this via migrations)
-- This init script just creates extensions + databases

-- Create dedicated user for app (not root)
CREATE USER app_user WITH PASSWORD 'app_password';
GRANT CONNECT ON DATABASE orders_db TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT CREATE ON SCHEMA public TO app_user;
```

**Database migrations (version control for schema):**

```typescript
// backend/db/migrations/001_create_users.ts
import { Knex } from "knex";

export async function up(knex: Knex): Promise<void> {
  return knex.schema.createTable("users", (table) => {
    table.uuid("id").primary().defaultTo(knex.raw("uuid_generate_v4()"));
    table.string("email").unique().notNullable();
    table.string("password_hash").notNullable();
    table.enum("role", ["user", "admin"]).defaultTo("user");
    table.timestamps(true, true);
    table.index("email");
  });
}

export async function down(knex: Knex): Promise<void> {
  return knex.schema.dropTable("users");
}
```

**Run migrations:**

```bash
# Terminal
npm run migrate:latest  # Run all pending migrations
npm run migrate:up     # Run next migration
npm run migrate:down   # Rollback last migration
npm run migrate:reset  # Drop all tables, run from start
```

**Seed script (populate test data):**

```typescript
// backend/db/seeds/01_seed_users.ts
import { Knex } from "knex";
import bcrypt from "bcrypt";

export async function seed(knex: Knex): Promise<void> {
  // Clear existing data
  await knex("orders").del();
  await knex("users").del();

  // Insert test users
  await knex("users").insert([
    {
      id: "11111111-1111-1111-1111-111111111111",
      email: "admin@example.com",
      password_hash: await bcrypt.hash("admin123", 10),
      role: "admin",
    },
    {
      id: "22222222-2222-2222-2222-222222222222",
      email: "user@example.com",
      password_hash: await bcrypt.hash("user123", 10),
      role: "user",
    },
  ]);

  // Insert test orders
  await knex("orders").insert([
    {
      id: "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
      customer_id: "22222222-2222-2222-2222-222222222222",
      total: 99.99,
      status: "pending",
    },
    {
      id: "bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb",
      customer_id: "22222222-2222-2222-2222-222222222222",
      total: 149.99,
      status: "shipped",
    },
  ]);
}
```

**Run seeds:**

```bash
npm run seed  # Populate with test data
```


***

### 6.1.3 Mock API Server Setup

**Frontend can develop without waiting for backend. Use mock API.**

**Mock server with MSW (Mock Service Worker):**

```typescript
// frontend/src/mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  // GET /api/orders
  http.get('*/api/v1/orders', () => {
    return HttpResponse.json({
      status: 'success',
      data: [
        {
          id: 'ORD-001',
          customer_id: 'CUST-001',
          total: 99.99,
          status: 'pending',
          created_at: '2024-01-15T10:30:00Z'
        },
        {
          id: 'ORD-002',
          customer_id: 'CUST-001',
          total: 149.99,
          status: 'shipped',
          created_at: '2024-01-14T14:22:00Z'
        }
      ],
      metadata: {
        total: 2,
        limit: 20,
        offset: 0
      }
    });
  }),

  // POST /api/orders
  http.post('*/api/v1/orders', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json(
      {
        status: 'success',
        data: {
          id: `ORD-${Math.random().toString().slice(2, 6)}`,
          customer_id: body.customer_id,
          total: body.total,
          status: 'pending',
          created_at: new Date().toISOString()
        }
      },
      { status: 201 }
    );
  }),

  // POST /api/auth/login
  http.post('*/api/v1/auth/login', () => {
    return HttpResponse.json({
      status: 'success',
      data: {
        access_token: 'mock_jwt_token_here',
        user: {
          id: 'USER-001',
          email: 'user@example.com',
          role: 'user'
        }
      }
    });
  })
];
```

**Setup MSW:**

```typescript
// frontend/src/mocks/setup.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

// Only setup mock server in development, not production
export const server = setupServer(...handlers);

if (process.env.NODE_ENV === 'development') {
  server.listen({ onUnhandledRequest: 'warn' });
}
```

**Use in tests:**

```typescript
// frontend/src/__tests__/OrdersList.test.tsx
import { render, screen } from '@testing-library/react';
import { server } from '../mocks/setup';
import { OrdersList } from '../components/OrdersList';

describe('OrdersList', () => {
  it('renders orders from mock API', async () => {
    render(<OrdersList />);
    
    // Wait for requests to complete
    await screen.findByText('ORD-001');
    
    expect(screen.getByText('$99.99')).toBeInTheDocument();
    expect(screen.getByText('$149.99')).toBeInTheDocument();
  });

  it('shows error when API fails', async () => {
    // Override specific handler for this test
    server.use(
      http.get('*/api/v1/orders', () => {
        return HttpResponse.error();
      })
    );

    render(<OrdersList />);
    
    await screen.findByText(/error/i);
  });
});
```

**Toggle between real and mock:**

```typescript
// frontend/src/api/client.ts
const API_BASE = process.env.REACT_APP_API_URL || 'http://localhost:3000';

const USE_MOCK_API = process.env.REACT_APP_USE_MOCK_API === 'true';

if (USE_MOCK_API) {
  // Import mock handlers
  await import('../mocks/setup');
}

export const apiClient = axios.create({
  baseURL: API_BASE
});
```

**Run with mock:**

```bash
# Terminal
REACT_APP_USE_MOCK_API=true npm run dev
```


***

### 6.1.4 Hot Reload Configuration

**Code change → App reloads in browser instantly (state preserved).**

**Vite hot reload (frontend):**

```typescript
// frontend/src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root')!);

root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// Vite HMR (Hot Module Replacement)
if (import.meta.hot) {
  import.meta.hot.accept();
}
```

**Result:**

- Edit component → Browser updates automatically
- State preserved (form input values stay)
- No full page reload

**Node hot reload (backend):**

```json
// backend/package.json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "start": "node dist/index.js"
  },
  "devDependencies": {
    "tsx": "^4.7.0"
  }
}
```

**tsx watch detects changes, restarts server in 200ms.**

***

### 6.1.5 Environment Variable Management

**Secrets stay secret. Configuration stays configurable.**

**Environment variable strategy:**

```
Development:   .env.local (git-ignored, local secrets)
Staging:       AWS Secrets Manager (environment-specific)
Production:    AWS Secrets Manager (environment-specific)
Testing:       .env.test (committed, fake data)
```

**Development setup:**

```bash
# frontend/.env.local (git-ignored)
VITE_API_URL=http://localhost:3000/api
VITE_ENV=development
VITE_USE_MOCK_API=false

# backend/.env.local (git-ignored)
NODE_ENV=development
DATABASE_URL=postgresql://dev_user:dev_password@localhost:5432/orders_db
REDIS_URL=redis://localhost:6379
JWT_SECRET=dev_secret_key_only_for_local
STRIPE_SECRET_KEY=sk_test_your_test_key
SENDGRID_API_KEY=SG.test_key
```

**Load environment variables safely:**

```typescript
// backend/src/config.ts
import dotenv from 'dotenv';
import path from 'path';

// Load .env based on NODE_ENV
const envFile = process.env.NODE_ENV === 'test' ? '.env.test' : '.env.local';
dotenv.config({ path: path.resolve(envFile) });

interface Config {
  nodeEnv: string;
  databaseUrl: string;
  redisUrl: string;
  jwtSecret: string;
  stripeSecretKey: string;
}

// Validate required environment variables
function validateEnv(): Config {
  const required = [
    'DATABASE_URL',
    'REDIS_URL',
    'JWT_SECRET',
    'STRIPE_SECRET_KEY'
  ];

  for (const key of required) {
    if (!process.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }

  return {
    nodeEnv: process.env.NODE_ENV || 'development',
    databaseUrl: process.env.DATABASE_URL!,
    redisUrl: process.env.REDIS_URL!,
    jwtSecret: process.env.JWT_SECRET!,
    stripeSecretKey: process.env.STRIPE_SECRET_KEY!
  };
}

export const config = validateEnv();
```

**Frontend environment variables:**

```typescript
// frontend/src/config.ts
const config = {
  apiUrl: import.meta.env.VITE_API_URL,
  env: import.meta.env.VITE_ENV,
  useMockApi: import.meta.env.VITE_USE_MOCK_API === 'true'
};

if (!config.apiUrl) {
  throw new Error('Missing VITE_API_URL environment variable');
}

export default config;
```

**Never commit secrets:**

```bash
# .gitignore
.env.local           # Never commit
.env.*.local         # Never commit
.env.production      # Never commit
.DS_Store
node_modules
dist
```

**Safe defaults in git:**

```bash
# .env.example (commit this, no secrets)
VITE_API_URL=http://localhost:3000/api
VITE_ENV=development
VITE_USE_MOCK_API=false

NODE_ENV=development
DATABASE_URL=postgresql://user:pass@localhost:5432/db
REDIS_URL=redis://localhost:6379
JWT_SECRET=change_me_in_development
STRIPE_SECRET_KEY=sk_test_your_key
```


***

## 6.2 CI/CD PIPELINE

### 6.2.1 Pipeline Stages Definition

**Code commit → Tests → Build → Security scan → Deploy (fully automated).**

**Pipeline stages:**

```
Trigger (git push)
    ↓
┌─────────────────────────────────────┐
│ Stage 1: LINT & FORMAT (2 min)      │
├─────────────────────────────────────┤
│ ├─ ESLint (code quality)            │
│ ├─ Prettier (formatting)            │
│ ├─ TypeScript compile check         │
│ └─ Fail if issues found             │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Stage 2: TEST (5 min)               │
├─────────────────────────────────────┤
│ ├─ Unit tests (Jest)                │
│ ├─ Integration tests                │
│ ├─ Component tests (React)          │
│ └─ Fail if coverage < 80%           │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Stage 3: BUILD (3 min)              │
├─────────────────────────────────────┤
│ ├─ Backend: npm run build           │
│ ├─ Frontend: npm run build          │
│ ├─ Docker image creation            │
│ └─ Push to ECR (AWS)                │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Stage 4: SECURITY SCAN (4 min)      │
├─────────────────────────────────────┤
│ ├─ OWASP/Snyk dependency scan       │
│ ├─ Container vulnerability scan     │
│ ├─ SAST (Sonarqube)                 │
│ └─ Fail if critical vulns found     │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Stage 5: DEPLOY STAGING (5 min)     │
├─────────────────────────────────────┤
│ ├─ Deploy to staging environment    │
│ ├─ Run smoke tests                  │
│ ├─ Manual approval to continue      │
│ └─ Fail if smoke tests fail         │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Stage 6: DEPLOY PRODUCTION (5 min)  │
├─────────────────────────────────────┤
│ ├─ Blue-green deployment            │
│ ├─ Health checks                    │
│ ├─ Monitor for 5 min                │
│ └─ Automatic rollback if errors     │
└─────────────────────────────────────┘
    ↓
SUCCESS (20 minutes total)
```

**GitHub Actions workflow:**

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    name: Lint & Format
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci

      - run: npm run lint
      - run: npm run format:check
      - run: npm run type-check

  test:
    name: Tests
    runs-on: ubuntu-latest
    needs: lint

    services:
      postgres:
        image: postgres:15-alpine
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npm run migrate:test
      - run: npm run test:ci
      - run: npm run test:coverage

      - uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
          fail_ci_if_error: true

  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: test
    if: github.event_name == 'push'

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
          aws-region: us-east-1

      - name: Login to ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: orders-api
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -f backend/Dockerfile.prod -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

  security-scan:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: build

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - run: npm install -g snyk
      - run: snyk test --fail-on=high

  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: [build, security-scan]
    environment: staging

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
          aws-region: us-east-1

      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster orders-staging \
            --service orders-api-staging \
            --force-new-deployment

      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster orders-staging \
            --services orders-api-staging

      - name: Run smoke tests
        run: |
          npx playwright test --project=smoke

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment: production
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
          aws-region: us-east-1

      - name: Blue-green deployment
        run: |
          # Get current active task definition
          CURRENT_TASK=$(aws ecs list-tasks \
            --cluster orders-prod \
            --service-name orders-api \
            --query 'taskArns[0]' --output text)

          # Deploy new task definition
          aws ecs update-service \
            --cluster orders-prod \
            --service orders-api \
            --force-new-deployment

          # Wait for new tasks to be healthy
          aws ecs wait services-stable \
            --cluster orders-prod \
            --services orders-api

          # Monitor error rate for 5 minutes
          # (done in background monitoring job)

      - name: Notify Slack
        if: success()
        run: |
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -d '{"text": "✅ Production deployment successful"}'

      - name: Automatic rollback on failure
        if: failure()
        run: |
          aws ecs update-service \
            --cluster orders-prod \
            --service orders-api \
            --task-definition orders-api:$(($TASK_VERSION - 1))
```


***

### 6.2.2 Automated Testing Integration

**Every commit tested automatically. No manual testing for basic cases.**

**Test pyramid (in time spent):**

```
         E2E Tests (10%)
           /\
          /  \
         /    \
    Integration (30%)
       /        \
      /          \
     /            \
   Unit Tests (60%)
```

**Unit tests (Jest):**

```typescript
// backend/src/__tests__/orders.service.test.ts
import { OrderService } from '../services/orders.service';
import { PrismaClient } from '@prisma/client';
import { Redis } from 'ioredis';

jest.mock('@prisma/client');
jest.mock('ioredis');

describe('OrderService', () => {
  let orderService: OrderService;
  let mockPrisma: jest.Mocked<PrismaClient>;
  let mockRedis: jest.Mocked<Redis>;

  beforeEach(() => {
    mockPrisma = new PrismaClient() as jest.Mocked<PrismaClient>;
    mockRedis = new Redis() as jest.Mocked<Redis>;
    orderService = new OrderService(mockPrisma, mockRedis);
  });

  describe('createOrder', () => {
    it('should create order with valid data', async () => {
      mockPrisma.order.create.mockResolvedValue({
        id: '123',
        customer_id: 'cust-1',
        total: 99.99,
        status: 'pending',
        created_at: new Date(),
        updated_at: new Date()
      });

      const result = await orderService.createOrder({
        customer_id: 'cust-1',
        total: 99.99
      });

      expect(result.id).toBe('123');
      expect(mockPrisma.order.create).toHaveBeenCalled();
    });

    it('should throw on insufficient inventory', async () => {
      mockPrisma.inventory.findUnique.mockResolvedValue({
        quantity: 2,
        sku: 'LAPTOP'
      });

      await expect(
        orderService.createOrder({
          items: [{ sku: 'LAPTOP', quantity: 5 }]
        })
      ).rejects.toThrow('Insufficient inventory');
    });
  });
});
```

**Integration tests (with real DB):**

```typescript
// backend/src/__tests__/orders.integration.test.ts
import { test, beforeAll, afterAll } from 'vitest';
import { PrismaClient } from '@prisma/client';
import { OrderService } from '../services/orders.service';

let prisma: PrismaClient;
let orderService: OrderService;

beforeAll(async () => {
  prisma = new PrismaClient({
    datasources: { db: { url: process.env.DATABASE_TEST_URL } }
  });
  orderService = new OrderService(prisma);
  
  // Setup: Create test user
  await prisma.user.create({
    data: {
      id: 'test-user-1',
      email: 'test@example.com',
      password_hash: 'hash'
    }
  });
});

afterAll(async () => {
  await prisma.$disconnect();
});

test('should create order and update inventory', async () => {
  // Setup
  await prisma.inventory.create({
    data: { sku: 'LAPTOP', quantity: 10 }
  });

  // Execute
  const order = await orderService.createOrder({
    customer_id: 'test-user-1',
    items: [{ sku: 'LAPTOP', quantity: 1 }]
  });

  // Verify
  const inventory = await prisma.inventory.findUnique({
    where: { sku: 'LAPTOP' }
  });

  expect(order.id).toBeDefined();
  expect(inventory?.quantity).toBe(9);
});
```

**E2E tests (Playwright):**

```typescript
// frontend/e2e/checkout.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Checkout Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Login before each test
    await page.goto('http://localhost:5173/auth/login');
    await page.fill('input[name="email"]', 'user@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button:has-text("Login")');
    await page.waitForNavigation();
  });

  test('should complete checkout successfully', async ({ page }) => {
    // Navigate to products
    await page.goto('http://localhost:5173/products');
    
    // Add item to cart
    await page.click('button:has-text("Add to Cart"):first');
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('1');

    // Go to checkout
    await page.click('button:has-text("Checkout")');
    await page.waitForURL('**/checkout');

    // Fill shipping address
    await page.fill('input[name="street"]', '123 Main St');
    await page.fill('input[name="city"]', 'San Francisco');
    await page.fill('input[name="state"]', 'CA');
    await page.fill('input[name="zip"]', '94102');

    // Fill payment
    await page.frameLocator('iframe[title="Stripe"]').locator('[placeholder="Card number"]')
      .fill('4242424242424242');
    await page.frameLocator('iframe[title="Stripe"]').locator('[placeholder="MM / YY"]')
      .fill('12 / 25');

    // Place order
    await page.click('button:has-text("Place Order")');

    // Verify confirmation
    await expect(page).toHaveURL('**/confirmation');
    await expect(page.locator('text=Order created successfully')).toBeVisible();
  });
});
```

**Run tests in CI:**

```bash
# Unit + integration tests
npm run test:ci

# E2E tests
npm run test:e2e

# Coverage report
npm run test:coverage
```


***

### 6.2.3 Code Quality Gates

**Bad code doesn't make it to main branch.**

**ESLint configuration:**

```javascript
// backend/.eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser',
  extends: ['eslint:recommended', 'plugin:@typescript-eslint/recommended'],
  rules: {
    'no-console': ['warn', { allow: ['error', 'warn'] }],
    'no-unused-vars': 'off',
    '@typescript-eslint/no-unused-vars': ['error'],
    '@typescript-eslint/explicit-function-return-types': 'warn',
    'no-var': 'error',
    'prefer-const': 'error'
  },
  overrides: [
    {
      files: ['**/*.test.ts'],
      env: { jest: true }
    }
  ]
};
```

**Prettier formatting:**

```javascript
// .prettierrc.js
module.exports = {
  semi: true,
  trailingComma: 'es5',
  singleQuote: true,
  printWidth: 100,
  tabWidth: 2,
  useTabs: false,
  arrowParens: 'always'
};
```

**Pre-commit hooks (prevent bad commits):**

```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.ts": ["eslint --fix", "prettier --write"],
    "*.tsx": ["eslint --fix", "prettier --write"]
  }
}
```

**Code quality scan (SonarQube):**

```yaml
# .github/workflows/sonarqube.yml
name: SonarQube Scan

on: [push, pull_request]

jobs:
  sonarqube:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - run: npm ci
      - run: npm run test:coverage

      - uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

**Quality gate prevents merge if:**

```
├─ Code coverage < 80%
├─ Duplication > 5%
├─ Critical/high bugs found
├─ Security hotspots unreviewed
└─ New vulnerabilities introduced
```


***

### 6.2.4 Security Scanning Setup

**Find vulnerabilities before attackers do.**

**Dependency scanning (Snyk):**

```bash
# Install Snyk CLI
npm install -g snyk

# Test dependencies
snyk test

# Monitor for new vulnerabilities
snyk monitor

# Fix known vulnerabilities
snyk fix
```

**SAST (Static Application Security Testing):**

```yaml
# .github/workflows/security.yml
name: Security Scan

on: [push, pull_request]

jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

      - name: Upload results
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: snyk.sarif

  container-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -f Dockerfile.prod -t myapp:${{ github.sha }} .

      - name: Scan with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: trivy-results.sarif
```

**Secret detection (prevent leaks):**

```bash
# Detect secrets before commit
npm install -D detect-secrets

# Scan entire codebase
detect-secrets scan

# Audit baseline
detect-secrets audit .secrets.baseline
```


***

### 6.2.5 Deployment Automation

**Push button (or automatic on merge) deployment.**

**GitOps with Flux (Kubernetes):**

```yaml
# infrastructure/flux/orders-api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders-api
  namespace: production
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: orders-api
  template:
    metadata:
      labels:
        app: orders-api
    spec:
      serviceAccountName: orders-api
      containers:
      - name: api
        image: 123456789.dkr.ecr.us-east-1.amazonaws.com/orders-api:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: orders-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: orders-secrets
              key: redis-url
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - orders-api
              topologyKey: kubernetes.io/hostname
```

**Flux automation:**

```yaml
# infrastructure/flux/orders-api-imagerepository.yaml
apiVersion: image.fluxcd.io/v1beta1
kind: ImageRepository
metadata:
  name: orders-api
spec:
  image: 123456789.dkr.ecr.us-east-1.amazonaws.com/orders-api
  interval: 1m

---
apiVersion: image.fluxcd.io/v1beta1
kind: ImagePolicy
metadata:
  name: orders-api
spec:
  imageRepositoryRef:
    name: orders-api
  policy:
    semver:
      range: '>=1.0.0 <2.0.0'

---
apiVersion: image.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: orders-api
spec:
  sourceRef:
    kind: GitRepository
    name: flux-system
  interval: 1m
  autoDiscoverBranches: true
  update:
    strategy: Setters
    path: ./infrastructure/flux
  commit:
    author:
      name: fluxcdbot
      email: fluxcdbot@example.com
    messageTemplate: 'Automated image update'
  push:
    branch: main
```

**Blue-green deployment (zero downtime):**

```bash
#!/bin/bash
# scripts/deploy.sh

CURRENT_ENV=$(aws ecs describe-services \
  --cluster orders-prod \
  --services orders-api \
  --query 'services[0].taskDefinition' \
  --output text | grep -oP 'orders-api:\K\d+' || echo "0")

NEW_ENV=$((CURRENT_ENV + 1))

# Register new task definition
aws ecs register-task-definition \
  --cli-input-json file://task-definition-v${NEW_ENV}.json

# Update service (creates new tasks, keeps old ones running)
aws ecs update-service \
  --cluster orders-prod \
  --service orders-api \
  --task-definition orders-api:${NEW_ENV}

# Wait for new tasks to be healthy
aws ecs wait services-stable \
  --cluster orders-prod \
  --services orders-api

# Monitor for 5 minutes
sleep 300

# Check error metrics
ERROR_RATE=$(aws cloudwatch get-metric-statistics \
  --namespace AWS/ECS \
  --metric-name Errors \
  --dimensions Name=ServiceName,Value=orders-api \
  --start-time $(date -u -d '5 minutes ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Sum \
  --query 'Datapoints[0].Sum' \
  --output text)

if (( $(echo "$ERROR_RATE > 10" | bc -l) )); then
  # Rollback
  aws ecs update-service \
    --cluster orders-prod \
    --service orders-api \
    --task-definition orders-api:${CURRENT_ENV}
  exit 1
fi

echo "✅ Deployment successful"
```


***

## 6.3 CLOUD INFRASTRUCTURE (IaC)

### 6.3.1 Infrastructure as Code (Terraform)

**All infrastructure defined in code. Version controlled. Reproducible.**

**Directory structure:**

```
infrastructure/
├─ terraform/
│  ├─ main.tf              # Main configuration
│  ├─ variables.tf         # Input variables
│  ├─ outputs.tf           # Output values
│  ├─ vpc.tf               # Network setup
│  ├─ rds.tf               # Database
│  ├─ elasticache.tf       # Redis
│  ├─ ecs.tf               # Container orchestration
│  ├─ alb.tf               # Load balancer
│  ├─ iam.tf               # Permissions
│  ├─ cloudwatch.tf        # Monitoring
│  ├─ terraform.tfvars     # Development values
│  ├─ prod.tfvars          # Production values
│  └─ .terraform.lock.hcl  # Locked versions
└─ kubernetes/             # K8s manifests
```

**Main Terraform configuration:**

```hcl
# infrastructure/terraform/main.tf
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket         = "orders-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Project     = "orders"
      Environment = var.environment
      ManagedBy   = "Terraform"
      CreatedAt   = timestamp()
    }
  }
}

# VPC + Networking
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "orders-${var.environment}"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  enable_vpn_gateway = false
  enable_dns_hostnames = true

  tags = {
    Name = "orders-vpc-${var.environment}"
  }
}

# RDS PostgreSQL
module "rds" {
  source = "terraform-aws-modules/rds/aws"
  version = "6.0.0"

  identifier = "orders-db-${var.environment}"

  engine               = "postgres"
  engine_version       = "15.3"
  family               = "postgres15"
  major_engine_version = "15"
  instance_class       = var.db_instance_class

  allocated_storage     = var.db_allocated_storage
  max_allocated_storage = var.db_max_allocated_storage

  db_name  = "orders_db"
  username = "postgres"
  password = random_password.db_password.result

  multi_az            = var.environment == "prod" ? true : false
  publicly_accessible = false

  backup_retention_period = var.environment == "prod" ? 30 : 7
  backup_window           = "03:00-04:00"
  maintenance_window      = "mon:04:00-mon:05:00"

  skip_final_snapshot = var.environment == "dev" ? true : false

  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = module.vpc.database_subnet_group_name

  enabled_cloudwatch_logs_exports = ["postgresql"]

  tags = {
    Name = "orders-db-${var.environment}"
  }
}

# Random password for RDS
resource "random_password" "db_password" {
  length  = 32
  special = true
}

# Store password in Secrets Manager
resource "aws_secretsmanager_secret" "db_password" {
  name = "orders/db/password-${var.environment}"
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id      = aws_secretsmanager_secret.db_password.id
  secret_string  = random_password.db_password.result
}

# ElastiCache Redis
module "redis" {
  source = "terraform-aws-modules/elasticache/aws"
  version = "1.0.0"

  name = "orders-cache-${var.environment}"

  engine               = "redis"
  engine_version       = "7.1"
  node_type            = var.redis_node_type
  num_cache_nodes      = var.environment == "prod" ? 3 : 1
  parameter_group_name = "default.redis7"
  port                 = 6379

  automatic_failover_enabled = var.environment == "prod" ? true : false
  multi_az_enabled          = var.environment == "prod" ? true : false

  at_rest_encryption_enabled = true
  transit_encryption_enabled = true

  security_group_ids = [aws_security_group.redis.id]
  subnet_group_name  = aws_elasticache_subnet_group.redis.name

  log_delivery_configuration = {
    slow_log = {
      cloudwatch_log_group      = aws_cloudwatch_log_group.redis_slow_log.name
      cloudwatch_log_group_arn  = "${aws_cloudwatch_log_group.redis_slow_log.arn}:*"
      enabled                   = true
    }
    engine_log = {
      cloudwatch_log_group      = aws_cloudwatch_log_group.redis_engine_log.name
      cloudwatch_log_group_arn  = "${aws_cloudwatch_log_group.redis_engine_log.arn}:*"
      enabled                   = true
    }
  }

  tags = {
    Name = "orders-cache-${var.environment}"
  }
}

# Output values (for reference)
output "rds_endpoint" {
  value       = module.rds.db_instance_endpoint
  description = "RDS database endpoint"
}

output "redis_endpoint" {
  value       = module.redis.primary_endpoint_address
  description = "Redis primary endpoint"
}

output "vpc_id" {
  value       = module.vpc.vpc_id
  description = "VPC ID"
}
```

**Apply Terraform:**

```bash
# Plan changes (dry run)
terraform plan -var-file=dev.tfvars -out=dev.tfplan

# Review plan
cat dev.tfplan

# Apply changes
terraform apply dev.tfplan

# Output values
terraform output rds_endpoint
terraform output redis_endpoint

# Destroy (careful!)
terraform destroy -var-file=dev.tfvars
```


***

### 6.3.2 Network Configuration

**Secure network isolation. Public/private subnets. Security groups.**

**Network diagram:**

```
┌─────────────────────────────────────────┐
│          VPC 10.0.0.0/16                 │
├─────────────────────────────────────────┤
│                                         │
│  PUBLIC SUBNET (10.0.101.0/24)          │
│  ├─ NAT Gateway                         │
│  ├─ Application Load Balancer (port 443)│
│  └─ Internet Gateway                    │
│                                         │
│  PRIVATE SUBNET (10.0.1.0/24)           │
│  ├─ ECS Tasks (API servers)             │
│  └─ Security Group: Allow ALB only      │
│                                         │
│  PRIVATE SUBNET (10.0.2.0/24)           │
│  ├─ RDS PostgreSQL                      │
│  └─ Security Group: Allow ECS only      │
│                                         │
│  PRIVATE SUBNET (10.0.3.0/24)           │
│  ├─ ElastiCache Redis                   │
│  └─ Security Group: Allow ECS only      │
│                                         │
└─────────────────────────────────────────┘

Traffic flow:
Internet → ALB (public) → ECS Tasks (private) → RDS/Redis (private)
```

**Security groups (Terraform):**

```hcl
# ALB security group (allows internet traffic)
resource "aws_security_group" "alb" {
  name        = "orders-alb-${var.environment}"
  description = "Security group for ALB"
  vpc_id      = module.vpc.vpc_id

  # Allow HTTPS from internet
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allow HTTP for redirect to HTTPS
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allow outbound traffic
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "orders-alb-sg-${var.environment}"
  }
}

# ECS security group (allows ALB only)
resource "aws_security_group" "ecs" {
  name        = "orders-ecs-${var.environment}"
  description = "Security group for ECS tasks"
  vpc_id      = module.vpc.vpc_id

  # Allow traffic from ALB
  ingress {
    from_port       = 3000
    to_port         = 3000
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  # Allow outbound to everything (needed for API calls, updates)
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "orders-ecs-sg-${var.environment}"
  }
}

# RDS security group (allows ECS only)
resource "aws_security_group" "rds" {
  name        = "orders-rds-${var.environment}"
  description = "Security group for RDS"
  vpc_id      = module.vpc.vpc_id

  # Allow PostgreSQL from ECS
  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.ecs.id]
  }

  # No outbound (database doesn't initiate connections)

  tags = {
    Name = "orders-rds-sg-${var.environment}"
  }
}

# Redis security group (allows ECS only)
resource "aws_security_group" "redis" {
  name        = "orders-redis-${var.environment}"
  description = "Security group for Redis"
  vpc_id      = module.vpc.vpc_id

  # Allow Redis from ECS
  ingress {
    from_port       = 6379
    to_port         = 6379
    protocol        = "tcp"
    security_groups = [aws_security_group.ecs.id]
  }

  # No outbound

  tags = {
    Name = "orders-redis-sg-${var.environment}"
  }
}
```


***

### 6.3.3 Load Balancer Configuration

**Distribute traffic across multiple app instances. Health checks. SSL termination.**

```hcl
# Application Load Balancer
resource "aws_lb" "main" {
  name               = "orders-alb-${var.environment}"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = module.vpc.public_subnets

  enable_deletion_protection = var.environment == "prod"
  enable_http2              = true
  enable_cross_zone_load_balancing = true

  tags = {
    Name = "orders-alb-${var.environment}"
  }
}

# Target group (where to send traffic)
resource "aws_lb_target_group" "api" {
  name        = "orders-api-tg-${var.environment}"
  port        = 3000
  protocol    = "HTTP"
  vpc_id      = module.vpc.vpc_id
  target_type = "ip"

  health_check {
    healthy_threshold   = 2
    unhealthy_threshold = 2
    timeout             = 3
    interval            = 30
    path                = "/health"
    matcher             = "200"
  }

  tags = {
    Name = "orders-api-tg-${var.environment}"
  }
}

# HTTPS listener (with SSL certificate)
resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.main.arn
  port              = "443"
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS-1-2-2017-01"
  certificate_arn   = aws_acm_certificate.main.arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.api.arn
  }
}

# HTTP redirect to HTTPS
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type = "redirect"

    redirect {
      port        = "443"
      protocol    = "HTTPS"
      status_code = "HTTP_301"
    }
  }
}

# SSL certificate (auto-renewal)
resource "aws_acm_certificate" "main" {
  domain_name       = "api.company.com"
  validation_method = "DNS"

  lifecycle {
    create_before_destroy = true
  }

  tags = {
    Name = "orders-api-cert-${var.environment}"
  }
}

# Output for CloudFront
output "alb_dns_name" {
  value = aws_lb.main.dns_name
}
```


***

### 6.3.4 Auto-Scaling Policies

**Automatically add/remove instances based on demand.**

```hcl
# ECS Auto Scaling
resource "aws_ecs_service" "api" {
  name            = "orders-api"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.api.arn
  desired_count   = var.ecs_desired_count

  launch_type = "FARGATE"

  load_balancer {
    target_group_arn = aws_lb_target_group.api.arn
    container_name   = "api"
    container_port   = 3000
  }

  network_configuration {
    subnets          = module.vpc.private_subnets
    security_groups  = [aws_security_group.ecs.id]
    assign_public_ip = false
  }

  depends_on = [aws_lb_listener.https]
}

# Auto Scaling Target (tracks the service)
resource "aws_autoscaling_target" "ecs_target" {
  max_capacity       = var.environment == "prod" ? 20 : 5
  min_capacity       = var.environment == "prod" ? 3 : 1
  resource_id        = "service/${aws_ecs_cluster.main.name}/${aws_ecs_service.api.name}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
}

# Scale up on high CPU (> 70%)
resource "aws_autoscaling_policy" "ecs_policy_up" {
  name               = "orders-scale-up-${var.environment}"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_autoscaling_target.ecs_target.resource_id
  scalable_dimension = aws_autoscaling_target.ecs_target.scalable_dimension
  service_namespace  = aws_autoscaling_target.ecs_target.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }

    target_value = 70.0
    scale_out_cooldown = 60
    scale_in_cooldown  = 300
  }
}

# Scale up on high memory (> 80%)
resource "aws_autoscaling_policy" "ecs_policy_memory" {
  name               = "orders-scale-memory-${var.environment}"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_autoscaling_target.ecs_target.resource_id
  scalable_dimension = aws_autoscaling_target.ecs_target.scalable_dimension
  service_namespace  = aws_autoscaling_target.ecs_target.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageMemoryUtilization"
    }

    target_value = 80.0
  }
}

# Target group attribute for deregistration delay
resource "aws_lb_target_group" "api" {
  # ... existing config ...

  deregistration_delay = 30

  stickiness {
    type            = "lb_cookie"
    cookie_duration = 86400
    enabled         = false  # Disable for stateless API
  }
}
```

**Scaling behavior:**

```
Load: 20 req/sec
├─ 1 instance @ 85% CPU
├─ Trigger: CPU > 70%
├─ Action: Launch 1 new instance
├─ Wait: 60 seconds (scale_out_cooldown)
└─ Result: 2 instances @ 42% CPU each

Load: 5 req/sec
├─ 2 instances @ 30% CPU each
├─ Trigger: CPU < 70% for 5 minutes
├─ Action: Terminate 1 instance
├─ Wait: 300 seconds (scale_in_cooldown)
└─ Result: 1 instance @ 60% CPU
```


***

## 6.4 MONITORING \& LOGGING

### 6.4.1 Log Aggregation Setup

**All logs centralized. Searchable. Queryable. Alertable.**

**CloudWatch Logs (AWS native):**

```hcl
# CloudWatch Log Group for backend
resource "aws_cloudwatch_log_group" "api" {
  name              = "/orders/api/${var.environment}"
  retention_in_days = var.environment == "prod" ? 30 : 7

  tags = {
    Name = "orders-api-logs-${var.environment}"
  }
}

# CloudWatch Log Group for database
resource "aws_cloudwatch_log_group" "rds" {
  name              = "/orders/rds/${var.environment}"
  retention_in_days = var.environment == "prod" ? 30 : 7

  tags = {
    Name = "orders-rds-logs-${var.environment}"
  }
}
```

**Backend logging configuration:**

```typescript
// backend/src/logger.ts
import winston from 'winston';

// JSON logs for CloudWatch parsing```

