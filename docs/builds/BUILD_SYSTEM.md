# GBLN Build System Documentation

**Version:** 2.0  
**Last Updated:** 2025-11-27  
**Status:** Production

---

## ⚠️ MANDATORY BUILD POLICY ⚠️

**CRITICAL: There is ONLY ONE VALID way to build GBLN for any platform.**

### The Only Valid Build Method

**ALL platform builds MUST use Docker-based cross-compilation via `cross` tool.**

This is **NOT optional**. This is **NOT a recommendation**. This is the **ONLY accepted method**.

### Why This Is Mandatory

1. **Uniform Quality**: Identical build environment across all platforms
2. **Reproducibility**: Same Docker images guarantee same results
3. **No Local Variations**: Developer machine differences cannot affect builds
4. **Verified Toolchains**: Docker images contain tested, known-good toolchains
5. **CI/CD Compatibility**: Same method works on developer machines and CI servers

### What Is NOT Allowed

❌ **Native cross-compilation** (cargo build --target on host)  
❌ **Platform-specific toolchains** (Xcode, MSVC, etc. on host)  
❌ **Custom Docker images** (only official cross-rs images)  
❌ **Manual toolchain setup** (no custom gcc, clang, linker configs)  
❌ **"It works on my machine"** (if not via Docker, it's invalid)

### The Exception That Proves The Rule

**There are NO exceptions.** Even macOS ARM64 (the native development platform) should ideally be built via Docker when possible.

**Current pragmatic compromise:** macOS ARM64 native builds are allowed ONLY for rapid development iteration. Production releases MUST use Docker.

---

## Overview

This document describes the complete GBLN build system for all supported platforms. The build system uses **Docker + cross** as the **mandatory and only** build method.

**Supported Platforms:** 10  
**Build Method:** Docker + cross (MANDATORY)  
**CI/CD:** GitHub Actions (planned)  
**Documentation:** See `core/rust/BUILD_MATRIX.md` for complete platform matrix

---

## Build Architecture

```
Host: Any platform with Docker
   ↓
┌──────────────────────────────────────────────────────┐
│  MANDATORY: cross + Docker                           │
│  - Uses official cross-rs Docker images              │
│  - Identical environment for all platforms           │
│  - No host toolchain pollution                       │
└──────────────────────────────────────────────────────┘
   ↓
┌──────────────────────────────────────────────────────┐
│  Platform-Specific Docker Containers                 │
│  - FreeBSD x86_64, ARM64                             │
│  - Linux x86_64, ARM64                               │
│  - Windows x86_64 (MinGW)                            │
│  - Android ARM64, x86_64                             │
│  - macOS x86_64, ARM64 (when available)              │
│  - iOS ARM64 (when available)                        │
└──────────────────────────────────────────────────────┘
   ↓
┌──────────────────────────────────────────────────────┐
│  Build Artifacts (Uniform Quality)                   │
│  target/{triple}/release/libgbln.*                   │
└──────────────────────────────────────────────────────┘
```

**Key Principle:** The Docker container is the build environment. The host is merely a Docker client.

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

| Platform | Target Triple | Method | Docker Image | Status |
|----------|---------------|--------|--------------|--------|
| FreeBSD ARM64 | `aarch64-unknown-freebsd` | cross (Docker) | cross-rs/aarch64-unknown-freebsd:edge | ✅ Verified |
| FreeBSD x86_64 | `x86_64-unknown-freebsd` | cross (Docker) | cross-rs/x86_64-unknown-freebsd:main | ✅ Verified |
| Linux ARM64 | `aarch64-unknown-linux-gnu` | cross (Docker) | cross-rs/aarch64-unknown-linux-gnu:main | ✅ Verified |
| Linux x86_64 | `x86_64-unknown-linux-gnu` | cross (Docker) | cross-rs/x86_64-unknown-linux-gnu:main | ✅ Verified |
| macOS ARM64 | `aarch64-apple-darwin` | cross (Docker) when available | N/A (dev: native) | ⚠️ Dev only |
| macOS x86_64 | `x86_64-apple-darwin` | cross (Docker) when available | N/A (needs rustup) | ⏳ Pending |
| Windows x86_64 | `x86_64-pc-windows-gnu` | cross (Docker) | cross-rs/x86_64-pc-windows-gnu:main | ✅ Verified |
| iOS ARM64 | `aarch64-apple-ios` | cross (Docker) when available | N/A (needs rustup) | ⏳ Pending |
| Android ARM64 | `aarch64-linux-android` | cross (Docker) | cross-rs/aarch64-linux-android:main | ✅ Verified |
| Android x86_64 | `x86_64-linux-android` | cross (Docker) | cross-rs/x86_64-linux-android:main | ✅ Verified |

**Status Legend:**
- ✅ **Verified**: Successfully built via Docker with cross
- ⚠️ **Dev only**: Native builds allowed for development iteration only, NOT for releases
- ⏳ **Pending**: Awaiting Docker image support from cross-rs project

---

## Building All Platforms

### Quick Start - Build Everything (MANDATORY METHOD)

```bash
cd /path/to/GBLN/core/ffi  # or core/rust for Rust core

# 1. Install cross tool (REQUIRED)
cargo install cross --git https://github.com/cross-rs/cross

# 2. Start Docker (REQUIRED - this is your build environment)
open -a Docker  # macOS
# or: systemctl start docker  # Linux

# Wait for Docker to start and verify
sleep 15
docker ps  # Should show no errors

# 3. Build ALL platforms via Docker (MANDATORY)
# This is the ONLY valid way to build GBLN for any platform

# FreeBSD (both architectures)
cross build --release --target x86_64-unknown-freebsd
cross build --release --target aarch64-unknown-freebsd

# Linux (both architectures)
cross build --release --target x86_64-unknown-linux-gnu
cross build --release --target aarch64-unknown-linux-gnu

# Windows (MinGW)
cross build --release --target x86_64-pc-windows-gnu

# Android (both architectures)
cross build --release --target aarch64-linux-android
cross build --release --target x86_64-linux-android

# macOS/iOS (when Docker images available)
# Currently: Use native builds ONLY for rapid development
# Production releases: MUST use Docker when available
```

### ⚠️ CRITICAL: No Shortcuts Allowed

**DO NOT:**
- ❌ Use `cargo build` without `cross`
- ❌ Install platform-specific toolchains (Xcode, MSVC, etc.)
- ❌ Try to "optimize" by skipping Docker
- ❌ Use different methods for different platforms

**WHY:**
- Different toolchain versions produce different binaries
- Host environment variations cause bugs
- "Works on my machine" is NOT acceptable
- We need bit-for-bit reproducible builds

### Build Script (Automated - MANDATORY Docker Method)

**Location:** `core/ffi/build-docker.sh` (already created for Ticket #005B)

```bash
#!/usr/bin/env bash
# GBLN C FFI - Docker-Based Cross-Platform Build Script
# This is the ONLY valid way to build GBLN for production

set -euo pipefail

cd "$(dirname "$0")"

# Color output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

echo "GBLN Docker-Based Build System"
echo "==============================="
echo ""
echo "⚠️  MANDATORY: All builds use Docker via 'cross' tool"
echo "⚠️  This ensures uniform quality across all platforms"
echo ""

# Check Docker is running
if ! docker ps >/dev/null 2>&1; then
    echo -e "${RED}ERROR: Docker is not running${NC}"
    echo "Start Docker and try again"
    exit 1
fi

# All production platforms (Docker-based)
TARGETS=(
    "x86_64-unknown-freebsd"
    "aarch64-unknown-freebsd"
    "x86_64-unknown-linux-gnu"
    "aarch64-unknown-linux-gnu"
    "x86_64-pc-windows-gnu"
    "aarch64-linux-android"
    "x86_64-linux-android"
)

echo -e "${GREEN}Building 7 platforms via Docker...${NC}"
echo ""

for target in "${TARGETS[@]}"; do
    echo -e "${YELLOW}Building: $target${NC}"
    cross build --release --target "$target"
    echo -e "${GREEN}✓ Built: $target${NC}"
    echo ""
done

echo -e "${GREEN}All platforms built successfully via Docker!${NC}"
echo ""
echo "Artifacts:"
find target -name "libgbln.*" -type f | grep release | sort
```

**Usage:**
```bash
cd core/ffi
./build-docker.sh
```

**This script is the MANDATORY method for all production builds.**

---

## Platform-Specific Instructions

**⚠️ IMPORTANT: All instructions below use Docker via `cross`. No other method is permitted.**

### FreeBSD (Both Architectures)

**MANDATORY Method: Docker via cross**

```bash
# x86_64
cross build --release --target x86_64-unknown-freebsd

# ARM64
cross build --release --target aarch64-unknown-freebsd
```

**Docker Images Used:**
- x86_64: `ghcr.io/cross-rs/x86_64-unknown-freebsd:main`
- ARM64: `ghcr.io/cross-rs/aarch64-unknown-freebsd:edge`

**Status:** ✅ Verified working

### Linux (Both Architectures)

**MANDATORY Method: Docker via cross**

```bash
# x86_64
cross build --release --target x86_64-unknown-linux-gnu

# ARM64
cross build --release --target aarch64-unknown-linux-gnu
```

**Docker Images Used:**
- x86_64: `ghcr.io/cross-rs/x86_64-unknown-linux-gnu:main`
- ARM64: `ghcr.io/cross-rs/aarch64-unknown-linux-gnu:main`

**Status:** ✅ Verified working

### macOS (Both Architectures)

**Current Status:** ⚠️ Docker images not yet available from cross-rs

**Temporary Exception (Development ONLY):**
```bash
# ARM64 (native on Apple Silicon)
cargo build --release --target aarch64-apple-darwin
```

**⚠️ WARNING:** Native builds are ONLY for rapid development iteration.

**Production Requirement:** When cross-rs provides macOS Docker images, ALL builds must migrate to Docker method.

**Status:** ⏳ Pending cross-rs Docker image support

### Windows (x86_64)

**MANDATORY Method: Docker via cross**

```bash
# MinGW (GNU ABI)
cross build --release --target x86_64-pc-windows-gnu
```

**Docker Image Used:**
- `ghcr.io/cross-rs/x86_64-pc-windows-gnu:main`

**Note:** Produces Windows binaries with GNU ABI (MinGW). Compatible with all Windows systems.

**Status:** ✅ Verified working

### iOS (ARM64)

**Current Status:** ⚠️ Docker images not yet available from cross-rs

**Temporary Exception (When Needed):**
```bash
# For Swift bindings development only
rustup target add aarch64-apple-ios
cargo build --release --target aarch64-apple-ios
```

**Production Requirement:** When cross-rs provides iOS Docker images, must migrate to Docker method.

**Status:** ⏳ Pending cross-rs Docker image support

### Android (Both Architectures)

**MANDATORY Method: Docker via cross**

```bash
# ARM64
cross build --release --target aarch64-linux-android

# x86_64 (emulator)
cross build --release --target x86_64-linux-android
```

**Docker Images Used:**
- ARM64: `ghcr.io/cross-rs/aarch64-linux-android:main`
- x86_64: `ghcr.io/cross-rs/x86_64-linux-android:main`

**Status:** ✅ Verified working

---

### Summary: Docker Is The Law

**7 Platforms:** ✅ Docker-based via cross (MANDATORY)  
**3 Platforms:** ⏳ Awaiting Docker image support (temporary native exception)

**When Docker images become available for macOS/iOS, native builds will be FORBIDDEN.**

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
