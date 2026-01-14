# Phase 1.4: Basic HTTP Server & Routing

**Phase Context**: Phase 1 builds the communication foundation. This sub-phase implements the HTTP server infrastructure.

**Sub-Phase Objective**: Implement HTTP server with routing, middleware chaining, and static file serving.

**Prerequisites**: 
- Phase 1.3 (Security) recommended

**Integration Point**: HTTP server provides the foundation for serving static files and proxying WebSocket connections.

---

## IMPLEMENTATION REQUIREMENTS

### Directory Structure

```
webos/
├── pkg/
│   └── server/
│       ├── doc.go              # Package documentation
│       ├── server.go           # HTTP server
│       ├── router.go           # Request router
│       ├── handler.go          # Handler interface
│       ├── middleware.go       # Middleware infrastructure
│       ├── static.go           # Static file serving
│       └── server_test.go      # Server tests
└── cmd/
    └── webos-server/
        └── main.go             # Demo server
```

### Handler Interface

```go
type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}

type HandlerFunc func(w ResponseWriter, r *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}

type ResponseWriter interface {
    Header() http.Header
    Write([]byte) (int, error)
    WriteHeader(statusCode int)
}

type Request struct {
    Method     string
    URL        *url.URL
    Proto      string
    Header     http.Header
    Body       io.Reader
    Context    context.Context
    RemoteAddr string
}
```

### Router

```go
type Route struct {
    Method     string
    Path       string
    Handler    Handler
    Middleware []Middleware
    Params     map[string]string
}

type Router struct {
    routes     []Route
    middleware []Middleware
    notFound   Handler
}

func NewRouter() *Router

func (r *Router) Get(path string, handler Handler)
func (r *Router) Post(path string, handler Handler)
func (r *Router) Put(path string, handler Handler)
func (r *Router) Delete(path string, handler Handler)
func (r *Router) Handle(method, path string, handler Handler)

func (r *Router) ServeHTTP(w ResponseWriter, req *Request)

func (r *Router) Use(middleware Middleware)
func (r *Router) UseFunc(middlewareFunc MiddlewareFunc)

func (r *Router) Group(prefix string, fn func(*Router))
```

### Middleware

```go
type Middleware interface {
    ServeHTTP(w ResponseWriter, r *Request, next Handler)
}

type MiddlewareFunc func(w ResponseWriter, r *Request, next Handler)

func (f MiddlewareFunc) ServeHTTP(w ResponseWriter, r *Request, next Handler) {
    f(w, r, next)
}

// Common middlewares
func Logging() Middleware
func Recover() Middleware
func CORS() Middleware
func Gzip() Middleware
func SecurityHeaders(headers *SecurityHeaders) Middleware
func RateLimit(limiter *RateLimiter) Middleware
```

### Server

```go
type ServerConfig struct {
    Addr         string
    ReadTimeout  time.Duration
    WriteTimeout time.Duration
    IdleTimeout  time.Duration
}

type Server struct {
    router  *Router
    config  *ServerConfig
    handler Handler
}

func NewServer(config *ServerConfig) *Server

func (s *Server) Use(middleware Middleware)
func (s *Server) UseFunc(middlewareFunc MiddlewareFunc)

func (s *Server) Get(path string, handler Handler)
func (s *Server) Post(path string, handler Handler)
func (s *Server) Put(path string, handler Handler)
func (s *Server) Delete(path string, handler Handler)
func (s *Server) Handle(method, path string, handler Handler)
func (s *Server) HandleFunc(method, path string, handler func(w ResponseWriter, r *Request))

func (s *Server) Serve(listener net.Listener) error
func (s *Server) ListenAndServe() error
func (s *Server) Shutdown(ctx context.Context) error

func (s *Server) FileServer(path string) Handler
func (s *Server) WebSocketHandler(path string, upgrader WebSocketUpgrader) Handler
```

### Static File Serving

```go
type StaticOptions struct {
    Directory   string
    IndexFiles  []string
    CacheControl map[string]string
    Gzip        bool
    Browsable   bool
}

func Static(options StaticOptions) Handler {
    // Serve files from directory
    // Handle index files (index.html, etc.)
    // Set cache headers
    // Enable gzip compression
    // Directory listing if enabled
}
```

### Context

```go
type Context struct {
    Request  *Request
    Response *Response
    Params   map[string]string
    handlers []Handler
    index    int
}

func (c *Context) Next() {
    c.index++
    for c.index < len(c.handlers) {
        c.handlers[c.index].ServeHTTP(c.Response, c.Request)
        c.index++
    }
}

func (c *Context) Param(key string) string
func (c *Context) Query(key string) string
func (c *Context) FormValue(key string) string

func (c *Context) JSON(status int, data interface{})
func (c *Context) String(status int, text string)
func (c *Context) File(filepath string)
func (c *Context) Redirect(url string, code int)
```

---

## TESTING REQUIREMENTS

1. **Routing**
   - Route matching
   - Method filtering
   - Parameter extraction
   - Wildcard routes
   - Route groups

2. **Middleware**
   - Middleware execution order
   - Chaining works correctly
   - Panic recovery
   - Context propagation

3. **Static Files**
   - Files served correctly
   - MIME types correct
   - Cache headers set
   - Gzip compression

4. **Server**
   - Clean shutdown
   - Timeout handling
   - Concurrent requests
   - Error responses

---

## DELIVERABLES

- `pkg/server/` - Complete server implementation
- `cmd/webos-server/` - Working demo server
- Integration with Phase 1.3 security
