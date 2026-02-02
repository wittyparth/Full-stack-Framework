# PHASE 7: BACKEND IMPLEMENTATION

**The difference between a junior and senior backend engineer: Juniors build features, seniors build systems that survive production.**

This phase transforms architecture into working code. No padding. Every line earns its place.

---

## 7.1 PROJECT SETUP & ARCHITECTURE

### 7.1.1 Project Structure (Feature-First)

**The only correct structure: Feature-first, not layer-first.**

❌ **WRONG - Layer-first (creates cross-cutting spaghetti):**
```
src/
├── controllers/
│  ├── userController.ts
│  ├── orderController.ts
│  └── productController.ts
├── services/
│  ├── userService.ts
│  ├── orderService.ts
│  └── productService.ts
├── models/
│  ├── User.ts
│  ├── Order.ts
│  └── Product.ts
```
Problem: Adding a feature requires touching 3+ directories. Import hell. Circular dependencies.

✅ **CORRECT - Feature-first (isolates changes):**

```
src/
├── features/
│  ├── auth/
│  │  ├── auth.controller.ts
│  │  ├── auth.service.ts
│  │  ├── auth.repository.ts
│  │  ├── auth.types.ts
│  │  ├── auth.middleware.ts
│  │  ├── auth.routes.ts
│  │  └── __tests__/
│  │
│  ├── users/
│  │  ├── users.controller.ts
│  │  ├── users.service.ts
│  │  ├── users.repository.ts
│  │  ├── users.types.ts
│  │  ├── users.routes.ts
│  │  └── __tests__/
│  │
│  ├── orders/
│  │  ├── orders.controller.ts
│  │  ├── orders.service.ts
│  │  ├── orders.repository.ts
│  │  ├── orders.types.ts
│  │  ├── orders.routes.ts
│  │  └── __tests__/
│  │
│  └── products/
│     ├── products.controller.ts
│     ├── products.service.ts
│     ├── products.repository.ts
│     ├── products.types.ts
│     ├── products.routes.ts
│     └── __tests__/
│
├── shared/
│  ├── config/
│  │  ├── app.config.ts
│  │  ├── database.config.ts
│  │  └── env.validation.ts
│  ├── middleware/
│  │  ├── errorHandler.ts
│  │  ├── requestLogger.ts
│  │  ├── rateLimiter.ts
│  │  └── cors.ts
│  ├── types/
│  │  ├── express.d.ts
│  │  ├── errors.ts
│  │  └── http.ts
│  ├── utils/
│  │  ├── crypto.ts
│  │  ├── jwt.ts
│  │  ├── response.ts
│  │  └── validation.ts
│  └── database/
│     ├── connection.ts
│     └── migrations/
│
├── app.ts
└── server.ts
```

**Benefit of feature-first:**
- Add new feature: Create one directory, touch no existing code
- Delete feature: Delete one directory, no cascading changes
- Team scaling: Each team owns one feature, no merge conflicts
- Testing: Feature test suite collocated with feature

### 7.1.2 Layered Architecture Within Each Feature

**Inside each feature: Repository → Service → Controller**

```
// Feature structure (auth example)

// 1. REPOSITORY (Data access only)
// auth.repository.ts
class AuthRepository {
  async findUserByEmail(email: string): Promise<User | null> {
    const user = await db.query('SELECT * FROM users WHERE email = ?', [email]);
    return user[0] || null;
  }
  
  async createSession(userId: string, token: string): Promise<void> {
    await db.query('INSERT INTO sessions (userId, token) VALUES (?, ?)', [userId, token]);
  }
}

// 2. SERVICE (Business logic only)
// auth.service.ts
class AuthService {
  constructor(private repo: AuthRepository) {}
  
  async login(email: string, password: string): Promise<LoginResponse> {
    // Business logic, not database logic
    const user = await this.repo.findUserByEmail(email);
    if (!user) throw new NotFoundError('User not found');
    
    const passwordMatch = await crypto.compare(password, user.passwordHash);
    if (!passwordMatch) throw new UnauthorizedError('Invalid password');
    
    const token = jwt.sign({ userId: user.id }, config.jwt.secret);
    await this.repo.createSession(user.id, token);
    
    return { token, user: { id: user.id, email: user.email } };
  }
}

// 3. CONTROLLER (HTTP handling only)
// auth.controller.ts
class AuthController {
  constructor(private service: AuthService) {}
  
  async login(req: Request, res: Response): Promise<void> {
    const { email, password } = req.body;
    
    const result = await this.service.login(email, password);
    
    res.status(200).json({
      success: true,
      data: result
    });
  }
}

// 4. ROUTES (Wiring only)
// auth.routes.ts
router.post('/login', validate(loginSchema), asyncHandler(authController.login));

// 5. MIDDLEWARE (Cross-cutting concerns)
// auth.middleware.ts
const authMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });
  
  try {
    const decoded = jwt.verify(token, config.jwt.secret);
    req.user = decoded;
    next();
  } catch (err) {
    res.status(401).json({ error: 'Invalid token' });
  }
};
```

**Key principle: Each layer has ONE responsibility**

Repository:
- SQL queries only
- No business logic
- Testable in isolation with database mocking

Service:
- Business logic only
- Calls repository for data
- No HTTP concerns
- Testable with repository mocking

Controller:
- HTTP request/response only
- Calls service for logic
- Formats response
- Error handling at HTTP level

---

### 7.1.3 Dependency Injection Pattern

**Inversion of Control prevents tight coupling. Services don't create dependencies—they receive them.**

