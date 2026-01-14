# Phase 1.3: Security Foundation

**Phase Context**: Phase 1 builds the communication foundation. This sub-phase implements security controls.

**Sub-Phase Objective**: Implement rate limiting, request validation, security headers, and input sanitization.

**Prerequisites**: 
- Phase 1.1 (Protocol) recommended

**Integration Point**: Security middleware integrates with WebSocket server (Phase 1.2).

---

## IMPLEMENTATION REQUIREMENTS

### Directory Structure

```
webos/
├── pkg/
│   └── security/
│       ├── doc.go              # Package documentation
│       ├── ratelimit.go        # Rate limiting
│       ├── validation.go       # Request validation
│       ├── headers.go          # Security headers
│       └── sanitizer.go        # Input sanitization
└── cmd/
    └── security-demo/
        └── main.go             # Demo program
```

### Rate Limiting

```go
type RateLimiter struct {
    requests  map[string][]time.Time
    limit     int
    window    time.Duration
    mu        sync.RWMutex
}

func NewRateLimiter(limit int, window time.Duration) *RateLimiter

func (l *RateLimiter) Allow(key string) bool {
    // Check if key has exceeded rate limit
    // Clean old entries
    // Add new request timestamp
    // Return true if allowed, false if rate limited
}

func (l *RateLimiter) LimitExceeded(key string) bool
func (l *RateLimiter) Reset(key string)
func (l *RateLimiter) ResetAll()
```

### Request Validation

```go
type ValidationResult struct {
    Valid  bool
    Errors []string
}

type RequestValidator struct {
    maxMessageSize int
    maxPayloadSize int
    allowedPaths   map[string]bool
    allowedOpcodes map[byte]bool
}

func NewRequestValidator() *RequestValidator

func (v *RequestValidator) ValidateMessage(data []byte) *ValidationResult {
    // Check message size limits
    // Validate opcode
    // Check for suspicious patterns
}

func (v *RequestValidator) ValidatePath(path string) *ValidationResult {
    // Check path is allowed
    // Check for path traversal
    // Check for null bytes
}

func (v *RequestValidator) ValidateAuthToken(token string) *ValidationResult {
    // Check token format
    // Check token expiration
}
```

### Security Headers

```go
type SecurityHeaders struct {
    ContentSecurityPolicy    string
    XFrameOptions            string
    XContentTypeOptions      string
    StrictTransportSecurity  string
    XXSSProtection           string
    ReferrerPolicy           string
}

func DefaultSecurityHeaders() *SecurityHeaders {
    return &SecurityHeaders{
        ContentSecurityPolicy: "default-src 'self'",
        XFrameOptions:         "DENY",
        XContentTypeOptions:   "nosniff",
        StrictTransportSecurity: "max-age=31536000; includeSubDomains",
        XXSSProtection:       "1; mode=block",
        ReferrerPolicy:       "strict-origin-when-cross-origin",
    }
}

func (h *SecurityHeaders) Apply(headers http.Header)
func (h *SecurityHeaders) Middleware(next http.Handler) http.Handler
```

### Input Sanitization

```go
type Sanitizer struct {
    maxLength      int
    allowedChars   *unicode.RangeTable
    stripTags      bool
    escapeHTML     bool
}

func NewSanitizer() *Sanitizer

func (s *Sanitizer) Sanitize(input string) string {
    // Truncate to max length
    // Remove/disallow dangerous characters
    // Strip HTML tags if configured
    // Escape HTML entities if configured
}

func (s *Sanitizer) SanitizeBytes(input []byte) []byte

func (s *Sanitizer) ValidateFilename(filename string) error {
    // Check for path traversal
    // Check for null bytes
    // Check for reserved names
    // Check file extension
}

func (s *Sanitizer) ValidateURL(url string) error {
    // Check for JavaScript protocol
    // Check for data URLs
    // Check for relative URLs
}
```

### Security Middleware

```go
type SecurityConfig struct {
    RateLimit     RateLimitConfig
    Headers       bool
    Validation    bool
    Sanitization  bool
}

type SecurityMiddleware struct {
    config  *SecurityConfig
    limiter *RateLimiter
    validator *RequestValidator
    sanitizer *Sanitizer
}

func NewSecurityMiddleware(config *SecurityConfig) *SecurityMiddleware

func (m *SecurityMiddleware) WrapConnection(conn net.Conn) net.Conn {
    // Return wrapped connection with security checks
}

func (m *SecurityMiddleware) Middleware(next http.Handler) http.Handler
```

---

## TESTING REQUIREMENTS

1. **Rate Limiting**
   - Limit enforcement
   - Window cleanup
   - Multiple keys
   - Reset functionality

2. **Request Validation**
   - Valid requests pass
   - Invalid requests rejected
   - Edge cases handled
   - Error messages clear

3. **Security Headers**
   - Headers set correctly
   - Middleware applies headers
   - CSP allows necessary sources

4. **Input Sanitization**
   - Dangerous input cleaned
   - Valid input preserved
   - Filename validation
   - URL validation

---

## DELIVERABLES

- `pkg/security/` - Complete security implementation
- `cmd/security-demo/` - Working demo program
- Integration with Phase 1.2 WebSocket server
