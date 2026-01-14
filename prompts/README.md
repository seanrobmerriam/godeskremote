# WebOS Project Prompts

This directory contains task prompts for each phase of the WebOS project.

## Phase 1: Communication Foundation

Phase 1 establishes the core communication infrastructure for the web-based operating system.

### Phase 1.1: Custom Protocol Design & Implementation
- Design a binary protocol for client-server communication
- Implement message encoding/decoding
- Create codec for binary data manipulation

**Deliverables:**
- `pkg/protocol/message.go` - Message structure
- `pkg/protocol/codec.go` - Binary codec
- `pkg/protocol/const.go` - Protocol constants
- `pkg/protocol/opcode.go` - Message opcodes
- `cmd/protocol-demo/main.go` - Demo program

### Phase 1.2: WebSocket Server & Connection Management
- Implement WebSocket server
- Handle connections with connection pooling
- Session management
- Frame parsing and writing

**Deliverables:**
- `pkg/websocket/handshake.go` - WebSocket handshake
- `pkg/websocket/frame.go` - Frame structure
- `pkg/websocket/frame_reader.go` - Frame reading
- `pkg/websocket/frame_writer.go` - Frame writing
- `pkg/websocket/connection.go` - Connection management
- `pkg/websocket/session.go` - Session management
- `pkg/websocket/server.go` - Server implementation
- `cmd/websocket-demo/main.go` - Demo program

### Phase 1.3: Security Foundation
- Rate limiting
- Request validation
- Security headers
- Input sanitization

**Deliverables:**
- `pkg/security/ratelimit.go` - Rate limiting
- `pkg/security/validation.go` - Request validation
- `pkg/security/headers.go` - Security headers
- `cmd/security-demo/main.go` - Demo program

### Phase 1.4: Basic HTTP Server & Routing
- HTTP server implementation
- Request routing
- Static file serving
- Handler chaining

**Deliverables:**
- `pkg/server/server.go` - HTTP server
- `pkg/server/router.go` - Request router
- `pkg/server/handler.go` - Handler interface
- `cmd/webos-server/main.go` - Demo server

### Phase 1.5: Client Foundation (JavaScript)
- Browser-based JavaScript client
- WebSocket connection management
- Canvas rendering
- Input event capture
- UI shell with window management

**Deliverables:**
- `static/index.html` - Main HTML page
- `static/js/protocol.js` - Protocol client
- `static/js/connection.js` - WebSocket management
- `static/js/display.js` - Canvas rendering
- `static/js/input.js` - Event capture
- `static/js/shell.js` - UI shell
- `static/js/state.js` - State management
- `static/js/client.js` - Main client
- `static/css/style.css` - Styling
- `cmd/client-demo/main.go` - Demo server