```typescript
// ❌ WRONG - Tight coupling
class AuthService {
  private repo = new AuthRepository(); // Hard-coded dependency
  private emailService = new EmailService(); // Can't test with mock
  
  async login(email: string, password: string) {
    // Can't control database in tests
    // Can't mock email service
  }
}

// ✅ CORRECT - Dependency injection
class AuthService {
  constructor(
    private repo: AuthRepository,
    private emailService: EmailService
  ) {}
  
  async login(email: string, password: string) {
    // Dependencies injected, mockable in tests
  }
}

// In Express setup:
const authRepository = new AuthRepository(dbConnection);
const emailService = new EmailService(config.email);
const authService = new AuthService(authRepository, emailService);
const authController = new AuthController(authService);

// In tests:
const mockRepo = {
  findUserByEmail: jest.fn(),
  createSession: jest.fn()
};
const mockEmail = { send: jest.fn() };
const service = new AuthService(mockRepo, mockEmail);

// Now you can test business logic without database or email
```

### 7.1.4 Configuration Management

**Centralized configuration, validated at startup. Fail fast.**

```typescript
// shared/config/env.validation.ts
import { z } from 'zod';

const envSchema = z.object({
  // Server
  NODE_ENV: z.enum(['development', 'staging', 'production']).default('development'),
  PORT: z.coerce.number().default(3000),
  
  // Database
  DATABASE_URL: z.string().url(),
  DB_POOL_MIN: z.coerce.number().default(2),
  DB_POOL_MAX: z.coerce.number().default(10),
  
  // JWT
  JWT_SECRET: z.string().min(32, 'JWT_SECRET must be at least 32 characters'),
  JWT_EXPIRES_IN: z.string().default('7d'),
  
  // Redis
  REDIS_URL: z.string().url(),
  
  // CORS
  CORS_ORIGIN: z.string().default('http://localhost:3000'),
  
  // External services
  SENDGRID_API_KEY: z.string(),
  STRIPE_SECRET_KEY: z.string(),
  
  // Logging
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info')
});

export function validateEnv() {
  const result = envSchema.safeParse(process.env);
  
  if (!result.success) {
    const errors = result.error.flatten().fieldErrors;
    const errorMessage = Object.entries(errors)
      .map(([key, value]) => `${key}: ${value?.join(', ')}`)
      .join('\n');
    
    throw new Error(`Environment validation failed:\n${errorMessage}`);
  }
  
  return result.data;
}

// shared/config/app.config.ts
export interface AppConfig {
  env: string;
  port: number;
  database: {
    url: string;
    poolMin: number;
    poolMax: number;
  };
  jwt: {
    secret: string;
    expiresIn: string;
  };
  // ... rest of config
}

export function getConfig(): AppConfig {
  const env = validateEnv();
  
  return {
    env: env.NODE_ENV,
    port: env.PORT,
    database: {
      url: env.DATABASE_URL,
      poolMin: env.DB_POOL_MIN,
      poolMax: env.DB_POOL_MAX
    },
    jwt: {
      secret: env.JWT_SECRET,
      expiresIn: env.JWT_EXPIRES_IN
    }
    // ... rest of config
  };
}

// server.ts
const config = getConfig(); // Validates and throws if invalid
const app = express();
// ... rest of app setup
```

**Benefits:**
- Validates environment at startup (fail fast, not 2 hours into processing)
- Type-safe configuration (TS knows structure)
- Clear required vs optional
- Single source of truth

---

### 7.1.5 Logging Framework

**Structured logging, not console.log. Searchable, traceable, production-ready.**

```typescript
// shared/config/logger.ts
import winston from 'winston';

interface LogContext {
  userId?: string;
  requestId?: string;
  feature?: string;
  [key: string]: any;
}

class Logger {
  private logger: winston.Logger;
  private context: LogContext = {};
  
  constructor(private serviceName: string) {
    this.logger = winston.createLogger({
      defaultMeta: { service: serviceName },
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.errors({ stack: true }),
        winston.format.json()
      ),
      transports: [
        new winston.transports.Console({
          format: winston.format.combine(
            winston.format.colorize(),
            winston.format.simple()
          )
        }),
        new winston.transports.File({
          filename: 'logs/error.log',
          level: 'error'
        }),
        new winston.transports.File({
          filename: 'logs/combined.log'
        })
      ]
    });
  }
  
  setContext(context: LogContext): void {
    this.context = { ...this.context, ...context };
  }
  
  info(message: string, meta?: any): void {
    this.logger.info(message, { ...meta, ...this.context });
  }
  
  error(message: string, error?: Error | any, meta?: any): void {
    this.logger.error(message, {
      error: error?.message,
      stack: error?.stack,
      ...meta,
      ...this.context
    });
  }
  
  warn(message: string, meta?: any): void {
    this.logger.warn(message, { ...meta, ...this.context });
  }
  
  debug(message: string, meta?: any): void {
    this.logger.debug(message, { ...meta, ...this.context });
  }
}

export const createLogger = (serviceName: string) => new Logger(serviceName);

// Usage in services:
class UserService {
  private logger = createLogger('UserService');
  
  async createUser(email: string): Promise<User> {
    this.logger.setContext({ email, feature: 'user-creation' });
    
    this.logger.info('Creating user', { email });
    
    try {
      const user = await this.repo.create(email);
      this.logger.info('User created successfully', { userId: user.id });
      return user;
    } catch (error) {
      this.logger.error('Failed to create user', error, { email });
      throw error;
    }
  }
}

// Middleware to attach request ID
const requestIdMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const requestId = req.headers['x-request-id'] || crypto.randomUUID();
  req.requestId = requestId;
  
  const logger = createLogger('HTTP');
  logger.setContext({ requestId, method: req.method, path: req.path });
  logger.info(`${req.method} ${req.path}`);
  
  next();
};
```

