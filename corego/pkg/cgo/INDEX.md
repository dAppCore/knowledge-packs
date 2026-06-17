---
type: Package Index
title: go-cgo Package Index
description: Complete index of go-cgo package components and API surface
module: dappco.re/go/cgo
---

# go-cgo — Package Index

> **Repository:** `core/go-cgo`
> **Module:** `dappco.re/go/cgo`
> **Type:** Library (CGo)
> **Status:** Production
> **Lines:** ~2,161 (source)

---

## 📚 Quick Links

- **[README.md](./README.md)** — Complete package documentation
- **[RFC Specification](file:///Users/snider/Code/meowmix/plans/code/core/go/cgo/RFC.md)** — Technical specification
- **[AGENTS.md](file:///Users/snider/Code/core/go-cgo/AGENTS.md)** — Agent guidance
- **[CLAUDE.md](file:///Users/snider/Code/core/go-cgo/CLAUDE.md)** — Implementation details

---

## 🗂️ File Structure

### Source Files (6)

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `doc.go` | 23 | Package documentation and usage examples | ✅ Complete |
| `buffer.go` | 161 | Buffer type: C-backed byte allocation with panic guards | ✅ Complete |
| `scope.go` | 126 | Scope type: grouped C allocations with bulk cleanup | ✅ Complete |
| `string_conversion.go` | 1,559 | C type conversions, string conversion, errno mapping | ✅ Complete |
| `call.go` | 200+ | C function pointer dispatcher (0-18 args) | ✅ Complete |
| `cstring.go` | 100+ | C string utilities | ✅ Complete |
| `call_test_support.go` | 145 | C test harness with function call stubs | ✅ Complete |

### Test Files (8)

| File | Purpose | Coverage |
|------|---------|----------|
| `buffer_test.go` | Buffer tests | ✅ Good/Bad/Ugly |
| `buffer_example_test.go` | Buffer examples | ✅ Examples |
| `scope_test.go` | Scope tests | ✅ Good/Bad/Ugly |
| `scope_example_test.go` | Scope examples | ✅ Examples |
| `call_test.go` | Call/Conversion tests | ✅ Good/Bad/Ugly |
| `call_example_test.go` | Call examples | ✅ Examples |
| `string_conversion_test.go` | String conversion tests | ✅ Good/Bad/Ugly |
| `assert_test.go` | Assertion helpers | ✅ Complete |
| `alloc_budget_test.go` | Allocation budget tests | ✅ Complete |

### CLI Test Files

| File | Purpose |
|------|---------|
| `tests/cli/cgo/main.go` | AX-10 binary scenario for real cgo calls |
| `tests/cli/cgo/main_cli_test.go` | CLI tests for cgo binary |

---

## 🔧 Public API Surface

### Buffer Type

| Symbol | Type | Description |
|--------|------|-------------|
| `Buffer` | `struct` | C-backed byte memory with panic guards |
| `NewBuffer` | `func(size int) *Buffer` | Allocate C-backed buffer |
| `(*Buffer).Free` | `func()` | Release memory, panic on double-free |
| `(*Buffer).Close` | `func() error` | io.Closer interface |
| `(*Buffer).CopyFrom` | `func(src []byte) int` | Copy bytes into buffer |
| `(*Buffer).Bytes` | `func() []byte` | Get Go slice |
| `(*Buffer).Ptr` | `func() unsafe.Pointer` | Get raw C pointer |
| `(*Buffer).Len` | `func() int` | Get allocated size |
| `(*Buffer).IsFreed` | `func() bool` | Check if freed |

### Scope Type

| Symbol | Type | Description |
|--------|------|-------------|
| `Scope` | `struct` | Grouped C allocations with bulk cleanup |
| `NewScope` | `func() *Scope` | Create new scope with finalizer |
| `(*Scope).FreeAll` | `func()` | Release all allocations |
| `(*Scope).Close` | `func() error` | io.Closer interface |
| `(*Scope).Buffer` | `func(size int) *Buffer` | Allocate managed buffer |
| `(*Scope).CString` | `func(value string) *C.char` | Allocate managed C string |
| `(*Scope).IsFreed` | `func() bool` | Check if freed |

### Type Conversions

| Symbol | Type | Description |
|--------|------|-------------|
| `SizeT` | `func(value int) C.size_t` | Convert Go int to C size_t |
| `Int` | `func(value int) C.int` | Convert Go int to C int |
| `Errno` | `func(rc C.int) error` | Convert C errno to Go error |
| `WithErrno` | `func(fn func() C.int) (int, error)` | Call C function, return errno as error |

### String Conversion

| Symbol | Type | Description |
|--------|------|-------------|
| `GoString` | `func(cs *C.char) string` | Convert C string to Go string |
| `CString` | `func(s string) *C.char` | Convert Go string to C string |
| `Free` | `func(ptr unsafe.Pointer)` | Release C-allocated memory |

### Function Calls

| Symbol | Type | Description |
|--------|------|-------------|
| `Call` | `func(function unsafe.Pointer, args ...interface{}) error` | Invoke C function pointer |

---

## 📊 Statistics

### Code Metrics

```
Total source files:        6
Total test files:         8
Total example files:      3
Total public symbols:     25+
Total lines (source):    ~2,161
Total lines (tests):     ~3,000+
```

### Test Coverage

| Category | Count | Status |
|----------|-------|--------|
| Good tests (happy path) | 30+ | ✅ Complete |
| Bad tests (expected errors) | 20+ | ✅ Complete |
| Ugly tests (edge cases) | 15+ | ✅ Complete |
| Example tests | 10+ | ✅ Complete |

### Complexity

| Component | Methods | Status |
|-----------|---------|--------|
| Buffer | 7 | ✅ Well-documented |
| Scope | 6 | ✅ Well-documented |
| Type Conversions | 4 | ✅ Well-documented |
| String Conversion | 3 | ✅ Well-documented |
| Function Calls | 1 | ✅ Well-documented |

---

## 🏷️ Tags & Categories

### Technology Tags

- `cgo` — Primary tag
- `c-interop` — C/Go interoperability
- `bindings` — C library bindings
- `ffi` — Foreign Function Interface
- `memory-management` — Memory allocation and cleanup
- `unsafe` — Uses unsafe package

### Usage Tags

- `buffer` — C-backed byte buffers
- `scope` — Grouped allocation management
- `conversion` — Type and string conversion
- `call` — Function pointer dispatch

---

## 🔗 Dependencies

### Internal Dependencies

| Package | Purpose | Import Path |
|---------|---------|-------------|
| Core framework | Result pattern, error handling | `dappco.re/go` |
| syscall | Errno type | `syscall` |
| unsafe | Pointer manipulation | `unsafe` |

### External Dependencies

None — Pure Go with CGO support.

Requires working C toolchain with `CGO_ENABLED=1`.

---

## 📁 Consumers

go-cgo is used by these Core packages:

| Package | Module | C Dependency | Usage |
|---------|--------|--------------|-------|
| `go-blockchain` | `dappco.re/go/blockchain` | Various | ECDSA curve operations via C |
| `go-mlx` | `dappco.re/go/mlx` | MLX | Apple MLX inference via C |
| `go-rocm` | `dappco.re/go/rocm` | ROCm | AMD ROCm kernel calls via C |
| `go-inference` | `dappco.re/go/inference` | Multiple | LLM inference bindings |

---

## 🎯 Usage Patterns

### Pattern 1: Scope-Based (Recommended)

```go
scope := corecgo.NewScope()
defer scope.FreeAll()

// All allocations are tracked
buf1 := scope.Buffer(32)
buf2 := scope.Buffer(64)
str1 := scope.CString("hello")

// Single cleanup call
```

### Pattern 2: Manual Management

```go
buffer := corecgo.NewBuffer(1024)
defer buffer.Free()

cStr := corecgo.CString("hello")
defer corecgo.Free(unsafe.Pointer(cStr))
```

### Pattern 3: C Function Calls

```go
err := corecgo.Call(
    unsafe.Pointer(C.my_function),
    buffer.Ptr(),
    corecgo.SizeT(len(data)),
    corecgo.Int(42),
)
```

### Pattern 4: Error Handling

```go
rc := C.my_function()
if err := corecgo.Errno(rc); err != nil {
    return err
}
```

### Pattern 5: WithErrno Closure

```go
result, err := corecgo.WithErrno(func() C.int {
    return C.my_function(args...)
})
```

---

## 📋 Compliance Summary

### File Organization

| File | Owns Tests | Status |
|------|------------|--------|
| `buffer.go` | `buffer_test.go`, `buffer_example_test.go` | ✅ Compliant |
| `scope.go` | `scope_test.go`, `scope_example_test.go` | ✅ Compliant |
| `string_conversion.go` | `string_conversion_test.go`, `call_test.go` | ✅ Compliant |
| `call.go` | `call_test.go`, `call_example_test.go` | ✅ Compliant |

### Import Restrictions

✅ **No banned imports:**
- `fmt` → Use `core.Println`, `core.Printf`
- `errors` → Use `core.E`
- `strings` → Use `core.String`
- `path`, `path/filepath` → Use `core.Path`
- `os`, `os/exec` → Use `core.IO`
- `io/ioutil` → Use `core.IO`
- `log` → Use `core.Log`
- `encoding/json` → Use `core.JSON`
- `bytes` → Use `core.Bytes`

### Test Organization

✅ **File-aware triplets:**
- Each production file has matching `_test.go` and `_example_test.go`
- Tests use `Test<File>_<Symbol>_{Good,Bad,Ugly}` naming
- Examples use Go's runnable `// Output:` form

---

## 📝 Maintenance Information

- **Author**: Mistral Vibe (Purberus <purberus@lthn.ai>)
- **Created**: 2026-06-17T23:45:00Z
- **Last Updated**: 2026-06-17T23:45:00Z
- **Version**: 1.0.0
- **Licence**: EUPL-1.2
- **Repository**: `forge.lthn.sh/core/go-cgo`

---

## 🔗 Related Documentation

- [CoreGo Framework](../../README.md) — Parent knowledge pack
- [go-mlx Package](../mlx/README.md) — Uses go-cgo for MLX inference
- [go-rocm Package](../rocm/README.md) — Uses go-cgo for ROCm
- [go-blockchain Package](../blockchain/README.md) — Uses go-cgo for crypto
- [go-inference Package](../inference/README.md) — Uses go-cgo for inference

---

*Package index generated: 2026-06-17T23:45:00Z*
