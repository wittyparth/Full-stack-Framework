# PHASE 10: SECURITY IMPLEMENTATION & AUDIT

## Overview
Security is not a feature—it's a property of your entire system. This phase covers security architecture, implementation patterns, and verification. Security issues discovered post-launch cost 10-100x more to fix than those caught during development.

***

## 10.1 SECURITY ARCHITECTURE

### 10.1.1 Defense in Depth Strategy

**What to do:**
- Never rely on a single security control
- Layer defenses so failure of one doesn't mean system compromise
- Assume each layer will eventually be bypassed
- Design for breach containment, not just prevention

**Defense layers:**

```
┌───────────────────────────────────────────────────┐
│  Layer 1: NETWORK SECURITY                        │
│  WAF, DDoS protection, firewall rules, VPN        │
├───────────────────────────────────────────────────┤
│  Layer 2: TRANSPORT SECURITY                      │
│  TLS 1.3, certificate pinning, HSTS               │
├───────────────────────────────────────────────────┤
│  Layer 3: APPLICATION SECURITY                    │
│  Authentication, authorization, input validation  │
├───────────────────────────────────────────────────┤
│  Layer 4: DATA SECURITY                           │
│  Encryption at rest, field-level encryption       │
├───────────────────────────────────────────────────┤
│  Layer 5: MONITORING & RESPONSE                   │
│  Audit logs, intrusion detection, alerting        │
└───────────────────────────────────────────────────┘
```

**Critical details:**
- Each layer operates independently
- Failure of outer layer triggers alerts for inner layers
- Regular testing of each layer in isolation
- Document what each layer protects against

**Output:** Defense-in-depth architecture diagram

***

### 10.1.2 Threat Modeling

**What to do:**
- Identify what you're protecting (assets)
- Identify who might attack (threat actors)
- Identify how they might attack (threat vectors)
- Prioritize threats by likelihood × impact

**STRIDE threat model:**

| Threat | Description | Example |
|--------|-------------|---------|
| **S**poofing | Impersonating another user/system | Stolen credentials, session hijacking |
| **T**ampering | Modifying data or code | SQL injection, parameter manipulation |
| **R**epudiation | Denying actions were taken | Missing audit logs |
| **I**nformation Disclosure | Exposing sensitive data | Data breach, error messages revealing internals |
| **D**enial of Service | Making system unavailable | DDoS, resource exhaustion |
| **E**levation of Privilege | Gaining unauthorized access | Broken access control, privilege escalation |

**Threat modeling process:**
```
1. Decompose application into components
2. Identify trust boundaries (where security context changes)
3. For each component, apply STRIDE
4. Rate each threat (High/Medium/Low probability × impact)
5. Define mitigations for High-rated threats
6. Accept or transfer remaining risks
```

**Output:** Threat model document with risk ratings

***

## 10.2 AUTHENTICATION

### 10.2.1 Authentication Strategy Selection

**Authentication options:**

| Method | Best For | Security Level |
|--------|----------|----------------|
| **Session-based** | Traditional web apps, server-rendered | High (if secured properly) |
| **JWT (stateless)** | APIs, microservices, mobile apps | Medium-High (short expiry critical) |
| **OAuth 2.0 / OIDC** | Third-party login, enterprise SSO | High (delegate to IdP) |
| **API Keys** | Machine-to-machine, simple integrations | Medium (no user context) |
| **mTLS** | Service-to-service, high security | Very High |

**JWT implementation rules:**

```javascript
// ✅ GOOD JWT configuration
{
  algorithm: "RS256",           // Asymmetric, more secure
  expiresIn: "15m",             // Short-lived access token
  issuer: "your-app.com",       // Verify issuer
  audience: "your-api.com",     // Verify audience
}

// ❌ BAD JWT configurations
{
  algorithm: "none",            // NEVER - allows unsigned tokens
  algorithm: "HS256",           // Symmetric key - shared secret risk
  expiresIn: "30d",             // Too long - stolen token = long exposure
}
```