**Benefits of structured logging:**
- Searchable in ELK/Datadog (grep through logs by userId, requestId, feature)
- Traces request through system (requestId correlation)
- Stack traces preserved (error debugging)
- Contextual information (not just "error occurred")

---

### 7.1.6 Error Handling Framework

**Typed errors, not generic strings. Consistent error responses.**

```typescript
// shared/types/errors.ts

// Base error class with HTTP status
abstract class AppError extends Error {
  abstract statusCode: number;
  abstract isOperational: boolean;
}

export class ValidationError extends AppError {
  statusCode = 400;
  isOperational = true;
  
  constructor(
    public message: string,
    public fields?: Record<string, string[]>
  ) {
    super(message);
  }
}

export class UnauthorizedError extends AppError {
  statusCode = 401;
  isOperational = true;
  
  constructor(message: string = 'Unauthorized') {
    super(message);
  }
}

export class ForbiddenError extends AppError {
  statusCode = 403;
  isOperational = true;
  
  constructor(message: string = 'Forbidden') {
    super(message);
  }
}

export class NotFoundError extends AppError {
  statusCode = 404;
  isOperational = true;
  
  constructor(public resource: string) {
    super(`${resource} not found`);
  }
}

export class ConflictError extends AppError {
  statusCode = 409;
  isOperational = true;
  
  constructor(public message: string) {
    super(message);
  }
}

// Generic server error
export class InternalServerError extends AppError {
  statusCode = 500;
  isOperational = false;
  
  constructor(message: string = 'Internal server error') {
    super(message);
  }
}

// shared/middleware/errorHandler.ts
const errorHandler = (
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  const logger = createLogger('ErrorHandler');
  logger.setContext({ requestId: req.requestId });
  
  // Operational errors (expected, user-facing)
  if (err instanceof AppError && err.isOperational) {
    logger.warn(`Operational error: ${err.message}`);
    
    const response: any = {
      success: false,
      error: {
        code: err.constructor.name,
        message: err.message
      }
    };
    
    if (err instanceof ValidationError && err.fields) {
      response.error.fields = err.fields;
    }
    
    res.status(err.statusCode).json(response);
    return;
  }
  
  // Programming errors (unexpected, must log)
  logger.error('Unhandled error', err);
  
  res.status(500).json({
    success: false,
    error: {
      code: 'INTERNAL_SERVER_ERROR',
      message: 'Internal server error'
    }
  });
};

// Async error wrapper (catches promise rejections)
export const asyncHandler = (fn: RequestHandler): RequestHandler => {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

// Usage in controller:
class UserController {
  async createUser(req: Request, res: Response): Promise<void> {
    // Validation error
    if (!req.body.email) {
      throw new ValidationError('Email is required', {
        email: ['Email is required']
      });
    }
    
    // Not found error
    const existingUser = await this.userService.findByEmail(req.body.email);
    if (existingUser) {
      throw new ConflictError('Email already registered');
    }
    
    // Success
    const user = await this.userService.createUser(req.body);
    res.status(201).json({ success: true, data: user });
  }
}

// In routes:
router.post('/users', asyncHandler(userController.createUser));
```

**Benefits:**
- Type-safe error handling (TS knows error structure)
- Consistent error responses (clients know what to expect)
- HTTP status automatic (no manual mapping)
- Operational vs programming errors (different handling)
- Stack traces in development (hidden in production)

---

## 7.2 DATABASE IMPLEMENTATION

### 7.2.1 Connection Pooling & Migrations

**Database connection pooling: Connection reuse, not creation per request.**

```typescript
// shared/database/connection.ts
import { Pool, Client } from 'pg';
import { config } from '../config';

class DatabaseConnection {
  private pool: Pool;
  
  constructor() {
    // Connection pooling: reuse connections, not create new
    this.pool = new Pool({
      connectionString: config.database.url,
      min: config.database.poolMin,  // Keep alive
      max: config.database.poolMax,  // Max concurrent connections
      idleTimeoutMillis: 30000,      // Close idle connections
      connectionTimeoutMillis: 2000  // Timeout on new connection
    });
    
    this.pool.on('error', (error) => {
      logger.error('Unexpected error on idle client', error);
    });
  }
  
  async query<T = any>(sql: string, params?: any[]): Promise<T[]> {
    const start = Date.now();
    try {
      const result = await this.pool.query(sql, params);
      const duration = Date.now() - start;
      
      // Log slow queries
      if (duration > 1000) {
        logger.warn('Slow query', { sql, duration });
      }
      
      return result.rows;
    } catch (error) {
      logger.error('Query error', error, { sql });
      throw error;
    }
  }
  
  async queryOne<T = any>(sql: string, params?: any[]): Promise<T | null> {
    const results = await this.query<T>(sql, params);
    return results[0] || null;
  }
  
  async transaction<T>(
    callback: (client: Client) => Promise<T>
  ): Promise<T> {
    const client = await this.pool.connect();
    
    try {
      await client.query('BEGIN');
      const result = await callback(client);
      await client.query('COMMIT');
      return result;
    } catch (error) {
      await client.query('ROLLBACK');
      throw error;
    } finally {
      client.release();
    }
  }
  
  async close(): Promise<void> {
    await this.pool.end();
  }
}

export const dbConnection = new DatabaseConnection();

// Migration execution (Flyway, Liquibase, or native)
// migrations/001_initial_schema.sql
/*
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);

CREATE TABLE sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  token VARCHAR(1000) NOT NULL,
  expires_at TIMESTAMP NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_sessions_user_id ON sessions(user_id);
CREATE INDEX idx_sessions_expires_at ON sessions(expires_at);
*/

// Run migrations on startup
async function runMigrations() {
  const migrationPath = path.join(__dirname, '../migrations');
  const files = fs.readdirSync(migrationPath).sort();
  
  for (const file of files) {
    const sql = fs.readFileSync(path.join(migrationPath, file), 'utf-8');
    await dbConnection.query(sql);
    logger.info(`Executed migration: ${file}`);
  }
}

// In server.ts
(async () => {
  await runMigrations();
  // Start server
})();
```

