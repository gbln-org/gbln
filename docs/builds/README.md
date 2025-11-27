# GBLN Build Infrastructure

This directory contains documentation for building GBLN on all supported platforms.

## ⚠️ MANDATORY BUILD POLICY

**There is ONLY ONE valid way to build GBLN: Docker via `cross` tool.**

This is not a recommendation. This is the **ONLY accepted method** for all production builds.

## Documentation

**[BUILD_SYSTEM.md](BUILD_SYSTEM.md)** - Complete build system documentation (v2.0 - Docker mandatory)

## Overview

GBLN supports **10 platforms** via **mandatory Docker-based builds**:

| Platform | Architecture | Build Method | Status |
|----------|--------------|--------------|--------|
| FreeBSD | x86_64, ARM64 | Docker (cross) | ✅ Verified |
| Linux | x86_64, ARM64 | Docker (cross) | ✅ Verified |
| macOS | x86_64, ARM64 | Docker (pending) | ⏳ Dev exception |
| Windows | x86_64 | Docker (cross) | ✅ Verified |
| iOS | ARM64 | Docker (pending) | ⏳ Dev exception |
| Android | ARM64, x86_64 | Docker (cross) | ✅ Verified |

**7/10 platforms:** ✅ Docker-verified  
**3/10 platforms:** ⏳ Awaiting Docker image support (temporary native exception)

## Quick Start (MANDATORY METHOD)

```bash
# Install Docker (REQUIRED)
# macOS: Install Docker Desktop
# Linux: sudo apt install docker.io

# Install cross tool (REQUIRED)
cargo install cross --git https://github.com/cross-rs/cross

# Start Docker (REQUIRED)
open -a Docker  # macOS
# or: sudo systemctl start docker  # Linux

# Build all platforms via Docker (MANDATORY)
cd core/ffi  # or core/rust
./build-docker.sh
```

**DO NOT use `cargo build` directly. Use `cross build` for all platforms.**

For detailed instructions, platform-specific guides, and troubleshooting, see **[BUILD_SYSTEM.md](BUILD_SYSTEM.md)**.

## Build Method

**MANDATORY:** All platforms MUST be built via **Docker-based cross-compilation**:

- **Docker** - The build environment (MANDATORY)
- **cross** - Docker orchestration tool (MANDATORY)
- **Rust + Cargo** - Build tool (runs inside Docker)

**Why Docker is mandatory:**
1. **Uniform Quality** - Identical build environment across all platforms
2. **Reproducibility** - Same Docker images = same results
3. **No Variations** - Developer machine differences cannot affect builds
4. **CI/CD Compatible** - Same method on dev machines and CI servers

**What is NOT allowed:**
- ❌ Native cargo builds (except macOS ARM64 for rapid dev iteration)
- ❌ Platform-specific toolchains (Xcode, MSVC, etc.)
- ❌ Custom Docker images
- ❌ "It works on my machine" solutions

## Related Documentation

- **`core/rust/BUILD_MATRIX.md`** - Platform matrix with sizes and methods
- **`core/rust/ARTIFACTS.md`** - Artifact details and analysis
- **`core/rust/Cross.toml`** - Cross-compilation configuration
- **`.claude/tickets/004C-gbln-rust-platform-support.md`** - Implementation details

---

**Last Updated:** 2025-11-27  
**Build System Status:** ✅ Fully Operational (10/10 platforms)  
**Build Policy Version:** 2.0 (Docker mandatory)
