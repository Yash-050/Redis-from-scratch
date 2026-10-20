# Redis-like Key-Value Store in C++ from Scratch

A minimal Redis implementation demonstrating core networking concepts:
- Non-blocking I/O server using `poll()` for handling multiple client connections
- Simple request-response protocol with length-prefixed messages
- Pipelining support allowing multiple requests without waiting for responses
- Echo server behavior (server sends back exactly what client sends)

## 📋 Project Overview

This project follows the "Build Your Own X From Scratch" principles to create a basic Redis-like key-value store in C/C++. While it doesn't implement actual data storage, it showcases the foundational networking and concurrency patterns used in Redis.

## 🔧 Dependencies

```bash
sudo apt-get install g++  # C++ compiler (g++)
```

## 📂 Project Structure

```
redis-from-scratch/
├── client/
│   └── main.cpp          # Client implementation
├── server/
│   ├── main.cpp          # Server implementation
│   └── helpers.h         # Shared constants, structs, and utility functions
├── scripts/
│   └── compiler.sh       # Build script
├── output/               # Generated binaries (created during build)
├── .gitignore
└── README.md
```

## 🛠️ How to Build & Run

### 1. Build the Project
```bash
cd redis-from-scratch
chmod +x scripts/compiler.sh  # Make executable if needed
./scripts/compiler.sh
```
This creates:
- `./output/server` - The Redis-like server
- `./output/client` - The test client

### 2. Run the Server
```bash
cd output
./server &
# Server starts on port 1234
```

### 3. Run the Client
```bash
./client
# Expected output:
# server says: hello1
# server says: hello2
# server says: hello3
```

### 4. Stop the Server
```bash
pkill -f "./server"
# or if running in background:
# kill %1
```

## 📚 Key STL Components Used

### Primary STL Usage:
- **`std::vector<Conn*>`**: Dynamically manages array of client connection pointers where index = file descriptor
  - Enables O(1) connection lookup by FD
  - Handles arbitrary number of concurrent connections
  - Demonstrates vector resizing, index access, and iteration

### System/Networking Headers (C++ equivalents):
- `<cstring>`: `memcpy()`, `memmove()`, `strlen()`
- `<cerrno>`: Error code handling
- `<cstdio>`: `printf()`, `fprintf()`
- `<cstdlib>`: `exit()`, `abort()`
- `<cfcntl>`: File control (`fcntl()` for non-blocking mode)
- `<cpoll>`: I/O multiplexing (`poll()` system call)
- `<cunistd>`: `read()`, `write()`, `close()`
- `<csys/socket.h>`: Socket operations
- `<cnetinet/in.h>`: IP address/port handling
- `<cassert>`: Debug assertions
- `<cstdint>`: Fixed-width integers (`uint32_t`)

## 🔍 Architecture Highlights

### Server (`server/main.cpp` + `server/helpers.h`):
1. **Connection Tracking**: `std::vector<Conn*> fd2conn` indexed by file descriptor
2. **Event Loop**: `poll()` monitors:
   - Listening socket for new connections (`POLLIN`)
   - Client sockets for read (`POLLIN` when STATE_REQ) or write (`POLLOUT` when STATE_RES)
3. **State Machine** per connection:
   - `STATE_REQ`: Reading client requests
   - `STATE_RES`: Sending server responses
   - `STATE_END`: Connection closed
4. **Non-blocking I/O**: Sockets set to `O_NONBLOCK` via `fcntl()`
5. **Buffer Management**:
   - Fixed-size buffers with 4-byte length prefix + message
   - Handles partial reads/writes via loops
   - Uses `memmove()` to shuffle buffer after processing

### Client (`client/main.cpp`):
- Simple pipeline: sends 3 requests, then reads 3 responses
- Length-prefixed protocol matching server expectations (4-byte length + message)
- Blocking I/O with simple `read()`/`write()` loops

## ⚙️ Core Networking Concepts Demonstrated

1. **File Descriptor Management**: Array-indexed vector for O(1) connection lookup
2. **Non-blocking Sockets**: Prevents thread blocking on I/O operations
3. **I/O Multiplexing**: `poll()` enables single-threaded concurrency handling
4. **Message Framing**: Length-prefixed protocol (4-byte length + payload)
5. **State Machines**: Per-connection state tracking for request/response handling
6. **Resource Cleanup**: Proper `close()` and `free()` on connection termination

## 📝 Current Limitations (vs Full Redis)

### What This Implements:
- Basic TCP server/client framework
- Connection state management
- Message framing with length prefixes
- Pipelining support
- Echo functionality (demonstration only)

### What's Missing:
- Actual key-value storage backend
- Command parsing (GET, SET, etc.)
- Persistence mechanisms (RDB/AOF)
- Replication capabilities
- Advanced data structures (lists, sets, hashes)
- Authentication system
- Complex data types and operations

## 🎯 Learning Outcomes

This project teaches:
- Network programming fundamentals in C/C++
- Effective `std::vector` usage for dynamic connection management
- Non-blocking I/O patterns with `poll()`
- State machine design for network protocols
- Buffer management techniques for efficient network I/O
- File descriptor-indexed connection tracking

## 🔩 STL Focus Area

The primary STL component to understand is **`std::vector`**, specifically how it's used here:
- Managing dynamic arrays where array index has special meaning (FD → connection mapping)
- Demonstrates vector resizing (`resize()`), direct index access, and iteration patterns
- Shows trade-offs between contiguous storage (cache-friendly) vs pointer indirection

For production use, consider:
- Smart pointers (`unique_ptr`/`shared_ptr`) for automatic memory management
- `unordered_map` for sparse FD spaces (though vector is efficient for sequential FDs)
- Advanced I/O multiplexing (`epoll` on Linux, `kqueue` on BSD/macOS) for better scalability

---

**Ready for GitHub upload!** Simply run `git init`, add files, commit, and push to your repository.