**Connection pooling benefits:**
- Reuse connections (expensive to create)
- Limit concurrent connections (prevent exhaustion)
- Timeout idle connections (cleanup)
- Monitoring (pool size, idle count)

### 7.2.2 Repository Pattern

**Single responsibility: Repository = data access only. No business logic.**

```typescript
// features/users/users.repository.ts
import { dbConnection } from '../../shared/database/connection';

interface UserRow {
  id: string;
  email: string;
  password_hash: string;
  created_at: Date;
  updated_at: Date;
}

export class UserRepository {
  async findById(id: string): Promise<UserRow | null> {
    return dbConnection.queryOne<UserRow>(
      'SELECT * FROM users WHERE id = $1',
      [id]
    );
  }
  
  async findByEmail(email: string): Promise<UserRow | null> {
    return dbConnection.queryOne<UserRow>(
      'SELECT * FROM users WHERE email = $1',
      [email]
    );
  }
  
  async create(email: string, passwordHash: string): Promise<UserRow> {
    const result = await dbConnection.query<UserRow>(
      `INSERT INTO users (email, password_hash) 
       VALUES ($1, $2) 
       RETURNING *`,
      [email, passwordHash]
    );
    return result[0];
  }
  
  async update(id: string, data: Partial<UserRow>): Promise<UserRow | null> {
    const fields = Object.keys(data)
      .map((key, index) => `${key} = $${index + 2}`)
      .join(', ');
    
    const values = [id, ...Object.values(data)];
    
    return dbConnection.queryOne<UserRow>(
      `UPDATE users SET ${fields}, updated_at = NOW() 
       WHERE id = $1 
       RETURNING *`,
      values
    );
  }
  
  async delete(id: string): Promise<void> {
    await dbConnection.query('DELETE FROM users WHERE id = $1', [id]);
  }
  
  async findMany(limit: number, offset: number): Promise<UserRow[]> {
    return dbConnection.query<UserRow>(
      'SELECT * FROM users ORDER BY created_at DESC LIMIT $1 OFFSET $2',
      [limit, offset]
    );
  }
}

// Usage in service:
class UserService {
  constructor(private repo: UserRepository) {}
  
  async createUser(email: string, password: string): Promise<User> {
    // Business logic
    if (!this.isValidEmail(email)) {
      throw new ValidationError('Invalid email');
    }
    
    const existing = await this.repo.findByEmail(email);
    if (existing) {
      throw new ConflictError('Email already registered');
    }
    
    const passwordHash = await crypto.hash(password);
    
    // Data access
    const row = await this.repo.create(email, passwordHash);
    
    // Transform to domain model
    return this.toUser(row);
  }
  
  private isValidEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
  
  private toUser(row: UserRow): User {
    return {
      id: row.id,
      email: row.email,
      createdAt: row.created_at
    };
  }
}
```

---

## 7.3 CORE SERVICES IMPLEMENTATION

### 7.3.1 Authentication Service (JWT-based)

**Stateless authentication: Token contains all needed claims. Validate signature, not lookup in database.**

```typescript
// features/auth/auth.service.ts
import jwt from 'jsonwebtoken';
import * as crypto from 'crypto';

interface TokenPayload {
  userId: string;
  email: string;
  iat: number;
  exp: number;
}

interface LoginResponse {
  token: string;
  refreshToken: string;
  user: { id: string; email: string };
}

export class AuthService {
  constructor(
    private userRepo: UserRepository,
    private sessionRepo: SessionRepository,
    private logger: Logger
  ) {}
  
  async register(email: string, password: string): Promise<LoginResponse> {
    this.logger.setContext({ feature: 'auth-register', email });
    
    // Validation
    if (!email || !password) {
      throw new ValidationError('Email and password required');
    }
    
    if (password.length < 8) {
      throw new ValidationError('Password must be at least 8 characters');
    }
    
    // Check if user exists
    const existing = await this.userRepo.findByEmail(email);
    if (existing) {
      throw new ConflictError('Email already registered');
    }
    
    // Hash password (bcrypt, not MD5 or SHA)
    const passwordHash = await crypto.bcrypt.hash(password, 12);
    
    // Create user
    const user = await this.userRepo.create(email, passwordHash);
    
    this.logger.info('User registered', { userId: user.id });
    
    // Generate tokens
    return this.createTokens(user);
  }
  
  async login(email: string, password: string): Promise<LoginResponse> {
    this.logger.setContext({ feature: 'auth-login', email });
    
    // Find user
    const user = await this.userRepo.findByEmail(email);
    if (!user) {
      throw new UnauthorizedError('Invalid email or password');
    }
    
    // Verify password
    const passwordMatch = await crypto.bcrypt.compare(
      password,
      user.password_hash
    );
    if (!passwordMatch) {
      throw new UnauthorizedError('Invalid email or password');
    }
    
    this.logger.info('User logged in', { userId: user.id });
    
    return this.createTokens(user);
  }
  
  async refreshToken(refreshToken: string): Promise<{ token: string }> {
    try {
      const decoded = jwt.verify(
        refreshToken,
        config.jwt.refreshSecret
      ) as TokenPayload;
      
      // Check session exists
      const session = await this.sessionRepo.findByToken(refreshToken);
      if (!session) {
        throw new UnauthorizedError('Invalid refresh token');
      }
      
      // Get user
      const user = await this.userRepo.findById(decoded.userId);
      if (!user) {
        throw new NotFoundError('User');
      }
      
      // Generate new access token
      const token = this.generateAccessToken(user);
      
      return { token };
    } catch (error) {
      throw new UnauthorizedError('Invalid refresh token');
    }
  }
  
  async validateToken(token: string): Promise<TokenPayload> {
    try {
      return jwt.verify(token, config.jwt.secret) as TokenPayload;
    } catch (error) {
      throw new UnauthorizedError('Invalid token');
    }
  }
  
  async logout(refreshToken: string): Promise<void> {
    await this.sessionRepo.deleteByToken(refreshToken);
  }
  
  private generateAccessToken(user: UserRow): string {
    return jwt.sign(
      {
        userId: user.id,
        email: user.email
      },
      config.jwt.secret,
      { expiresIn: config.jwt.expiresIn } // 15 min
    );
  }
  
  private generateRefreshToken(user: UserRow): string {
    return jwt.sign(
      {
        userId: user.id,
        email: user.email,
        type: 'refresh'
      },
      config.jwt.refreshSecret,
      { expiresIn: '7d' }
    );
  }
  
  private async createTokens(user: UserRow): Promise<LoginResponse> {
    const token = this.generateAccessToken(user);
    const refreshToken = this.generateRefreshToken(user);
    
    // Store refresh token in database for revocation
    await this.sessionRepo.create(user.id, refreshToken);
    
    return {
      token,
      refreshToken,
      user: { id: user.id, email: user.email }
    };
  }
}

// Middleware to require authentication
export const authMiddleware = (
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    throw new UnauthorizedError('No token provided');
  }
  
  try {
    const decoded = jwt.verify(token, config.jwt.secret) as TokenPayload;
    req.user = { userId: decoded.userId, email: decoded.email };
    next();
  } catch (error) {
    throw new UnauthorizedError('Invalid token');
  }
};
```

