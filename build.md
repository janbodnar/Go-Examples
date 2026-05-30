# Go Build System & Tooling

## Introduction

The Go toolchain follows a *batteries-included* philosophy: a single `go` binary
covers package management, compilation, formatting, vetting, code generation, and
more. There is no need for a separate build tool or dependency manager for the
vast majority of projects.

Compilation is fast by design. The language spec forbids circular imports, header
files are not used, and the compiler is engineered for speed. Incremental builds
cache compiled packages under `$GOPATH/pkg/mod/cache` (or the module cache), so
only changed packages are recompiled.

The `go` command acts as a unified front-end:

```text
go <command> [arguments]
```

Common sub-commands include `run`, `build`, `fmt`, `vet`, `generate`, `tool`, and
`mod`. All of them understand Go modules and respect the `go.mod` / `go.sum` files
at the root of a module.

---

## Building and Running

### `go run`

`go run` compiles and immediately executes one or more Go source files without
producing a persistent binary on disk. It is ideal for quick scripts and
development iteration.

```bash
# Run a single file
go run main.go

# Run multiple files that share the same package main
go run main.go helpers.go

# Run the package in the current directory (module-aware)
go run .

# Run a specific package by import path
go run github.com/user/repo/cmd/myapp
```

Under modules, `go run .` compiles the `main` package in the current directory
using the module graph defined in `go.mod`. The resulting binary is placed in a
temporary directory and removed after execution.

### `go build`

`go build` compiles packages and their dependencies and writes the resulting
binary to disk.

```bash
# Build the package in the current directory
go build .

# Specify the output file name
go build -o myapp .

# Build a specific package by import path
go build github.com/user/repo/cmd/myapp

# Build all packages in a module (useful for checking compile errors)
go build ./...
```

```text
$ go build -o myapp .
$ ls -lh myapp
-rwxr-xr-x  1 user group 6.2M May 30 10:00 myapp
```

When building a library package (not `package main`), no binary is produced; the
compiler checks for errors and caches the result.

### Essential Build Flags

#### `-race` — Data Race Detector

The race detector instruments memory accesses at compile time and reports
concurrent access violations at runtime.

```bash
go build -race -o myapp_race .
```

Enable it during development and in CI but not in production binaries: it adds
roughly 5–10× overhead in CPU and memory.

#### `-tags` — Build Tags

Build tags control which files are included in a compilation. A file is included
only when its tags are satisfied.

```go
//go:build production

package config

const Debug = false
```

```bash
# Build with the production tag
go build -tags production -o myapp .

# Combine multiple tags
go build -tags "production linux" -o myapp .
```

#### `-ldflags` — Linker Flags

`-ldflags` passes flags to the linker. The most common use is `-X` to stamp
version information into a binary at build time without modifying source code.

```go
package main

import "fmt"

var (
	Version   = "dev"
	BuildTime = "unknown"
)

func main() {
	fmt.Printf("version=%s built=%s\n", Version, BuildTime)
}
```

```bash
go build \
  -ldflags "-X main.Version=1.2.3 -X 'main.BuildTime=$(date -u +%Y-%m-%dT%H:%M:%SZ)'" \
  -o myapp .
```

```text
$ ./myapp
version=1.2.3 built=2026-05-30T10:00:00Z
```

Other useful linker flags:

| Flag | Effect |
|------|--------|
| `-s` | Strip symbol table |
| `-w` | Strip DWARF debug info |
| `-extldflags "-static"` | Produce a fully static binary (requires CGO disabled) |

---

## Code Quality & Maintenance

### `go fmt`

`go fmt` formats Go source files according to the canonical style defined by the
Go team. It is non-configurable by design, eliminating style debates.

```bash
# Format files in the current package
go fmt .

# Format all packages recursively
go fmt ./...

# Preview changes without writing (uses gofmt directly)
gofmt -d .
```

`gofmt` is the underlying tool; `go fmt` is a thin wrapper that accepts package
patterns. Editors typically call `gofmt` (or the LSP equivalent) on save.

### `go vet`