**Critical details:**
- Access tokens: 15-60 minutes expiry maximum
- Refresh tokens: 7-30 days, stored securely, rotated on use
- Never store JWTs in localStorage (XSS vulnerable)
- Store in httpOnly, secure, sameSite cookies
- Always validate signature, expiry, issuer, audience

**Output:** Authentication architecture document

***

### 10.2.2 Password Security

**Password requirements:**
```
Minimum length: 12 characters (NIST recommends 8+, 12 is safer)
Complexity: Not required (length > complexity per NIST 800-63B)
Password history: Prevent reuse of last 10 passwords
Breached password check: Verify against HaveIBeenPwned
Rate limiting: 5 attempts, then progressive delay or lockout
```

**Password storage:**
```
// ✅ CORRECT: bcrypt with sufficient work factor
const hash = await bcrypt.hash(password, 12);

// ✅ ALSO CORRECT: Argon2id (memory-hard, preferred for new apps)
const hash = await argon2.hash(password, {
  type: argon2.argon2id,
  memoryCost: 65536,
  timeCost: 3,
  parallelism: 4
});

// ❌ WRONG: MD5, SHA1, SHA256 without salt
// ❌ WRONG: bcrypt with work factor < 10
// ❌ WRONG: Storing plaintext (obviously)
```

**Password reset flow:**
```
1. User requests reset → Generate cryptographically random token
2. Store token hash (not token) in database with expiry (1 hour)
3. Email link with token to user
4. User clicks link → Verify token hash, check expiry
5. User enters new password → Invalidate all existing sessions
6. Delete used token immediately
```

**Output:** Password policy document

***

### 10.2.3 Multi-Factor Authentication (MFA)

**MFA factors:**

| Factor | Type | Security | UX |
|--------|------|----------|-----|
| **TOTP (Authenticator app)** | Something you have | High | Good |
| **SMS OTP** | Something you have | Medium (SIM swap risk) | Convenient |
| **Email OTP** | Something you have | Medium | Convenient |
| **Hardware key (FIDO2/WebAuthn)** | Something you have | Very High | Medium |
| **Biometric** | Something you are | High | Excellent |

**MFA implementation:**
```
When to require MFA:
- Login from new device/location
- Sensitive actions (password change, financial transactions)
- Admin/elevated access
- After X days since last MFA

Recovery options:
- Backup codes (generate 10, show once, allow single use)
- Secondary email/phone verification
- Human verification (ID check) for recovery
```

**Critical details:**
- SMS is weakest factor (SIM swap attacks)
- TOTP apps (Google Authenticator, Authy) are baseline
- Hardware keys (YubiKey) for high-security use cases
- Always provide backup codes for account recovery

**Output:** MFA implementation plan

***

## 10.3 AUTHORIZATION

### 10.3.1 Authorization Models

**Common authorization models:**

| Model | Description | Use Case |
|-------|-------------|----------|
| **RBAC** (Role-Based) | Users assigned roles, roles have permissions | Most applications |
| **ABAC** (Attribute-Based) | Policies based on user/resource/environment attributes | Complex, dynamic rules |
| **ReBAC** (Relationship-Based) | Permissions based on resource relationships | Social apps, Google Drive-like sharing |
| **ACL** (Access Control List) | Explicit permission entries per resource | File systems, simple sharing |

**RBAC implementation:**
```
Roles:
├── admin (all permissions)
├── manager (read all, write own department)
├── user (read/write own data only)
└── viewer (read only)

Permission structure:
resource:action (e.g., users:read, orders:delete)

Check format:
can(user, 'orders:delete', orderResource)
```

**Critical details:**
- Always default deny (whitelist, not blacklist)
- Check authorization on every request (never trust client-side)
- Implement at API level, not just UI (security through obscurity fails)
- Log all authorization failures for auditing

**Output:** Authorization model design document

***

### 10.3.2 Broken Access Control Prevention

**OWASP #1 vulnerability - broken access control:**

```
// ❌ INSECURE: No ownership check
app.get('/api/orders/:id', (req, res) => {
  const order = await Order.findById(req.params.id);
  res.json(order); // Anyone can view any order!
});

// ✅ SECURE: Ownership verification
app.get('/api/orders/:id', (req, res) => {
  const order = await Order.findById(req.params.id);
  
  if (!order) return res.status(404).json({ error: 'Not found' });
  if (order.userId !== req.user.id) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  res.json(order);
});
```

