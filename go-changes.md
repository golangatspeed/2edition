# Go 1.19 to 1.25 Feature Changes

## Overview
This document outlines the major features and changes introduced in Go versions 1.20 through 1.25 that need to be incorporated into the second edition.

## Go 1.20 (February 2023)
- **Profile-Guided Optimization (PGO)** - Preview feature for performance optimization
- **Comparable types** - Enhanced support for comparable type constraints
- **Cover profiling improvements** - Better code coverage analysis
- **New errors.Join function** - Ability to combine multiple errors

## Go 1.21 (August 2023)
- **Profile-Guided Optimization (PGO)** - Now stable and ready for general use (2-7% performance improvements)
- **New built-in functions**: `min`, `max`, and `clear`
- **Backwards compatibility improvements** - Enhanced GODEBUG support
- **WebAssembly enhancements** - New `go:wasmimport` directive
- **Structured logging** - New `slog` package for structured logging
- **Maps and slices packages** - New utility functions for common operations

## Go 1.22 (February 2024)
- **math/rand/v2 package** - First "v2" package in standard library with improved API
- **For loop variable semantics** - Fixed per-iteration variable capture
- **TLS 1.2 minimum** - Updated default minimum TLS version
- **Improved ARM support** - GOARM environment variable enhancements
- **Enhanced range over integers** - Direct integer iteration in for-range loops

## Go 1.23 (August 2024)
- **Range over functions** - Iterator functions as range expressions
- **Generic type aliases (preview)** - GOEXPERIMENT=aliastypeparams
- **New GOARM64 environment variable** - ARM64 architecture targeting
- **OpenBSD RISC-V support** - Experimental platform support
- **Enhanced timer implementation** - Improved runtime performance

## Go 1.24 (February 2025)
- **Generic type aliases** - Now fully supported (stable)
- **Tool directives in go.mod** - Track executable dependencies
- **os.Root type** - Filesystem operations within specific directories
- **WebAssembly exports** - `go:wasmexport` compiler directive
- **Testing improvements** - Enhanced testing tools and capabilities

## Go 1.25 (August 2025)
- **testing/synctest package** - Now stable (was experimental in 1.24)
- **json/v2 package** - Major update with breaking changes
- **FlightRecorder** - Lightweight execution tracing
- **WaitGroup.Go method** - Convenient goroutine creation and counting
- **Test attributes** - T.Attr, B.Attr, F.Attr methods for test logging
- **macOS 12+ requirement** - Updated minimum macOS version

## Integration Analysis

### Features that fit existing chapters:
1. **Chapter 2 (CLI)** - Tool directives, GOARM64, PGO flags
2. **Chapter 6 (Variables/Constants)** - Built-in functions (min, max, clear)
3. **Chapter 7 (Data Types)** - Generic type aliases, math/rand/v2
4. **Chapter 8 (Program Flow)** - Range over functions, enhanced for loops
5. **Chapter 8 (Error Handling)** - errors.Join function, structured logging
6. **Chapter 9 (Functions)** - Iterator functions, PGO considerations
7. **Chapter 10 (Concurrency)** - WaitGroup.Go, testing/synctest
8. **Chapter 11 (QA)** - Test attributes, FlightRecorder, enhanced testing

### Features needing new sections/chapters:
1. **Performance Optimization** - Could be new chapter or major section
   - Profile-Guided Optimization (PGO)
   - FlightRecorder for debugging
   - Performance analysis techniques

2. **Modern JSON Handling** - Significant enough for dedicated section
   - json/v2 package and migration
   - Breaking changes and new patterns

3. **WebAssembly Development** - Could warrant dedicated section
   - go:wasmimport and go:wasmexport
   - WASM-specific development patterns

4. **Advanced Standard Library** - New packages overview
   - slog for structured logging
   - maps and slices utility packages
   - os.Root for secure filesystem operations

## Recommendation
**Hybrid approach**: Integrate most features into existing chapters while adding 1-2 new chapters for substantial new areas (Performance Optimization, WebAssembly).