**Key decisions:**
- Access token: Short-lived (15 min), stored in header
- Refresh token: Long-lived (7 days), stored in database for revocation
- Password hashing: bcrypt (salted, slow), not MD5/SHA
- Token validation: Verify signature (stateless), not database lookup

### 7.3.2 Authorization Service (RBAC)

**Role-based access control: What can users do?**

```typescript
// features/auth/authorization.service.ts

interface Permission {
  resource: string; // 'orders', 'users', 'products'
  action: string;   // 'create', 'read', 'update', 'delete'
}

interface Role {
  name: string;
  permissions: Permission[];
}

export class AuthorizationService {
  private roles: Map<string, Role> = new Map([
    [
      'admin',
      {
        name: 'admin',
        permissions: [
          { resource: '*', action: '*' } // Admin: all access
        ]
      }
    ],
    [
      'user',
      {
        name: 'user',
        permissions: [
          { resource: 'orders', action: 'create' },
          { resource: 'orders', action: 'read' },
          { resource: 'orders', action: 'update' }, // Own orders only
          { resource: 'products', action: 'read' }
        ]
      }
    ],
    [
      'guest',
      {
        name: 'guest',
        permissions: [
          { resource: 'products', action: 'read' }
        ]
      }
    ]
  ]);
  
  async checkPermission(
    userId: string,
    resource: string,
    action: string
  ): Promise<boolean> {
    // Get user role from database
    const user = await userRepo.findById(userId);
    if (!user?.role) return false;
    
    const role = this.roles.get(user.role);
    if (!role) return false;
    
    // Check if user has permission
    return role.permissions.some(perm =>
      (perm.resource === '*' || perm.resource === resource) &&
      (perm.action === '*' || perm.action === action)
    );
  }
}

// Middleware to check authorization
export const authorize = (resource: string, action: string) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    const hasPermission = await authorizationService.checkPermission(
      req.user.userId,
      resource,
      action
    );
    
    if (!hasPermission) {
      throw new ForbiddenError(`No permission to ${action} ${resource}`);
    }
    
    next();
  };
};

// Usage in routes:
router.post(
  '/orders',
  authMiddleware,
  authorize('orders', 'create'),
  asyncHandler(orderController.create)
);
```

---

## 7.4 API IMPLEMENTATION

### 7.4.1 Controllers: HTTP Handling

**Controllers are thin: Request → Service → Response. No business logic here.**

```typescript
// features/users/users.controller.ts
import { Request, Response } from 'express';
import { UserService } from './users.service';

export class UserController {
  constructor(private userService: UserService) {}
  
  async getProfile(req: Request, res: Response): Promise<void> {
    // req.user set by authMiddleware
    const user = await this.userService.getUserById(req.user.userId);
    
    res.status(200).json({
      success: true,
      data: user
    });
  }
  
  async updateProfile(req: Request, res: Response): Promise<void> {
    const { firstName, lastName, phone } = req.body;
    
    const user = await this.userService.updateUser(req.user.userId, {
      firstName,
      lastName,
      phone
    });
    
    res.status(200).json({
      success: true,
      data: user
    });
  }
  
  async listUsers(req: Request, res: Response): Promise<void> {
    const page = parseInt(req.query.page as string) || 1;
    const limit = parseInt(req.query.limit as string) || 20;
    
    const { users, total } = await this.userService.listUsers(page, limit);
    
    res.status(200).json({
      success: true,
      data: users,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit)
      }
    });
  }
  
  async deleteUser(req: Request, res: Response): Promise<void> {
    const { id } = req.params;
    
    await this.userService.deleteUser(id);
    
    res.status(204).send();
  }
}
```

### 7.4.2 Request Validation

**Validate at entry point: Express middleware. Type-safe with Zod or Joi.**

