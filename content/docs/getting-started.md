+++
title = "Getting Started"
weight = 1
+++

Get up and running with RustVNC libraries in minutes.

## Installation

Add the libraries to your `Cargo.toml`:

```toml
[dependencies]
rustvncserver = "2.0"
tokio = { version = "1", features = ["rt-multi-thread", "macros"] }
```

For encoding-only functionality:

```toml
[dependencies]
rfb-encodings = "0.1"
```

## Quick Start

Here's a minimal VNC server:

```rust
use rustvncserver::VncServer;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Create a 1920x1080 VNC server
    let server = VncServer::new(1920, 1080);

    // Optional: Set password
    server.set_password(Some("secret".to_string()));

    // Start listening on port 5900
    server.listen(5900).await?;

    Ok(())
}
```

## Updating the Framebuffer

Push framebuffer updates to connected clients:

```rust
// RGBA pixel data
let pixels: Vec<u8> = capture_screen();

// Update the entire framebuffer
server.update_framebuffer(&pixels, 0, 0, 1920, 1080);

// Or update a specific region
server.update_framebuffer(&region_pixels, x, y, width, height);
```

## Handling Events

Receive input events from VNC clients:

```rust
use rustvncserver::{VncServer, ServerEvent};

let mut events = server.events();

tokio::spawn(async move {
    while let Ok(event) = events.recv().await {
        match event {
            ServerEvent::ClientConnected { id, address } => {
                println!("Client {} connected from {}", id, address);
            }
            ServerEvent::PointerEvent { x, y, button_mask, .. } => {
                // Handle mouse movement/clicks
            }
            ServerEvent::KeyEvent { key, pressed, .. } => {
                // Handle keyboard input
            }
            ServerEvent::ClipboardReceived { text, .. } => {
                // Handle clipboard sync
            }
            _ => {}
        }
    }
});
```

## Next Steps

- [Server Configuration](/docs/server-config/) - Configure encodings, authentication, and more
- [Encodings](/docs/encodings/) - Learn about available encoding types
- [API Reference](https://docs.rs/rustvncserver) - Full API documentation