**IDOR (Insecure Direct Object Reference) prevention:**
- Never trust user-supplied IDs without ownership check
- Use indirect references where possible (UUID instead of sequential IDs)
- Implement row-level security in database (PostgreSQL RLS)
- Always filter queries by authenticated user context

**Authorization testing checklist:**
- [ ] Can user A access user B's resources?
- [ ] Can regular user access admin endpoints?
- [ ] Can user access resources after permission revoked?
- [ ] Does horizontal privilege escalation work (same role, different data)?
- [ ] Does vertical privilege escalation work (lower role to higher)?

**Output:** Access control verification checklist

***

## 10.4 INPUT VALIDATION & OUTPUT ENCODING

### 10.4.1 Input Validation

**Validation strategy: Validate everything, trust nothing**

```
Input validation layers:
1. Client-side (UX only, not security)
2. API gateway (schema validation, rate limiting)
3. Application (business logic validation)
4. Database (constraints, triggers)
```

**Validation rules:**
```javascript
// Schema validation example (using Zod)
const userSchema = z.object({
  email: z.string().email().max(254),
  password: z.string().min(12).max(128),
  age: z.number().int().min(13).max(120),
  role: z.enum(['user', 'admin']).default('user'),
});

// Validate at API boundary
app.post('/api/users', (req, res) => {
  const result = userSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ errors: result.error.issues });
  }
  // Proceed with validated data
});
```

**Critical details:**
- Whitelist validation (allow known good) over blacklist (block known bad)
- Validate type, length, format, range
- Reject invalid input, don't sanitize silently (hiding issues)
- Use parameterized queries, never string concatenation for SQL

**Output:** Input validation requirements by field type

***

### 10.4.2 SQL Injection Prevention

**SQL injection is solved—if you use parameterized queries:**

```javascript
// ❌ VULNERABLE: String concatenation
const query = `SELECT * FROM users WHERE id = ${userId}`;
// Attack: userId = "1 OR 1=1" → Returns all users

// ✅ SAFE: Parameterized query
const query = 'SELECT * FROM users WHERE id = $1';
const result = await db.query(query, [userId]);

// ✅ SAFE: ORM with parameterization
const user = await User.findOne({ where: { id: userId } });
```

**Additional protections:**
- Least privilege database user (no DROP, CREATE permissions for app user)
- Stored procedures with parameterized inputs
- Web Application Firewall (WAF) as additional layer
- Regular SQL injection scanning (SQLMap in CI)

***

### 10.4.3 XSS Prevention

**Cross-Site Scripting (XSS) types:**

| Type | Attack Vector | Prevention |
|------|---------------|------------|
| **Stored XSS** | Malicious script saved in database | Output encoding |
| **Reflected XSS** | Script in URL parameter reflected | Output encoding |
| **DOM XSS** | Client-side script manipulation | Avoid innerHTML, use textContent |

**Output encoding:**
```javascript
// ❌ VULNERABLE: Raw HTML insertion
document.innerHTML = userInput;
element.innerHTML = `<span>${userInput}</span>`;

// ✅ SAFE: Text content (auto-escapes)
document.textContent = userInput;

// ✅ SAFE: Framework auto-escaping (React, Vue, Angular)
// React auto-escapes by default
<span>{userInput}</span>

// ⚠️ DANGEROUS: dangerouslySetInnerHTML (only with sanitized input)
<div dangerouslySetInnerHTML={{ __html: sanitizedHtml }} />
```

**Content Security Policy (CSP):**
```
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' https://trusted-cdn.com;
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  connect-src 'self' https://api.example.com;
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';
```

**Output:** XSS prevention guidelines

***

## 10.5 DATA PROTECTION

### 10.5.1 Encryption at Rest

**What to encrypt:**
- Personally Identifiable Information (PII)
- Payment card data (PCI-DSS requirement)
- Passwords (hashing, not encryption)
- API keys, secrets
- Health data (HIPAA requirement)