```typescript
// shared/utils/validation.ts
import { z } from 'zod';

// Schema definitions
export const createUserSchema = z.object({
  email: z.string().email('Invalid email format'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  firstName: z.string().optional(),
  lastName: z.string().optional()
});

export const updateUserSchema = z.object({
  firstName: z.string().optional(),
  lastName: z.string().optional(),
  phone: z.string().regex(/^\+?[1-9]\d{1,14}$/, 'Invalid phone number').optional()
});

export const loginSchema = z.object({
  email: z.string().email('Invalid email format'),
  password: z.string().min(1, 'Password is required')
});

// Validation middleware
export const validate = (schema: z.ZodSchema) => {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);
    
    if (!result.success) {
      const fieldErrors = result.error.flatten().fieldErrors;
      const fields: Record<string, string[]> = {};
      
      Object.entries(fieldErrors).forEach(([key, errors]) => {
        fields[key] = errors as string[];
      });
      
      throw new ValidationError('Validation failed', fields);
    }
    
    // Type-safe: req.body now matches schema
    req.body = result.data;
    next();
  };
};

// Usage in routes:
router.post(
  '/register',
  validate(createUserSchema),
  asyncHandler(authController.register)
);

// In controller, req.body is type-safe
async register(req: Request, res: Response): Promise<void> {
  const { email, password, firstName, lastName } = req.body; // Typed
  // ...
}
```

### 7.4.3 Response Formatting

**Consistent responses: Success and error responses follow same structure.**

```typescript
// shared/utils/response.ts

interface SuccessResponse<T> {
  success: true;
  data: T;
  pagination?: {
    page: number;
    limit: number;
    total: number;
  };
}

interface ErrorResponse {
  success: false;
  error: {
    code: string;
    message: string;
    details?: any;
  };
}

// In controller:
res.status(200).json({
  success: true,
  data: user
});

// In error handler:
res.status(400).json({
  success: false,
  error: {
    code: 'VALIDATION_ERROR',
    message: 'Validation failed',
    details: { email: ['Email already registered'] }
  }
});
```

---

## 7.5 BACKGROUND JOBS

### 7.5.1 Job Queue Setup

**Async work (emails, reports, exports) shouldn't block HTTP requests. Use job queue.**

```typescript
// shared/jobs/job.ts
import Bull from 'bull';

// Redis-backed job queue
export const emailQueue = new Bull('emails', {
  redis: {
    host: config.redis.host,
    port: config.redis.port,
    password: config.redis.password
  }
});

export const reportQueue = new Bull('reports', {
  redis: { /* same as above */ }
});

// Job definitions
interface EmailJobData {
  userId: string;
  email: string;
  template: 'welcome' | 'reset-password' | 'order-confirmation';
  variables: Record<string, any>;
}

interface ReportJobData {
  userId: string;
  reportType: 'sales' | 'inventory';
  dateRange: { start: Date; end: Date };
}

// Publish job
async function sendWelcomeEmail(userId: string, email: string) {
  await emailQueue.add({
    userId,
    email,
    template: 'welcome',
    variables: { email }
  } as EmailJobData, {
    delay: 0,           // Send immediately
    attempts: 3,        // Retry 3 times
    backoff: {
      type: 'exponential',
      delay: 2000       // Start with 2s delay, exponential increase
    },
    removeOnComplete: true
  });
}

// Process job
emailQueue.process(async (job) => {
  const { email, template, variables } = job.data as EmailJobData;
  
  try {
    await emailService.send({
      to: email,
      template,
      variables
    });
  } catch (error) {
    // Retry on failure (Bull handles this)
    throw error;
  }
});

// Monitor job
emailQueue.on('completed', (job) => {
  logger.info('Email sent', { jobId: job.id, email: job.data.email });
});

emailQueue.on('failed', (job, err) => {
  logger.error('Email failed', err, { jobId: job.id, attempts: job.attemptsMade });
  
  // Send to dead letter queue if max retries exceeded
  if (job.attemptsMade >= job.opts.attempts!) {
    deadLetterQueue.add(job.data);
  }
});
```

### 7.5.2 Scheduled Tasks

**Recurring work (cleanup, analytics, sync) on schedule.**

```typescript
// shared/jobs/scheduled.ts
import cron from 'node-cron';

// Run every day at 2 AM
cron.schedule('0 2 * * *', async () => {
  logger.info('Running scheduled cleanup');
  
  try {
    // Delete expired sessions
    await sessionRepo.deleteExpired();
    
    // Archive old logs
    await logRepository.archiveOlderThan(30); // 30 days
    
    // Clean up temporary files
    await fileService.cleanupTemporary();
    
    logger.info('Cleanup completed');
  } catch (error) {
    logger.error('Cleanup failed', error);
  }
});

// Run every hour
cron.schedule('0 * * * *', async () => {
  logger.info('Running hourly analytics');
  
  try {
    await analyticsService.calculateMetrics();
  } catch (error) {
    logger.error('Analytics failed', error);
  }
});
```

---

## 7.6 TESTING

### 7.6.1 Unit Tests (70% of testing)

**Test business logic in isolation. Mock dependencies. Fast, deterministic.**

