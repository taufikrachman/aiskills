# Auth & Security

You are an authentication and security expert. Apply these rules to every backend and frontend.

## Authentication Rules

### 1. JWT Implementation
```ts
// Token Service — generate and verify
@Injectable()
export class TokenService {
  async generateTokenPair(userId: string): Promise<{ accessToken: string; refreshToken: string }> {
    const accessToken = jwt.sign({ sub: userId, scope: 'app' }, ACCESS_SECRET, { expiresIn: '15m' });
    const refreshToken = jwt.sign({ sub: userId, scope: 'refresh' }, REFRESH_SECRET, { expiresIn: '7d' });
    return { accessToken, refreshToken };
  }
}
```
- **Separate secrets** for access and refresh tokens.
- Access token: 15 minutes. Refresh token: 7 days.
- Refresh token rotation: revoke old session, issue new pair on each refresh.
- Store refresh token HASH (bcrypt/argon2) in DB, never the raw token.

### 2. Session Management
```ts
// Session entity
@Entity()
class Session {
  @Column() userId: string;
  @Column() refreshTokenHash: string;
  @Column() deviceId: string;
  @Column() expiresAt: Date;
  @Column({ nullable: true }) revokedAt: Date;
}
```
- One session per device. Revoke all sessions on password change.
- Invalidate sessions on logout.
- Enforce device limits per plan (Free = 1 device).

### 3. Frontend Auth Flow
```ts
// Dio interceptor pattern
onRequest: attach accessToken from SecureStorage
onError (401): try refresh → retry with new token → if fails, clear storage & redirect to login
```
- Store tokens in secure storage (Keychain/Keystore), NOT localStorage.
- Use interceptor for automatic token refresh.
- Coalesce concurrent 401s into single refresh call.

## Security Rules

### 4. Input Validation & Sanitization
- Validate ALL input at API boundary (DTOs + class-validator).
- Sanitize user input: strip HTML, script tags, SQL injection patterns.
- Use parameterized queries — NEVER template strings for SQL.
- Rate limit: 60 req/min per IP globally.
- Use `@nestjs/throttler` or similar.

### 5. HTTP Security Headers
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0  (deprecated, rely on CSP instead)
Content-Security-Policy: default-src 'self'; script-src 'self'
Referrer-Policy: strict-origin-when-cross-origin
```
Use `helmet` middleware.

### 6. CORS & CSRF
```ts
app.enableCors({
  origin: ['https://example.com', 'https://admin.example.com'],
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  credentials: true,
});
```
- CSRF protection for cookie-based auth. Not needed for token-based (Bearer header).

### 7. Password & PIN Hashing
- Passwords: bcrypt with salt rounds ≥ 12.
- PINs: bcrypt (4-6 digits, but bcrypt still best choice).
- NEVER store plain text. NEVER use MD5/SHA1/SHA256 for passwords.
- Lock account after N failed attempts (e.g., 5 attempts = 30 min lockout).

### 8. Data Protection
- PII (email, phone): encrypt at rest, mask in logs.
- API keys/secrets: environment variables only, never in code.
- Logs: sanitize sensitive data (tokens, passwords, emails).
- DB backups: encrypted.

## Security Checklist
- [ ] JWT with separate access/refresh secrets
- [ ] Refresh token rotation
- [ ] Token stored in secure storage (not localStorage)
- [ ] All endpoints validated (DTOs + class-validator)
- [ ] Rate limiting enabled
- [ ] HTTP security headers (helmet)
- [ ] CORS configured correctly
- [ ] Passwords hashed with bcrypt
- [ ] Account lockout after failed attempts
- [ ] PII encrypted at rest
- [ ] No secrets in source code
