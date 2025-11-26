# GBLN Build System Documentation

**Version:** 1.0  
**Last Updated:** 2025-11-26  
**Status:** Production

---

## Overview

This document describes the complete GBLN build system for all supported platforms. The build system uses **Rust + Cargo** as the primary build tool, with **cross** for Docker-based cross-compilation.

**Supported Platforms:** 10  
**Build Method:** Native + Cross-compilation  
**CI/CD:** GitHub Actions (planned)  
**Documentation:** See `core/rust/BUILD_MATRIX.md` for complete platform matrix

---

## Build Architecture

```
Host: macOS ARM64 (or any platform with Rust + Docker)
   ↓
┌──────────────────────────────────────────────────────┐
│  Native Builds (cargo)                               │
│  - macOS ARM64 (host)                                │
│  - macOS x86_64, iOS ARM64 (cross-arch on macOS)    │
└──────────────────────────────────────────────────────┘
   ↓
┌──────────────────────────────────────────────────────┐
│  Cross-Compilation (cross + Docker)                  │
│  - FreeBSD x86_64, ARM64                             │
│  - Linux x86_64, ARM64                               │
│  - Windows x86_64 (MinGW)                            │
│  - Android ARM64, x86_64                             │
└──────────────────────────────────────────────────────┘
   ↓
┌──────────────────────────────────────────────────────┐
│  Build Artifacts                                     │
│  target/{triple}/release/libgbln.rlib                │
│  (or target/release/ for native)                     │
└──────────────────────────────────────────────────────┘
```

---

## Prerequisites

### Required Tools

1. **Rust Toolchain** (via rustup):
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   source "$HOME/.cargo/env"
   ```

2. **cross** (for cross-compilation):
   ```bash
   cargo install cross --git https://github.com/cross-rs/cross
   ```

3. **Docker Desktop** (required by cross):
   - macOS: Download from https://www.docker.com/products/docker-desktop
   - Linux: Use system package manager
   - Must be running before using `cross`

### Optional Tools

- **rustfmt**: Code formatting (usually included)
- **clippy**: Linting (usually included)
- **cargo-watch**: Auto-rebuild on file changes

---

## Platform Matrix

| Platform | Target Triple | Method | Tier | Status |
|----------|---------------|--------|------|--------|
| FreeBSD ARM64 | `aarch64-unknown-freebsd` | cross + nightly | 3 | ✅ 361 KB |
| FreeBSD x86_64 | `x86_64-unknown-freebsd` | cross | 2 | ✅ 384 KB |
| Linux ARM64 | `aarch64-unknown-linux-gnu` | cross | 2 | ✅ 386 KB |
| Linux x86_64 | `x86_64-unknown-linux-gnu` | cross | 1 | ✅ 385 KB |
| macOS ARM64 | `aarch64-apple-darwin` | cargo (native) | 2 | ✅ 296 KB |
| macOS x86_64 | `x86_64-apple-darwin` | cargo | 1 | ✅ 291 KB |
| Windows x86_64 | `x86_64-pc-windows-gnu` | cross (MinGW) | - | ✅ 290 KB |
| iOS ARM64 | `aarch64-apple-ios` | cargo | 2 | ✅ 290 KB |
| Android ARM64 | `aarch64-linux-android` | cross | 2 | ✅ 390 KB |
| Android x86_64 | `x86_64-linux-android` | cross | 2 | ✅ 387 KB |

**Rust Tier Explanation:**
- **Tier 1**: Guaranteed to work, tested on CI
- **Tier 2**: Guaranteed to build, best effort testing
- **Tier 3**: No guarantees, requires special setup

---

## Building All Platforms

### Quick Start - Build Everything

```bash
cd /path/to/GBLN/core/rust

# Install all required Rust targets
rustup target add \
    x86_64-unknown-freebsd \
    aarch64-unknown-freebsd \
    x86_64-unknown-linux-gnu \
    aarch64-unknown-linux-gnu \
    x86_64-apple-darwin \
    aarch64-apple-darwin \
    x86_64-pc-windows-gnu \
    aarch64-apple-ios \
    aarch64-linux-android \
    x86_64-linux-android

# Install nightly for Tier 3 targets (FreeBSD ARM64)
rustup toolchain install nightly
rustup component add rust-src --toolchain nightly

# Start Docker
open -a Docker  # macOS
# or: systemctl start docker  # Linux

# Wait for Docker to start
sleep 15

# Build all platforms (10-15 minutes total)
# Native builds
cargo build --release --target aarch64-apple-darwin  # macOS ARM64 (native)
cargo build --release --target x86_64-apple-darwin    # macOS x86_64
cargo build --release --target aarch64-apple-ios      # iOS ARM64