```typescript
// features/auth/auth.service.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { AuthService } from './auth.service';
import { UnauthorizedError, ConflictError } from '../../shared/types/errors';

describe('AuthService', () => {
  let authService: AuthService;
  let mockUserRepo: any;
  let mockSessionRepo: any;
  let mockLogger: any;
  
  beforeEach(() => {
    // Mock dependencies
    mockUserRepo = {
      findByEmail: vi.fn(),
      create: vi.fn(),
      findById: vi.fn()
    };
    
    mockSessionRepo = {
      create: vi.fn(),
      findByToken: vi.fn(),
      deleteByToken: vi.fn()
    };
    
    mockLogger = {
      setContext: vi.fn(),
      info: vi.fn(),
      error: vi.fn()
    };
    
    authService = new AuthService(mockUserRepo, mockSessionRepo, mockLogger);
  });
  
  describe('register', () => {
    it('should create new user with hashed password', async () => {
      const email = 'test@example.com';
      const password = 'password123';
      
      mockUserRepo.findByEmail.mockResolvedValue(null);
      mockUserRepo.create.mockResolvedValue({
        id: 'user-123',
        email,
        password_hash: 'hashed-password',
        created_at: new Date()
      });
      mockSessionRepo.create.mockResolvedValue({});
      
      const result = await authService.register(email, password);
      
      expect(result.token).toBeDefined();
      expect(result.refreshToken).toBeDefined();
      expect(mockUserRepo.create).toHaveBeenCalled();
    });
    
    it('should throw error if email already registered', async () => {
      mockUserRepo.findByEmail.mockResolvedValue({
        id: 'user-123',
        email: 'test@example.com'
      });
      
      await expect(
        authService.register('test@example.com', 'password123')
      ).rejects.toThrow(ConflictError);
      
      expect(mockUserRepo.create).not.toHaveBeenCalled();
    });
    
    it('should throw error if password too short', async () => {
      await expect(
        authService.register('test@example.com', 'short')
      ).rejects.toThrow('Password must be at least 8 characters');
      
      expect(mockUserRepo.findByEmail).not.toHaveBeenCalled();
    });
  });
  
  describe('login', () => {
    it('should return tokens on successful login', async () => {
      const email = 'test@example.com';
      const password = 'password123';
      const passwordHash = await crypto.bcrypt.hash(password, 12);
      
      mockUserRepo.findByEmail.mockResolvedValue({
        id: 'user-123',
        email,
        password_hash: passwordHash,
        created_at: new Date()
      });
      mockSessionRepo.create.mockResolvedValue({});
      
      const result = await authService.login(email, password);
      
      expect(result.token).toBeDefined();
      expect(result.user.id).toBe('user-123');
    });
    
    it('should throw error if user not found', async () => {
      mockUserRepo.findByEmail.mockResolvedValue(null);
      
      await expect(
        authService.login('test@example.com', 'password123')
      ).rejects.toThrow(UnauthorizedError);
    });
    
    it('should throw error if password incorrect', async () => {
      mockUserRepo.findByEmail.mockResolvedValue({
        id: 'user-123',
        email: 'test@example.com',
        password_hash: await crypto.bcrypt.hash('wrongpassword', 12)
      });
      
      await expect(
        authService.login('test@example.com', 'password123')
      ).rejects.toThrow(UnauthorizedError);
    });
  });
});
```

### 7.6.2 Integration Tests (20% of testing)

**Test features end-to-end: API endpoints + real database + real dependencies.**

```typescript
// features/auth/auth.integration.test.ts
import request from 'supertest';
import { describe, it, expect, beforeEach, afterEach } from 'vitest';
import app from '../../app';
import { dbConnection } from '../../shared/database/connection';

describe('Auth API Integration', () => {
  let testUserId: string;
  
  beforeEach(async () => {
    // Clear users table
    await dbConnection.query('DELETE FROM users');
  });
  
  afterEach(async () => {
    await dbConnection.query('DELETE FROM users');
  });
  
  describe('POST /auth/register', () => {
    it('should create user and return tokens', async () => {
      const response = await request(app)
        .post('/auth/register')
        .send({
          email: 'test@example.com',
          password: 'password123'
        });
      
      expect(response.status).toBe(201);
      expect(response.body.success).toBe(true);
      expect(response.body.data.token).toBeDefined();
      expect(response.body.data.user.email).toBe('test@example.com');
      
      // Verify user was created in database
      const user = await dbConnection.queryOne(
        'SELECT * FROM users WHERE email = $1',
        ['test@example.com']
      );
      expect(user).toBeDefined();
    });
    
    it('should return 409 if email already registered', async () => {
      // Create first user
      await request(app)
        .post('/auth/register')
        .send({
          email: 'test@example.com',
          password: 'password123'
        });
      
      // Try to register with same email
      const response = await request(app)
        .post('/auth/register')
        .send({
          email: 'test@example.com',
          password: 'different123'
        });
      
      expect(response.status).toBe(409);
      expect(response.body.error.code).toBe('ConflictError');
    });
    
    it('should return 400 for invalid email', async () => {
      const response = await request(app)
        .post('/auth/register')
        .send({
          email: 'invalid-email',
          password: 'password123'
        });
      
      expect(response.status).toBe(400);
      expect(response.body.error.fields.email).toBeDefined();
    });
  });
  
  describe('POST /auth/login', () => {
    beforeEach(async () => {
      // Create user
      await request(app)
        .post('/auth/register')
        .send({
          email: 'test@example.com',
          password: 'password123'
        });
    });
    
    it('should return tokens on successful login', async () => {
      const response = await request(app)
        .post('/auth/login')
        .send({
          email: 'test@example.com',
          password: 'password123'
        });
      
      expect(response.status).toBe(200);
      expect(response.body.data.token).toBeDefined();
      expect(response.body.data.refreshToken).toBeDefined();
    });
    
    it('should return 401 for invalid credentials', async () => {
      const response = await request(app)
        .post('/auth/login')
        .send({
          email: 'test@example.com',
          password: 'wrongpassword'
        });
      
      expect(response.status).toBe(401);
      expect(response.body.error.code).toBe('UnauthorizedError');
    });
  });
  
  describe('GET /users/profile', () => {
    let token: string;
    
    beforeEach(async () => {
      const response = await request(app)
        .post('/auth/register')
        .send({
          email: 'test@example.com',
          password: 'password123'
        });
      
      token = response.body.data.token;
    });
    
    it('should return user profile when authenticated', async () => {
      const response = await request(app)
        .get('/users/profile')
        .set('Authorization', `Bearer ${token}`);
      
      expect(response.status).toBe(200);
      expect(response.body.data.email).toBe('test@example.com');
    });
    
    it('should return 401 without token', async () => {
      const response = await request(app)
        .get('/users/profile');
      
      expect(response.status).toBe(401);
    });
    
    it('should return 401 with invalid token', async () => {
      const response = await request(app)
        .get('/users/profile')
        .set('Authorization', 'Bearer invalid-token');
      
      expect(response.status).toBe(401);
    });
  });
});
```