`go vet` runs a suite of static analysis checks that catch suspicious constructs
the compiler does not reject.

```bash
go vet .
go vet ./...
```

```text
$ go vet ./...
./main.go:14:2: call has possible formatting directive %v
./server.go:42:9: unreachable code
```

Examples of issues `go vet` detects:

- Mismatched `Printf` format verbs and arguments.
- Copying a value that contains a `sync.Mutex`.
- Unreachable code after a `return` or `panic`.
- Incorrect use of `//go:build` constraints.
- Suspicious composite literal usages.

`go vet` is zero-configuration and is implicitly run by `go test` before
executing tests.

### `go fix`

`go fix` rewrites Go source code to use newer API idioms after a language or
standard library change. It applies automated rewrites defined by the Go team for
specific version transitions.

```bash
go fix ./...
```

```text
$ go fix ./...
mypackage/old_api.go: ioutil.ReadAll -> io.ReadAll
```

`go fix` is used infrequently—primarily when migrating code between major Go
versions that introduced API changes (for example, the `ioutil` deprecation in
Go 1.16). Always review and commit the diff after running it.

---

## Code Generation

### `go generate`

`go generate` scans Go source files for lines of the form:

```go
//go:generate command [arguments]
```

and executes each command as a shell command. It does **not** run automatically
during `go build`; it must be invoked explicitly.

```bash
go generate ./...
```

#### Example: Generating String Methods with `stringer`

Given a package with an integer enumeration:

```go
package color

//go:generate stringer -type=Color

type Color int

const (
	Red Color = iota
	Green
	Blue
)
```

Running `go generate` invokes `stringer`, which creates `color_string.go`
containing a `String() string` method for `Color`.

```bash
# Install stringer once
go install golang.org/x/tools/cmd/stringer@latest

# Generate in the color package
go generate ./color/...
```

```text
$ go generate ./color/...
# Writes: color/color_string.go
```

#### Example: Running a Custom Script

```go
//go:generate go run ./gen/main.go -output generated.go
```

Best practices:

- Commit generated files to the repository so consumers do not need to re-run
  generation.
- Document the tools required by `//go:generate` directives in a `README` or a
  `tools.go` file using blank imports.

---

## Cross-Compilation

Go compiles to native machine code and ships with standard library sources for
every supported platform. This makes cross-compilation trivially easy: set two
environment variables and run `go build`.

### `GOOS` and `GOARCH`

`GOOS` specifies the target operating system; `GOARCH` specifies the target
architecture.

```bash
# Build a Linux amd64 binary from any host
GOOS=linux  GOARCH=amd64   go build -o myapp-linux-amd64   .

# Build a Windows amd64 binary
GOOS=windows GOARCH=amd64  go build -o myapp-windows-amd64.exe .

# Build for Apple Silicon (macOS arm64)
GOOS=darwin  GOARCH=arm64  go build -o myapp-darwin-arm64   .

# Build for ARM Linux (e.g., Raspberry Pi)
GOOS=linux   GOARCH=arm    GOARM=7 go build -o myapp-linux-arm .
```

List all supported target combinations:

```bash
go tool dist list
```

```text
aix/ppc64
android/386
android/amd64
android/arm
android/arm64
darwin/amd64
darwin/arm64
freebsd/amd64
linux/amd64
linux/arm
linux/arm64
windows/amd64
windows/arm64
...
```

### `CGO_ENABLED=0` — Fully Static Binaries

By default, some standard library packages (notably `net` and `os/user`) use
`cgo` to call into the host C library. Setting `CGO_ENABLED=0` forces pure Go
implementations, producing a self-contained static binary with no external shared
library dependencies.

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
  go build -ldflags "-s -w" -o myapp-static .
```

This is particularly useful when deploying to minimal container images (e.g.,
`scratch`) or environments without a C runtime.

```text
$ file myapp-static
myapp-static: ELF 64-bit LSB executable, x86-64, statically linked, stripped
```

---

## Profiling & Diagnostics

### `go tool pprof`

`pprof` is Go's built-in profiling tool. It reads profiles produced by the
`runtime/pprof` package or the `net/http/pprof` HTTP handler.

#### Generating Profiles

**Programmatic (batch programs):**

```go
package main

