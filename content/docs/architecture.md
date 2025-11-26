+++
title = "Architecture"
weight = 3
+++

Understanding the RustVNC library ecosystem.

## Library Structure

```
┌─────────────────────────────────────────────┐
│         Your Application                     │
│    (VNC server, client, proxy, recorder)    │
└─────────────────┬───────────────────────────┘
                  │ uses
┌─────────────────▼───────────────────────────┐
│         rustvncserver                        │
│    ├─ Protocol handling (RFC 6143)          │
│    ├─ Client connection management          │
│    ├─ Authentication                        │
│    └─ Async I/O (Tokio)                     │
└─────────────────┬───────────────────────────┘
                  │ uses
┌─────────────────▼───────────────────────────┐
│         rfb-encodings                        │
│    ├─ 10 encoding implementations           │
│    ├─ Compression stream management         │
│    └─ Pixel format translation              │
└─────────────────────────────────────────────┘
```

## Design Principles

### Memory Safety

All libraries are written in safe Rust. No `unsafe` blocks in core logic. This eliminates:

- Buffer overflows
- Use-after-free bugs
- Data races
- Null pointer dereferences

### Zero-Copy Architecture

Framebuffer data is shared using `Arc<[u8]>`:

```rust
// Single allocation shared by all clients
let framebuffer = Arc::new(pixel_data);

// Clients read directly, no copying
for client in clients {
    client.send_update(Arc::clone(&framebuffer));
}
```

### Persistent Compression Streams

Zlib compressors maintain state across frames for better compression:

```rust
// Tight encoding uses 4 persistent streams
struct TightStreamState {
    basic_zlib: ZlibEncoder,   // Stream 0
    fill_zlib: ZlibEncoder,    // Stream 1
    jpeg_zlib: ZlibEncoder,    // Stream 2
    filter_zlib: ZlibEncoder,  // Stream 3
}
```

### Async I/O

Built on Tokio for efficient concurrent connections:

```rust
// Handles hundreds of clients efficiently
// No thread-per-client overhead
async fn handle_client(socket: TcpStream, server: Arc<VncServer>) {
    // Non-blocking I/O
    loop {
        let message = read_message(&socket).await?;
        process(message).await;
    }
}
```

## Using rfb-encodings Standalone

The encoding library can be used independently:

```rust
use rfb_encodings::{TightEncoder, ZrleEncoder, PixelFormat};

// For a VNC client decoder
let decoder = ZrleDecoder::new();
let pixels = decoder.decode(&wire_data, width, height, &format)?;

// For a VNC proxy/recorder
let encoder = TightEncoder::new();
let wire_data = encoder.encode(&pixels, width, height, &format)?;
```

## Thread Safety

All public types implement `Send` and `Sync`:

```rust
// Safe to share across threads
let server = Arc::new(VncServer::new(1920, 1080));

// Spawn from multiple threads
let server_clone = Arc::clone(&server);
std::thread::spawn(move || {
    server_clone.update_framebuffer(&pixels, 0, 0, w, h);
});
```