**Encryption strategies:**

| Strategy | Use Case | Complexity |
|----------|----------|------------|
| **Full Disk Encryption** | Cloud storage volumes | Low (managed by cloud) |
| **Database Encryption (TDE)** | Entire database | Low (database feature) |
| **Column-Level Encryption** | Specific sensitive fields | Medium |
| **Application-Level Encryption** | End-to-end encryption needs | High |

**Application-level encryption:**
```javascript
// Field-level encryption for sensitive data
const crypto = require('crypto');

class EncryptionService {
  constructor(key) {
    this.key = Buffer.from(key, 'hex'); // 256-bit key
    this.algorithm = 'aes-256-gcm';
  }

  encrypt(plaintext) {
    const iv = crypto.randomBytes(12);
    const cipher = crypto.createCipheriv(this.algorithm, this.key, iv);
    const encrypted = Buffer.concat([cipher.update(plaintext), cipher.final()]);
    const tag = cipher.getAuthTag();
    return { iv: iv.toString('hex'), encrypted: encrypted.toString('hex'), tag: tag.toString('hex') };
  }

  decrypt({ iv, encrypted, tag }) {
    const decipher = crypto.createDecipheriv(this.algorithm, this.key, Buffer.from(iv, 'hex'));
    decipher.setAuthTag(Buffer.from(tag, 'hex'));
    return Buffer.concat([decipher.update(Buffer.from(encrypted, 'hex')), decipher.final()]).toString();
  }
}
```

**Output:** Data encryption strategy document

***

### 10.5.2 Encryption in Transit

**TLS configuration:**
```
Minimum TLS version: 1.2 (prefer 1.3)
Allowed cipher suites (TLS 1.3):
- TLS_AES_256_GCM_SHA384
- TLS_AES_128_GCM_SHA256
- TLS_CHACHA20_POLY1305_SHA256

Disable:
- TLS 1.0, 1.1 (deprecated)
- SSLv2, SSLv3 (vulnerable)
- Weak ciphers (RC4, DES, 3DES)
```

**HTTP security headers:**
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0  // Deprecated, use CSP instead
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

**Certificate management:**
- Use Let's Encrypt for automatic renewal
- Monitor certificate expiry (alert 30 days before)
- Implement certificate pinning for mobile apps (optional, complex)
- Use separate certificates per environment

**Output:** TLS configuration specification

***

## 10.6 SECRETS MANAGEMENT

### 10.6.1 Secrets Strategy

**What counts as a secret:**
- Database credentials
- API keys (internal and external)
- Encryption keys
- OAuth client secrets
- Service account credentials
- TLS private keys

**Secrets management tools:**

| Tool | Best For | Complexity |
|------|----------|------------|
| **Environment variables** | Simple apps, containers | Low |
| **AWS Secrets Manager** | AWS-native apps | Low-Medium |
| **HashiCorp Vault** | Multi-cloud, enterprise | High |
| **Azure Key Vault** | Azure-native apps | Low-Medium |
| **Doppler/1Password** | Team/startup-friendly | Low |

**Secret handling rules:**
```
✅ DO:
- Store secrets in dedicated secrets manager
- Rotate secrets regularly (90 days minimum)
- Use different secrets per environment
- Audit secret access
- Encrypt secrets at rest

❌ DON'T:
- Commit secrets to Git (use pre-commit hooks)
- Store secrets in plaintext config files
- Log secrets (even accidentally)
- Share secrets via email/Slack
- Use same secret across environments
```

**Output:** Secrets management policy

***

### 10.6.2 Secret Rotation

**Rotation strategy:**
```
Rotation frequency:
- Database passwords: 90 days
- API keys: 180 days or on suspected compromise
- Encryption keys: Annually (with key versioning)
- Service account keys: 90 days

Rotation process:
1. Generate new secret
2. Deploy new secret alongside old
3. Verify new secret works
4. Remove old secret
5. Audit that old secret is no longer used
```

