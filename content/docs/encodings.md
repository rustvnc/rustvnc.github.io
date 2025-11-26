+++
title = "Encodings"
weight = 2
+++

RustVNC supports all standard RFB protocol encodings defined in RFC 6143.

## Supported Encodings

| Encoding | Type ID | Description | Best For |
|----------|---------|-------------|----------|
| **Raw** | 0 | Uncompressed pixels | LAN, fallback |
| **CopyRect** | 1 | Copy screen regions | Scrolling, window moves |
| **RRE** | 2 | Rise-and-Run-length | Simple graphics |
| **CoRRE** | 4 | Compact RRE | Small rectangles |
| **Hextile** | 5 | 16×16 tile-based | General purpose |
| **Zlib** | 6 | Zlib compression | High compression |
| **Tight** | 7 | Multi-mode adaptive | Universal (recommended) |
| **ZlibHex** | 8 | Zlib + Hextile | Alternative compression |
| **ZRLE** | 16 | Zlib Run-Length | Text, UI elements |
| **ZYWRLE** | 17 | Wavelet lossy | Low bandwidth |
| **TightPng** | -260 | PNG compression | Lossless quality |

## Tight Encoding Modes

Tight encoding automatically selects the best compression mode:

1. **Solid Fill** - Single color rectangles (5 bytes)
2. **Mono Rectangle** - Two-color 1-bit bitmaps
3. **Indexed Palette** - 3-16 colors with palette
4. **Zlib Compressed** - Lossless RGB24 compression
5. **JPEG** - Lossy compression (requires `turbojpeg` feature)

## Using rfb-encodings Directly

For custom implementations, use the encoding library directly:

```rust
use rfb_encodings::{Encoder, TightEncoder, PixelFormat};

let format = PixelFormat::rgba32();
let mut encoder = TightEncoder::new();

let encoded = encoder.encode(&pixels, width, height, &format)?;
```

## Performance Characteristics

For a 1920×1080 full-screen update:

| Encoding | Size | Compression Ratio | CPU Usage |
|----------|------|-------------------|-----------|
| Raw | 8.3 MB | 1:1 | Minimal |
| CopyRect | 8 bytes | ∞:1 | Minimal |
| Hextile | 800 KB - 2 MB | 4-10:1 | Low |
| Tight JPEG | 100-500 KB | 16-83:1 | Medium |
| ZRLE | 400 KB - 1.5 MB | 5-20:1 | Medium |
| ZYWRLE | 150-800 KB | 10-55:1 | High |

## TurboJPEG Feature

Enable hardware-accelerated JPEG compression:

```toml
[dependencies]
rustvncserver = { version = "2.0", features = ["turbojpeg"] }
```

Requires libjpeg-turbo installed on your system:

```bash
# Ubuntu/Debian
sudo apt-get install libturbojpeg0-dev

# macOS
brew install jpeg-turbo

# Windows
# Download from https://libjpeg-turbo.org/
```