### 7.6.3 Test Coverage Target

**70% coverage: Focus on critical paths, not every line.**

```
src/
├── features/
│  ├── auth/
│  │  ├── auth.service.ts       ← 100% (critical: auth)
│  │  ├── auth.controller.ts    ← 90% (HTTP layer)
│  │  └── auth.repository.ts    ← 70% (data access)
│  │
│  └── users/
│     ├── users.service.ts      ← 85% (business logic)
│     ├── users.controller.ts   ← 80%
│     └── users.repository.ts   ← 70%
│
└── shared/
   ├── middleware/
   │  ├── errorHandler.ts       ← 95% (critical)
   │  ├── authMiddleware.ts     ← 100% (critical)
   │  └── validation.ts         ← 80%
   │
   └── utils/
      ├── crypto.ts             ← 90% (security critical)
      └── jwt.ts                ← 95% (security critical)

Target: 70%+ overall
Critical paths: 90%+
```

---

## 7.7 API DOCUMENTATION

### 7.7.1 OpenAPI/Swagger Documentation

**Auto-generated from code, not manual documentation (which gets outdated).**

```typescript
// features/auth/auth.routes.ts
import { Router } from 'express';

/**
 * @swagger
 * /auth/register:
 *   post:
 *     summary: Register new user
 *     tags: [Auth]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               email:
 *                 type: string
 *                 format: email
 *               password:
 *                 type: string
 *                 minLength: 8
 *     responses:
 *       201:
 *         description: User registered successfully
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   type: object
 *                   properties:
 *                     token:
 *                       type: string
 *                     refreshToken:
 *                       type: string
 *       400:
 *         description: Validation error
 *       409:
 *         description: Email already registered
 */
router.post(
  '/register',
  validate(createUserSchema),
  asyncHandler(authController.register)
);

/**
 * @swagger
 * /auth/login:
 *   post:
 *     summary: Login user
 *     tags: [Auth]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               email:
 *                 type: string
 *               password:
 *                 type: string
 *     responses:
 *       200:
 *         description: Login successful
 *       401:
 *         description: Invalid credentials
 */
router.post(
  '/login',
  validate(loginSchema),
  asyncHandler(authController.login)
);

// Setup Swagger in app.ts
import swaggerUi from 'swagger-ui-express';
import swaggerJsdoc from 'swagger-jsdoc';

const swaggerOptions = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'Orders API',
      version: '1.0.0',
      description: 'Orders management API'
    },
    servers: [
      { url: 'http://localhost:3000', description: 'Development' },
      { url: 'https://api.example.com', description: 'Production' }
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT'
        }
      }
    },
    security: [{ bearerAuth: [] }]
  },
  apis: ['./src/features/**/*.routes.ts']
};

const swaggerSpec = swaggerJsdoc(swaggerOptions);
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));
```

---

## 7.8 DELIVERABLES CHECKLIST

### Before shipping to production:

**Code Quality:**
- [ ] ESLint: 0 errors, warnings resolved
- [ ] Prettier: Consistent formatting
- [ ] TypeScript: Strict mode, no `any` types (except justified escapes)
- [ ] Code review: 2+ approvals from team

**Testing:**
- [ ] Unit tests: 70%+ coverage of critical paths
- [ ] Integration tests: All API endpoints tested
- [ ] Tests passing: 100% pass rate
- [ ] Performance tests: API response time < 200ms (p95)

**Security:**
- [ ] Dependencies: No critical vulnerabilities (`npm audit`)
- [ ] Secrets: No credentials in code, all in environment
- [ ] HTTPS: Enforced in production
- [ ] CORS: Configured for known origins
- [ ] Rate limiting: Implemented on sensitive endpoints
- [ ] Input validation: All inputs validated and sanitized
- [ ] Password hashing: bcrypt with salt (not MD5/SHA)

**Monitoring:**
- [ ] Logging: Structured logs for all critical operations
- [ ] Error tracking: Sentry or similar configured
- [ ] Metrics: Request count, error rate, response time tracked
- [ ] Alerts: Email/Slack alerts for errors > threshold

**Documentation:**
- [ ] README: How to setup, run, deploy
- [ ] API docs: Swagger/OpenAPI auto-generated
- [ ] Database schema: DDL documented
- [ ] Deployment: Step-by-step instructions

**Database:**
- [ ] Migrations: All DDL checked in, versioned
- [ ] Backups: Automated, tested restore
- [ ] Indexes: Critical queries indexed
- [ ] Connection pooling: Configured

**Deployment:**
- [ ] CI/CD: All tests pass before merge
- [ ] Environment parity: Dev == Staging == Prod (except secrets)
- [ ] Rollback plan: Quick rollback procedure documented
- [ ] Health check: `/health` endpoint returning 200

---

**Final Principle:**

A backend engineer's job is not to write code—it's to write systems that survive production. 

Code anyone can write. Systems that don't wake you up at 2 AM? That's the job.

Every line, every decision, every test exists to make that happen.

Start building.