**Automated rotation (example: AWS Secrets Manager):**
```javascript
// Secrets Manager auto-rotation Lambda
exports.handler = async (event) => {
  const secretId = event.SecretId;
  const step = event.Step;

  switch (step) {
    case 'createSecret':
      // Generate new secret value
      break;
    case 'setSecret':
      // Set new secret in target service (e.g., RDS)
      break;
    case 'testSecret':
      // Verify new secret works
      break;
    case 'finishSecret':
      // Mark new secret as current
      break;
  }
};
```

**Output:** Secret rotation runbook

***

## 10.7 API SECURITY

### 10.7.1 Rate Limiting

**Rate limiting strategies:**

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **Fixed window** | X requests per minute | Simple, bursty |
| **Sliding window** | X requests in rolling window | Smoother distribution |
| **Token bucket** | Refilling bucket of tokens | Allows bursts with recovery |
| **Leaky bucket** | Constant drain rate | Steady rate enforcement |

**Rate limit configuration:**
```javascript
// Example rate limits
const rateLimits = {
  // Public endpoints
  'public:*': { requests: 100, window: '1m' },
  
  // Authentication (prevent brute force)
  'auth:login': { requests: 5, window: '15m' },
  'auth:register': { requests: 3, window: '1h' },
  'auth:password-reset': { requests: 3, window: '1h' },
  
  // Authenticated users
  'api:*': { requests: 1000, window: '1m' },
  
  // Heavy operations
  'api:export': { requests: 5, window: '1h' },
  'api:bulk-create': { requests: 10, window: '1m' },
};
```

**Rate limit response:**
```
HTTP/1.1 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1609459200
```

**Output:** Rate limiting configuration

***

### 10.7.2 CORS Configuration

**CORS (Cross-Origin Resource Sharing):**
```javascript
// ❌ DANGEROUS: Allow all origins
cors({ origin: '*' })

// ❌ DANGEROUS: Reflect origin header
cors({ origin: req.headers.origin })

// ✅ SECURE: Whitelist specific origins
const allowedOrigins = [
  'https://app.example.com',
  'https://admin.example.com',
];

cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
});
```

**Output:** CORS policy document

***

## 10.8 SECURITY TESTING

### 10.8.1 Static Application Security Testing (SAST)

**SAST tools:**
- **SonarQube**: Code quality + security
- **Semgrep**: Pattern-based scanning
- **Snyk Code**: Real-time vulnerability detection
- **CodeQL**: Deep semantic analysis (GitHub)

**SAST in CI/CD:**
```yaml
security-scan:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    - name: Run Semgrep
      uses: returntocorp/semgrep-action@v1
      with:
        config: "p/owasp-top-ten"
        
    - name: Run CodeQL
      uses: github/codeql-action/analyze@v2
```

**What SAST catches:**
- SQL injection patterns
- XSS vulnerabilities
- Hardcoded secrets
- Insecure cryptography
- Path traversal

***

### 10.8.2 Dynamic Application Security Testing (DAST)

**DAST tools:**
- **OWASP ZAP**: Open-source, comprehensive
- **Burp Suite**: Industry standard, commercial
- **Nuclei**: Template-based scanning

**DAST automation:**
```yaml
# OWASP ZAP in CI
dast-scan:
  runs-on: ubuntu-latest
  steps:
    - name: Start application
      run: docker-compose up -d
      
    - name: OWASP ZAP Scan
      uses: zaproxy/action-full-scan@v0.4.0
      with:
        target: 'https://staging.example.com'
        fail_action: true
        rules_file_name: 'zap-rules.tsv'
```

**What DAST catches:**
- Missing security headers
- Actual injection vulnerabilities
- Authentication bypass
- Information disclosure

***

### 10.8.3 Dependency Scanning

**Tools:**
- **Snyk**: Real-time vulnerability database
- **Dependabot**: GitHub-native, automatic PRs
- **Renovate**: Flexible dependency updates
- **npm audit / yarn audit**: Built-in for JavaScript

**CI integration:**
```yaml
dependency-scan:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    
    - name: Run Snyk
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        args: --severity-threshold=high
```

**Policy:**
- Block deployment on critical vulnerabilities
- Warn on high vulnerabilities
- Weekly scan even if no code changes
- Automatic PR for patch updates