import (
	"os"
	"runtime/pprof"
)

func main() {
	f, _ := os.Create("cpu.prof")
	defer f.Close()
	pprof.StartCPUProfile(f)
	defer pprof.StopCPUProfile()

	// ... application logic ...

	mf, _ := os.Create("mem.prof")
	defer mf.Close()
	pprof.WriteHeapProfile(mf)
}
```

**HTTP server (long-running services):**

```go
import _ "net/http/pprof" // registers handlers on DefaultServeMux
import "net/http"

func main() {
	go http.ListenAndServe("localhost:6060", nil)
	// ... application logic ...
}
```

Fetch a 30-second CPU profile from a running server:

```bash
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
```

#### Reading Profiles with `go tool pprof`

```bash
# Open an interactive session for a CPU profile
go tool pprof cpu.prof

# Open an interactive session for a heap profile
go tool pprof mem.prof

# Open the web UI (requires Graphviz)
go tool pprof -http=:8080 cpu.prof
```

Inside the interactive REPL:

```text
(pprof) top
Showing nodes accounting for 1.23s, 98.40% of 1.25s total
      flat  flat%   sum%        cum   cum%
     0.80s 64.00% 64.00%      0.80s 64.00%  runtime.mallocgc
     0.30s 24.00% 88.00%      0.30s 24.00%  encoding/json.Marshal
     0.13s 10.40% 98.40%      1.23s 98.40%  main.processRecords

(pprof) list main.processRecords
(pprof) web
```

Useful `pprof` commands:

| Command | Description |
|---------|-------------|
| `top [n]` | Show top N functions by self CPU |
| `list <func>` | Annotate source lines for a function |
| `web` | Open a call graph in the browser |
| `pdf` | Save call graph as PDF |
| `traces` | Show individual goroutine stack traces |

---

## Reproducible Builds

A Go build is reproducible when the same source code and dependencies always
produce a bit-for-bit identical binary, regardless of which machine or user
performs the build.

### `-trimpath`

By default, the Go compiler embeds absolute file system paths in binaries (for
stack traces and `runtime.Caller`). This makes two builds on different machines
differ even when the source is identical.

The `-trimpath` flag removes all local machine-specific path prefixes:

```bash
go build -trimpath -o myapp .
```

```text
# Without -trimpath, a panic might show:
# goroutine 1 [running]:
# main.main()
#         /home/alice/projects/myapp/main.go:12 +0x58

# With -trimpath:
# goroutine 1 [running]:
# main.main()
#         github.com/alice/myapp/main.go:12 +0x58
```

### `go.sum` and Module Proxies

Reproducible dependency resolution relies on two mechanisms:

**`go.sum`** — A checksums file that records the cryptographic hash of every
module version used by the build. The `go` command refuses to use a module whose
downloaded content does not match the recorded hash.

```text
github.com/some/dep v1.2.3 h1:abc123...==
github.com/some/dep v1.2.3/go.mod h1:def456...==
```

**Module Proxy (`GOPROXY`)** — The Go toolchain fetches modules through a proxy
(defaulting to `https://proxy.golang.org`). The proxy caches immutable module
zip files, ensuring the same version always resolves to the same content.

```bash
# Use the default public proxy
GOPROXY=https://proxy.golang.org,direct go build .

# Use a private proxy with fallback to direct
GOPROXY=https://goproxy.company.internal,direct go build .

# Disable the proxy entirely (fetch directly from VCS)
GOPROXY=direct go build .
```

**`GONOSUMCHECK` and `GONOSUMDB`** allow bypassing checksum verification for
private modules that are not indexed in the public checksum database
(`sum.golang.org`).

**Summary of reproducible build flags:**

```bash
CGO_ENABLED=0 \
  go build \
  -trimpath \
  -ldflags "-s -w" \
  -o myapp \
  .
```

This command produces a stripped, statically linked binary with no embedded host
paths—suitable for reproducible, portable distribution.