# Cross-compilation
cross build --release --target x86_64-unknown-freebsd
cross +nightly build --release --target aarch64-unknown-freebsd  # Tier 3
cross build --release --target x86_64-unknown-linux-gnu
cross build --release --target aarch64-unknown-linux-gnu
cross build --release --target x86_64-pc-windows-gnu  # MinGW
cross build --release --target aarch64-linux-android
cross build --release --target x86_64-linux-android
```

### Build Script (Automated)

Create `scripts/build-all.sh`:

```bash
#!/bin/bash
set -euo pipefail

cd "$(dirname "$0")/../core/rust"
source "$HOME/.cargo/env"

TARGETS=(
    "aarch64-apple-darwin"      # Native on Apple Silicon
    "x86_64-apple-darwin"
    "aarch64-apple-ios"
    "x86_64-unknown-freebsd"
    "aarch64-unknown-freebsd"   # Tier 3 - needs nightly
    "x86_64-unknown-linux-gnu"
    "aarch64-unknown-linux-gnu"
    "x86_64-pc-windows-gnu"
    "aarch64-linux-android"
    "x86_64-linux-android"
)

echo "Building GBLN for all platforms..."
echo "===================================="

for target in "${TARGETS[@]}"; do
    echo ""
    echo "Building: $target"
    
    if [[ "$target" == "aarch64-apple-darwin" ]]; then
        # Native build
        cargo build --release --target "$target"
    elif [[ "$target" == "aarch64-unknown-freebsd" ]]; then
        # Tier 3 - needs nightly
        cross +nightly build --release --target "$target"
    else
        # Cross-compilation
        cross build --release --target "$target"
    fi
    
    echo "✓ Built: $target"
done

echo ""
echo "All platforms built successfully!"
```

Usage:
```bash
chmod +x scripts/build-all.sh
./scripts/build-all.sh
```

---

## Platform-Specific Instructions

### FreeBSD (Both Architectures)

**Requirements:**
- Rust nightly toolchain (for ARM64 only)
- `cross` tool
- Docker running

**Configuration** (`core/rust/Cross.toml`):
```toml
[build]
default-target = "aarch64-unknown-freebsd"

[target.aarch64-unknown-freebsd]
build-std = ["core", "alloc", "std", "panic_abort"]
```

**Build commands:**
```bash
# x86_64 (Tier 2)
cross build --release --target x86_64-unknown-freebsd

# ARM64 (Tier 3 - requires nightly + build-std)
rustup component add rust-src --toolchain nightly
cross +nightly build --release --target aarch64-unknown-freebsd
```

**Why nightly for ARM64?**
- FreeBSD ARM64 is Tier 3 (no pre-built standard library)
- Requires `-Z build-std` to compile std from source
- Only available on nightly toolchain

### Linux (Both Architectures)

**Requirements:**
- `cross` tool
- Docker running

**Build commands:**
```bash
# x86_64 (Tier 1)
cross build --release --target x86_64-unknown-linux-gnu

# ARM64 (Tier 2)
cross build --release --target aarch64-unknown-linux-gnu
```

### macOS (Both Architectures)

**Requirements:**
- Native Rust toolchain
- Xcode Command Line Tools

**Build commands:**
```bash
# ARM64 (native on Apple Silicon)
cargo build --release --target aarch64-apple-darwin

# x86_64 (cross-compile on Apple Silicon)
rustup target add x86_64-apple-darwin
cargo build --release --target x86_64-apple-darwin
```

**Artifact location:**
- Native (ARM64): `target/release/libgbln.rlib`
- x86_64: `target/x86_64-apple-darwin/release/libgbln.rlib`

### Windows (x86_64)

**Requirements:**
- `cross` tool
- Docker running

**Build command:**
```bash
# MinGW (GNU ABI)
rustup target add x86_64-pc-windows-gnu
cross build --release --target x86_64-pc-windows-gnu
```

**Note:** MSVC not available for cross-compilation from macOS/Linux. MinGW produces compatible Windows binaries with GNU ABI.

For MSVC builds, use Windows CI runner:
```bash
# On Windows with MSVC
rustup target add x86_64-pc-windows-msvc
cargo build --release --target x86_64-pc-windows-msvc
```

### iOS (ARM64)

**Requirements:**
- Xcode with iOS SDK
- Native Rust toolchain

**Build command:**
```bash
rustup target add aarch64-apple-ios
cargo build --release --target aarch64-apple-ios
```

### Android (Both Architectures)

**Requirements:**
- `cross` tool
- Docker running

**Build commands:**
```bash
# ARM64
cross build --release --target aarch64-linux-android