***

## 10.9 SECURITY MONITORING & INCIDENT RESPONSE

### 10.9.1 Security Logging

**What to log:**
```
Authentication events:
- Login success/failure
- Logout
- Password change
- MFA enrollment/use

Authorization events:
- Access denied
- Privilege escalation attempts
- Admin actions

Data access:
- Sensitive data access (PII)
- Bulk data exports
- Data modifications

Security events:
- Rate limit violations
- Input validation failures
- Suspicious patterns
```

**Log format:**
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "WARN",
  "event": "AUTH_FAILURE",
  "userId": "user123",
  "ip": "192.168.1.100",
  "userAgent": "Mozilla/5.0...",
  "details": {
    "reason": "invalid_password",
    "attemptCount": 3
  }
}
```

**Critical details:**
- Never log passwords, tokens, or secrets
- Include correlation ID for request tracing
- Ship logs to central location (SIEM)
- Retain security logs for 1+ year (compliance)

***

### 10.9.2 Security Alerting

**Alert triggers:**
```
Critical (page on-call immediately):
- Multiple failed logins from single IP
- Admin account login from new location
- Database access outside application
- Security scan detection

High (respond within 1 hour):
- Unusual API usage patterns
- Rate limit exceeded significantly
- Failed authorization on sensitive endpoints

Medium (review within 24 hours):
- Single failed login attempts
- Minor vulnerability detected in scan
```

**Output:** Security alerting rules configuration

***

## 10.10 COMPLIANCE CHECKLIST

### 10.10.1 OWASP Top 10 Coverage

| Risk | Status | Mitigation |
|------|--------|------------|
| A01: Broken Access Control | [ ] | Ownership checks, RBAC |
| A02: Cryptographic Failures | [ ] | TLS 1.3, AES-256, bcrypt |
| A03: Injection | [ ] | Parameterized queries, input validation |
| A04: Insecure Design | [ ] | Threat modeling, secure patterns |
| A05: Security Misconfiguration | [ ] | Hardened defaults, security headers |
| A06: Vulnerable Components | [ ] | Dependency scanning, updates |
| A07: Auth Failures | [ ] | MFA, session management, rate limiting |
| A08: Integrity Failures | [ ] | Signed deployments, SRI |
| A09: Logging Failures | [ ] | Security logging, alerting |
| A10: SSRF | [ ] | URL validation, network isolation |

***

## 10.11 DELIVERABLES & GATES

### Deliverable 1: Security Architecture Document
**Contains:**
- Defense-in-depth layers
- Threat model
- Authentication/authorization strategy

### Deliverable 2: Security Implementation Checklist
**Contains:**
- OWASP Top 10 coverage
- Encryption configuration
- Secrets management setup

### Deliverable 3: Security Testing Results
**Contains:**
- SAST scan results
- DAST scan results
- Dependency vulnerability report
- Penetration test findings

### Deliverable 4: Security Runbooks
**Contains:**
- Incident response playbook
- Secret rotation procedures
- Security alert handling

***

## 10.12 PHASE 10 CHECKLIST

- [ ] Threat model completed
- [ ] Authentication implemented (JWT/session, MFA)
- [ ] Authorization implemented (RBAC/ABAC)
- [ ] Input validation on all endpoints
- [ ] Output encoding preventing XSS
- [ ] SQL injection prevention verified
- [ ] TLS 1.2+ enforced
- [ ] Security headers configured
- [ ] Secrets in secrets manager (not in code)
- [ ] SAST scan passing
- [ ] DAST scan passing
- [ ] Dependencies scanned and updated
- [ ] Security logging implemented
- [ ] Security alerting configured
- [ ] Incident response plan documented

***

## 10.13 ESTIMATED DURATION

**Timeline: 1-2 weeks** (integrated with development)

- Security architecture design: 2-3 days
- Authentication/authorization implementation: 3-5 days
- Security testing setup: 2-3 days
- Documentation and runbooks: 1-2 days

***

**END OF PHASE 10**

Security is everyone's job. The most secure code is code that's designed with security from the start, not patched after the fact.
