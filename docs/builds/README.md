# GBLN Build Infrastructure

This directory contains documentation for building GBLN on all supported platforms.

## Documentation

**[BUILD_SYSTEM.md](BUILD_SYSTEM.md)** - Complete build system documentation

## Overview

GBLN supports **10 platforms** via Rust + Cargo + cross-compilation:

| Platform | Architecture | Status |
|----------|--------------|--------|
| FreeBSD | x86_64, ARM64 | ✅ |
| Linux | x86_64, ARM64 | ✅ |
| macOS | x86_64, ARM64 | ✅ |
| Windows | x86_64 | ✅ |
| iOS | ARM64 | ✅ |
| Android | ARM64, x86_64 | ✅ |

## Quick Start

```bash
# Install prerequisites
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
cargo install cross --git https://github.com/cross-rs/cross

# Build all platforms
cd core/rust
./scripts/build-all.sh
```

For detailed instructions, platform-specific guides, and troubleshooting, see **[BUILD_SYSTEM.md](BUILD_SYSTEM.md)**.

## Build Method

All platforms are built via **cross-compilation** using:
- **Rust + Cargo** - Native and cross-arch builds
- **cross** - Docker-based cross-compilation for other platforms
- **Docker** - Provides platform-specific build environments

No VMs required. All builds work from any host with Rust + Docker.

## Related Documentation

- **`core/rust/BUILD_MATRIX.md`** - Platform matrix with sizes and methods
- **`core/rust/ARTIFACTS.md`** - Artifact details and analysis
- **`core/rust/Cross.toml`** - Cross-compilation configuration
- **`.claude/tickets/004C-gbln-rust-platform-support.md`** - Implementation details

---

**Last Updated:** 2025-11-26  
**Build System Status:** ✅ Fully Operational (10/10 platforms)
