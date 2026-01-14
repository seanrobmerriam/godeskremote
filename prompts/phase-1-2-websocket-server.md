# Phase 1.2: WebSocket Server & Connection Management

**Phase Context**: Phase 1 builds the communication foundation. This sub-phase implements the WebSocket server.

**Sub-Phase Objective**: Implement a WebSocket server with connection pooling, session management, and frame handling.

**Prerequisites**: 
- Phase 1.1 (Protocol) must be complete

**Integration Point**: Server accepts connections, reads protocol messages, and routes to appropriate handlers.

---

## IMPLEMENTATION REQUIREMENTS

### Directory Structure

```
webos/
├── pkg/
│   └── websocket/
│       ├── doc.go              # Package documentation
│       ├── frame.go            # Frame structure and types
│       ├── frame_reader.go     # Frame reading logic
│       ├── frame_writer.go     # Frame writing logic
│       ├── handshake.go        # WebSocket handshake
│       ├── connection.go       # Connection management
│       ├── session.go          # Session management
│       ├── server.go           # Server implementation
│       └── server_test.go      # Server tests
└── cmd/
    └── websocket-demo/
        └── main.go             # Demo program
```

### Frame Types

```go
// Frame types (opcode values)
const (
    OpcodeContinuation = 0
    OpcodeText         = 1
    OpcodeBinary       = 2
    OpcodeClose        = 8
    OpcodePing         = 9
    OpcodePong         = 10
)

type Frame struct {
    Opcode   byte
    Payload  []byte
    Fin      bool
    RSV1     bool
    RSV2     bool
    RSV3     bool
}
```

### Frame Reader

```go
type FrameReader struct {
    reader io.Reader
}

func (r *FrameReader) ReadFrame() (*Frame, error) {
    // Read first byte (FIN + RSV + Opcode)
    // Read second byte (MASK + Payload length)
    // Handle extended payload length (16 or 64 bit)
    // Read mask key if masked
    // Read and unmask payload
}
```

### Frame Writer

```go
type FrameWriter struct {
    writer io.Writer
}

func (w *FrameWriter) WriteFrame(frame *Frame) error {
    // Write FIN + RSV + Opcode
    // Write MASK + Payload length
    // Write extended payload length if needed
    // Write mask key if needed
    // Write masked payload
}

func (w *FrameWriter) WriteText(data []byte) error
func (w *FrameWriter) WriteBinary(data []byte) error
func (w *FrameWriter) WritePing(data []byte) error
func (w *FrameWriter) WritePong(data []byte) error
func (w *FrameWriter) WriteClose(code uint16, reason string) error
```

### Handshake

```go
const WebSocketGUID = "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"

func GenerateAcceptKey(key string) string {
    // SHA1 hash of key + GUID
    // Base64 encode
}

func ValidateRequest(r *http.Request) (string, error) {
    // Check HTTP method (GET)
    // Check Upgrade header (websocket)
    // Check Connection header (Upgrade)
    // Validate Sec-WebSocket-Key
}
```

### Connection

```go
type Connection struct {
    ID        string
    Conn      net.Conn
    Session   *Session
    CreatedAt time.Time
    LastPing  time.Time
    
    reader *FrameReader
    writer *FrameWriter
}

func (c *Connection) ReadFrame() (*Frame, error)
func (c *Connection) WriteFrame(frame *Frame) error
func (c *Connection) ReadMessage() (*protocol.Message, error)
func (c *Connection) WriteMessage(msg *protocol.Message) error
func (c *Connection) Close() error
```

### Session

```go
type Session struct {
    ID      string
    UserID  string
    Created time.Time
    Data    map[string]interface{}
}

type SessionManager struct {
    sessions map[string]*Session
    mu       sync.RWMutex
}

func (m *SessionManager) Create(id, userID string) (*Session, error)
func (m *SessionManager) Get(id string) (*Session, error)
func (m *SessionManager) Destroy(id string) error
func (m *SessionManager) DestroyAll()
func (m *SessionManager) Count() int
```

### Server

```go
type ServerConfig struct {
    Addr          string
    PoolConfig    *PoolConfig
    SessionConfig *SessionConfig
    ReadTimeout   time.Duration
    WriteTimeout  time.Duration
    PingInterval  time.Duration
}

type PoolConfig struct {
    MaxConnections int
    MaxIdle        int
}

type Server struct {
    pool        *ConnectionPool
    sessions    *SessionManager
    handler     MessageHandler
    onAccept    func(*Connection)
    onUpgrade   func(*Connection)
}

func (s *Server) SetHandler(fn func(*Connection, *protocol.Message) error)
func (s *Server) OnAccept(fn func(*Connection))
func (s *Server) OnUpgrade(fn func(*Connection))
func (s *Server) Start() error
func (s *Server) Stop() error
func (s *Server) Pool() *ConnectionPool
func (s *Server) SessionManager() *SessionManager
```

### Connection Pool

```go
type ConnectionPool struct {
    connections map[string]*Connection
    mu          sync.RWMutex
    stats       PoolStats
}

type PoolStats struct {
    ActiveConnections int
    TotalAccepted     int
    TotalClosed       int
    TotalFailed       int
}

func (p *ConnectionPool) Add(conn *Connection)
func (p *ConnectionPool) Remove(conn *Connection)
func (p *ConnectionPool) Get(id string) (*Connection, error)
func (p *ConnectionPool) All() []*Connection
func (p *ConnectionPool) Count() int
func (p *ConnectionPool) BroadcastMessage(msg *protocol.Message)
```

---

## TESTING REQUIREMENTS

1. **Frame Operations**
   - Read/write all frame types
   - Mask/unmask payload
   - Fragmentation (multi-frame messages)
   - Invalid frame handling

2. **Handshake**
   - Valid handshake
   - Invalid method rejection
   - Missing headers rejection
   - Key validation

3. **Connection Management**
   - Connection lifecycle
   - Multiple concurrent connections
   - Connection cleanup on close

4. **Session Management**
   - Create/get/destroy sessions
   - Session data storage
   - Concurrent access

5. **Pool Operations**
   - Add/remove connections
   - Broadcast messages
   - Statistics tracking

---

## DELIVERABLES

- `pkg/websocket/` - Complete WebSocket implementation
- `cmd/websocket-demo/` - Working demo program
- Integration with Phase 1.1 protocol