# x86_64 (emulator)
cross build --release --target x86_64-linux-android
```

---

## Build Artifacts

### Location

**Native builds:**
```
target/release/libgbln.rlib        # macOS ARM64 (host)
```

**Cross-compiled builds:**
```
target/{triple}/release/libgbln.rlib
```

**Examples:**
```
target/x86_64-unknown-freebsd/release/libgbln.rlib
target/aarch64-unknown-linux-gnu/release/libgbln.rlib
target/x86_64-pc-windows-gnu/release/libgbln.rlib
```

### Artifact Types

- **`.rlib`**: Rust library archive (static linking into Rust projects)
- **`.so`** / **`.dylib`** / **`.dll`**: Dynamic libraries (via C FFI, see #005A)
- **`.a`**: Static libraries (via C FFI, see #005A)

---

## Build Times

**Approximate times on Apple M1 Mac:**

| Platform | Method | First Build | Incremental |
|----------|--------|-------------|-------------|
| macOS ARM64 | native | 2s | <1s |
| macOS x86_64 | cargo | 7s | 2s |
| iOS ARM64 | cargo | 7s | 2s |
| FreeBSD x86_64 | cross | 15s | 5s |
| FreeBSD ARM64 | cross + nightly | 3m | 30s |
| Linux x86_64 | cross | 2s | <1s |
| Linux ARM64 | cross | 15s | 5s |
| Windows x86_64 | cross | 4m | 30s |
| Android ARM64 | cross | 50s | 10s |
| Android x86_64 | cross | 50s | 10s |

**Total (clean build all platforms): ~15 minutes**

---

## Troubleshooting

### Docker Not Running

**Symptom:**
```
Error: could not get os and arch
Cannot connect to the Docker daemon
```

**Solution:**
```bash
# macOS
open -a Docker && sleep 15

# Linux
sudo systemctl start docker

# Verify
docker ps
```

### FreeBSD ARM64 Build Fails

**Symptom:**
```
error[E0463]: can't find crate for `core`
```

**Solution:**
```bash
# Install nightly + rust-src
rustup toolchain install nightly
rustup component add rust-src --toolchain nightly

# Use nightly for build
cross +nightly build --release --target aarch64-unknown-freebsd
```

### Windows Build Fails (Missing stdlib.h)

**Symptom:**
```
fatal error: 'stdlib.h' file not found
```

**Solution:**
Use MinGW target instead of MSVC:
```bash
rustup target add x86_64-pc-windows-gnu
cross build --release --target x86_64-pc-windows-gnu
```

### Cross Image Pull Fails

**Symptom:**
```
Unable to find image 'ghcr.io/cross-rs/...'
```

**Solution:**
```bash
# Ensure Docker is running and has internet access
docker pull ghcr.io/cross-rs/aarch64-unknown-linux-gnu:main

# Retry build
cross build --release --target aarch64-unknown-linux-gnu
```

---

## CI/CD Integration

### GitHub Actions Matrix (Planned)

See `.claude/tickets/004C-gbln-rust-platform-support.md` for complete CI/CD workflow.

**Key points:**
- Native builds on GitHub-hosted runners (Linux, macOS, Windows)
- Cross-compilation for ARM platforms
- FreeBSD builds via self-hosted runners or cross-compilation
- Artifact upload to GitHub Releases

---

## Development Workflow

### Quick Development Cycle

```bash
cd core/rust

# Native build (fast)
cargo build --target aarch64-apple-darwin

# Run tests
cargo test

# Check specific platform
cross build --release --target x86_64-unknown-freebsd

# Format code
cargo fmt

# Lint
cargo clippy
```

### Full Release Build

```bash
# Build all platforms
./scripts/build-all.sh

# Verify artifacts
ls -lh target/*/release/libgbln.rlib

# Create release documentation
cat core/rust/BUILD_MATRIX.md
cat core/rust/ARTIFACTS.md
```

---

## Documentation

### Related Documents

- **BUILD_MATRIX.md**: `core/rust/BUILD_MATRIX.md` - Complete platform matrix with sizes
- **ARTIFACTS.md**: `core/rust/ARTIFACTS.md` - Artifact details and analysis
- **Cross.toml**: `core/rust/Cross.toml` - Cross-compilation configuration
- **Ticket #004C**: `.claude/tickets/004C-gbln-rust-platform-support.md` - Platform support details
- **Ticket #005A**: `.claude/tickets/005A-gbln-ffi-platform-matrix.md` - C FFI binary distribution

---

## Summary

**Build System Status:** ✅ Fully Operational

- **10/10 platforms** building successfully
- **Average artifact size:** 344 KB
- **Build time:** ~15 minutes for all platforms
- **Method:** Rust + Cargo + cross + Docker
- **Documentation:** Complete

**Next Steps:**
1. Set up CI/CD for automated builds
2. Create C FFI dynamic libraries (see #005A)
3. Package for distribution
4. Integrate with language bindings

---

**Last Updated:** 2025-11-26  
**Maintained by:** GBLN Project  
**Contact:** ask@vvoss.dev
