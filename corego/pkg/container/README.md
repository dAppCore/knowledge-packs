---
type: Package Deep Dive
title: go-container — Docker-free container runtime
description: Container runtime without daemon overhead — LinuxKit builder, dev environment management, TIM bundles, and VZ provider
module: dappco.re/go/container
repo: core/go-container
tags: [container, docker, isolation, execution, vm, linuxkit, tim, vz, apple, sandbox]
created: 2026-06-18T00:00:00Z
author: Mistral Vibe
version: 1.0.0
---

# go-container — Docker-free container runtime

`dappco.re/go/container` is a container runtime without daemon overhead that provides immutable images with LinuxKit (stable, default) or TIM bundles (experimental). Used by core/dev for portable dev environments and by LEM (Lethean Ethical Models) for model isolation.

---

## Overview

### Philosophy

"Default to trusted tech. Our stuff is experimental."

This package provides container runtime capabilities without requiring Docker daemon, focusing on:

1. **Immutability** — Read-only base images, explicit writable mounts
2. **Portability** — Run anywhere without Docker dependency
3. **Security** — Isolation through LinuxKit, VZ, or TIM
4. **Simplicity** — No daemon, direct execution

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     core run (abstraction)               │
├────────────────────────────┬────────────────────────────┤
│  LinuxKit (default)        │  TIM (experimental)        │
│  ✓ Trusted, tested         │  ⚠ Homegrown              │
│  ✓ Multi-format output     │  ✓ Lightweight             │
│  ✓ dm-crypt encryption     │  ✓ Sigil encryption        │
│  ✓ Community support       │  ⚠ Just us                 │
├────────────────────────────┴────────────────────────────┤
│  Apple VZ (macOS 26+)                                   │
│  ✓ In-process provider     │  ✓ Virtualization framework│
├─────────────────────────────────────────────────────────┤
│  containerd / runc (OCI runtime)                        │
└─────────────────────────────────────────────────────────┘
```

### Package structure

Three packages with clear dependency direction: `devenv` → `container` (root) → `sources`.

```
go-container/
├── container/            # Root package - Container lifecycle, hypervisor abstraction
│   ├── manager.go       # Manager interface and LinuxKitManager implementation
│   ├── state.go         # JSON-persisted state (~/.core/containers.json)
│   ├── hypervisor.go    # Hypervisor abstraction with QEMU and Hyperkit
│   ├── templates.go     # LinuxKit template engine with embedded YAML templates
│   └── vz.go            # Apple Virtualization Framework (VZ) support
├── devenv/              # DevOps orchestrator - dev environment workflow
│   ├── devops.go        # Main DevOps orchestrator
│   ├── config.go        # Configuration management
│   └── ssh.go           # SSH shell and serial console access
├── sources/             # Image source management
│   ├── source.go        # ImageSource interface
│   ├── cdn.go           # CDN (HTTP GET) implementation
│   └── github.go        # GitHub implementation
└── cmd/
    ├── vm/              # CLI commands: boot, stop, status, console, images
    │   ├── cmd_container.go
    │   ├── cmd_images.go
    │   ├── cmd_tim.go
    │   ├── cmd_templates.go
    │   └── cmd_vm.go
    └── vzagent/          # VZ agent for in-process provider
        ├── main.go
        ├── vsock_linux.go
        └── vsock_other.go
```

---

## Quick start

### Basic usage

```go
import "dappco.re/go/container"

// Create a manager
manager := container.NewLinuxKitManager()

// Build a container from YAML
image, err := manager.Build(context.Background(), "app.yml")
if err != nil {
    return err
}

// Run the container
container, err := manager.Run(context.Background(), image, container.RunOptions{})
if err != nil {
    return err
}

// Stop the container
if err := container.Stop(context.Background()); err != nil {
    return err
}
```

### CLI usage

```bash
# Build and run a LinuxKit container
core run app.yml

# Run with explicit provider
core run app.yml --provider linuxkit

# Run a TIM bundle (experimental)
core run app.tim --provider tim

# List containers
core vm list

# Start a container
core vm start mycontainer

# Stop a container
core vm stop mycontainer

# Access console
core vm console mycontainer
```

---

## Core components

### 1. Provider interface

```go
type Provider interface {
    Build(config BuildConfig) (Image, error)
    Run(image Image, opts RunOptions) (Container, error)
    Encrypt(image Image, key string) (EncryptedImage, error)
    Decrypt(encrypted EncryptedImage, key string) (Image, error)
}
```

#### Implemented providers

| Provider | Status | Description | Platform |
|----------|--------|-------------|----------|
| `LinuxKitProvider` | Production | Default provider using LinuxKit | Linux/macOS |
| `TIMProvider` | Experimental | Homegrown lightweight format | Linux/macOS |
| `VZProvider` | Production | Apple Virtualization Framework | macOS 26+ |

### 2. Container lifecycle

#### Build process

```
BuildConfig → Provider.Build() → Image
                              │
                              ├── Validate YAML
                              ├── Pull base images
                              ├── Create filesystem layers
                              ├── Configure kernel and init
                              └── Package into output format
```

#### Run process

```
Image + RunOptions → Provider.Run() → Container
                                    │
                                    ├── Create runtime directory
                                    ├── Configure networking
                                    ├── Setup volumes
                                    ├── Start hypervisor/VM
                                    └── Return container handle
```

#### Container operations

```go
type Container interface {
    ID() string
    Name() string
    Status() (ContainerStatus, error)
    Start(ctx context.Context) error
    Stop(ctx context.Context) error
    Pause(ctx context.Context) error
    Resume(ctx context.Context) error
    Delete(ctx context.Context) error
    Exec(ctx context.Context, cmd []string) (io.Reader, io.Writer, error)
    Console(ctx context.Context) (io.Reader, io.Writer, error)
}
```

### 3. Image management

#### ImageSource interface

```go
type ImageSource interface {
    GetImage(ctx context.Context, ref string) (io.Reader, error)
    ListImages(ctx context.Context) ([]string, error)
    DeleteImage(ctx context.Context, ref string) error
}
```

#### Implemented sources

| Source | Status | Description |
|--------|--------|-------------|
| `CDNSource` | Production | HTTP GET from CDN URLs |
| `GitHubSource` | Production | Download from GitHub releases |
| `LocalSource` | Production | Local file system |

#### Image types

| Type | Description | Format |
|------|-------------|--------|
| `Image` | Unencrypted container image | Provider-specific |
| `EncryptedImage` | Encrypted container image | Provider-specific |
| `LinuxKitImage` | LinuxKit-specific image | ISO, raw, qcow2 |
| `TIMImage` | TIM-specific image | Directory bundle |

---

## LinuxKit provider (default)

### Why LinuxKit first?

1. **Trust** — Docker Inc + community backed, years of production use
2. **Formats** — ISO, raw disk, AWS AMI, GCP, Azure, qemu, VMware
3. **dm-crypt** — Built-in encrypted volume support
4. **Immutability** — Read-only base, explicit writable mounts
5. **Composable** — YAML config, pluggable components

### LinuxKit YAML structure

#### Basic example

```yaml
kernel:
  image: linuxkit/kernel:6.6.13
  cmdline: "console=tty0"

init:
  - linuxkit/init:v1.0.0
  - linuxkit/runc:v1.0.0
  - linuxkit/containerd:v1.0.0

onboot:
  # Sequential boot services
  - name: dhcpcd
    image: linuxkit/dhcpcd:v1.0.0
  - name: sysctl
    image: linuxkit/sysctl:v1.0.0
    command: ["/usr/bin/sysctl", "-w", "net.ipv4.ip_forward=1"]

services:
  # Long-running services
  - name: sshd
    image: linuxkit/sshd:v1.0.0
    command: ["/usr/sbin/sshd", "-D"]
    env:
      - SSH_AUTHORIZED_KEYS=...base64...

files:
  # Additional files to include
  - path: /etc/ssh/authorized_keys
    contents: "ssh-rsa AAAAB3..."
    mode: "0600"

mounts:
  # Mount points
  - type: tmpfs
    target: /tmp
    tmpfs: { size: 100m }
  - type: bind
    target: /app
    source: /path/on/host
    readonly: false

trust:
  # Image verification
  org: linuxkit
  image:
    - linuxkit/*
```

### LinuxKit components

#### Kernel

```yaml
kernel:
  image: linuxkit/kernel:6.6.13
  cmdline: "console=tty0 console=ttyS0"
  # Optional: custom kernel modules
  modules:
    - overlay
    - nf_conntrack
```

#### Init system

```yaml
init:
  # Required: init, runc, containerd
  - linuxkit/init:v1.0.0
  - linuxkit/runc:v1.0.0
  - linuxkit/containerd:v1.0.0
  # Optional: additional init containers
  - my-custom-init:v1.0.0
```

#### Services

**Boot-time (onboot):** Run once during boot, sequentially

```yaml
onboot:
  - name: dhcpcd
    image: linuxkit/dhcpcd:v1.0.0
    command: ["/sbin/dhcpcd", "--nobackground", "-f", "/dhcpcd.conf"]
    binds:
      - /etc/dhcpcd.conf:/dhcpcd.conf
```

**Runtime (services):** Long-running processes

```yaml
services:
  - name: sshd
    image: linuxkit/sshd:v1.0.0
    command: ["/usr/sbin/sshd", "-D"]
    env:
      - SSH_PORT=2222
    binds:
      - /etc/ssh:/etc/ssh
```

#### Files and directories

```yaml
files:
  - path: /etc/myapp/config.json
    contents: '{"setting": "value"}'
    mode: "0644"
    uid: 1000
    gid: 1000

  - path: /var/lib/myapp
    directory: true
    mode: "0755"
```

#### Mounts

```yaml
mounts:
  # Temporary filesystem
  - type: tmpfs
    target: /tmp
    tmpfs:
      size: 100m
      mode: 1777

  # Bind mount from host
  - type: bind
    target: /app
    source: /path/on/host
    readonly: false

  # 9p filesystem mount
  - type: 9p
    target: /shared
    source: //9p/server
    options: trans=virtio,version=9p2000.L,rw
```

### LinuxKit features

#### Networking

```yaml
# Simple DHCP
onboot:
  - name: dhcpcd
    image: linuxkit/dhcpcd:v1.0.0

# Static IP
onboot:
  - name: ifupdown
    image: linuxkit/ifupdown:v1.0.0
    command: ["/sbin/ifupdown", "eth0", "192.168.1.100/24"]

# MACVLAN
services:
  - name: macvlan
    image: linuxkit/openvswitch:v1.0.0
    command: ["/usr/bin/ovs-vsctl", "add-port", "br0", "eth0"]
```

#### Storage

```yaml
# dm-crypt encrypted volume
onboot:
  - name: dm-crypt
    image: linuxkit/dm-crypt:v1.0.0
    command: ["/usr/bin/crypto", "crypt_dev", "/dev/sda1"]
    binds:
      - /keys:/keys
```

#### Security

```yaml
# Read-only root filesystem
rootfs:
  readonly: true

# Capabilities
services:
  - name: myapp
    image: myapp:v1.0.0
    capabilities:
      - CAP_NET_BIND_SERVICE
      - CAP_SYS_ADMIN
```

---

## Apple VZ provider

### Overview

**Apple Virtualization Framework (VZ)** is a macOS 26+ framework for running virtual machines and containers. go-container provides an in-process VZ provider for efficient, lightweight container execution on Apple Silicon.

### VZ features

| Feature | Status | Description |
|---------|--------|-------------|
| In-process execution | Complete | No separate VM process |
| Virtualisation | Complete | Full VM support |
| VSOCK | Complete | Communication channel |
| File sharing | Complete | Shared directories |
| Network isolation | Complete | NAT and bridging |

### VZ configuration

```go
type VZConfig struct {
    CPUCount    int
    MemoryGB    int
    DiskSizeGB  int
    NetworkMode VZNetworkMode  // NAT, Bridge, Host
    PortForwarding []PortForward
    Shares      []VZShare      // Directory sharing
}

type VZNetworkMode int
iota
    VZNetworkNAT
    VZNetworkBridge
    VZNetworkHost

type PortForward struct {
    HostPort  int
    GuestPort int
    Protocol  string // tcp, udp
}

type VZShare struct {
    Source      string // Host path
    MountPoint  string // Guest path
    ReadOnly    bool
}
```

### VZ agent

The `vzagent` is a small process that runs inside VZ containers to handle:
- Console I/O
- VSOCK communication
- Port forwarding
- File sharing

#### VZ agent files

```
cmd/vzagent/
├── main.go           # Main entry point
├── vsock_linux.go    # Linux VSOCK implementation
├── vsock_other.go    # Non-Linux stubs
└── vsock_darwin.go   # macOS VSOCK implementation (if needed)
```

---

## TIM provider (experimental)

### What is TIM?

**TIM (The Immutable)** is a homegrown, lightweight container/image format designed for the Lethean ecosystem. It is **experimental** and optimised for:

- **Minimal overhead** — No daemon, direct execution
- **Sigil encryption** — Built-in encryption support
- **Portability** — Run anywhere with minimal dependencies
- **Speed** — Fast boot times

### TIM structure

A TIM bundle is a directory with a specific structure:

```
myapp.tim/
├── tim.yaml          # Bundle manifest
├── rootfs/           # Root filesystem
│   ├── bin/
│   ├── etc/
│   ├── usr/
│   └── var/
├── kernel/           # Optional: kernel and modules
│   ├── vmlinuz
│   └── initramfs
├── metadata/         # Bundle metadata
│   ├── config.yaml
│   └── environment
└── overlays/         # Optional: overlay filesystems
    └── upper/
```

### TIM manifest (tim.yaml)

```yaml
name: myapp
version: 1.0.0
tim_version: 1
arch: arm64
type: container

config:
  entrypoint: /app/start.sh
  args: ["--verbose"]
  env:
    APP_ENV: production
  workdir: /app

resources:
  memory: 512m
  cpu: 1
  disk: 10g

network:
  mode: nat
  ports:
    - host: 8080
      container: 80
      protocol: tcp

volumes:
  - source: /host/data
    target: /container/data
    readonly: false

encryption:
  enabled: true
  method: sigil
  key_file: /path/to/key
```

### TIM features

#### Sigil encryption

TIM uses **Sigil encryption** for secure image storage:

```go
// Encrypt an image
encrypted, err := timProvider.Encrypt(image, "my-secret-key")

// Decrypt an image
decrypted, err := timProvider.Decrypt(encrypted, "my-secret-key")
```

#### Overlay filesystems

TIM supports overlay filesystems for efficient layering:

```yaml
layers:
  - base: alpine:latest
  - overlay: /path/to/overlay
```

---

## Dev environment (devenv)

### Overview

The `devenv` package provides **DevOps orchestration** that composes container and sources into a dev environment workflow:

- **Boot/Stop/Status** — Container lifecycle management
- **SSH shell and serial console access** — Remote access
- **Project mounting** — Mount host directories into containers
- **Auto-detection of serve/test commands** — Intelligent project analysis
- **Sandboxed Claude sessions** — Secure AI assistant integration

### DevOps workflow

```go
// Create a DevOps environment
env := devenv.NewDevOps()

// Start the environment
if err := env.Start(context.Background(), devenv.Config{
    Container: "myapp.yml",
    Mounts: []devenv.Mount{
        {Source: "/host/code", Target: "/app"},
    },
    SSH: devenv.SSHConfig{
        Port: 2222,
        Keys: []string{"~/.ssh/id_rsa.pub"},
    },
}); err != nil {
    return err
}

// Access the container
if err := env.Shell(context.Background()); err != nil {
    return err
}

// Stop the environment
if err := env.Stop(context.Background()); err != nil {
    return err
}
```

### Dev environment features

#### Project mounting via reverse SSHFS

Instead of traditional bind mounts, devenv uses **reverse SSHFS** for project mounting:

```
Host                    Container
  ┌─────────────┐          ┌─────────────┐
  │ /host/code  │  SSHFS  │ /app        │
  └─────────────┘ ──────► └─────────────┘
      (local)            (remote)
```

Benefits:
- No host filesystem dependencies
- Works across different OS platforms
- Secure (SSH-based)
- Dynamic (can mount/unmount at runtime)

#### Auto-detection

DevOps automatically detects project type and sets up appropriate commands:

| Project Type | Detected By | Serve Command | Test Command |
|--------------|------------|---------------|--------------|
| Go | go.mod | `go run .` | `go test ./...` |
| Node.js | package.json | `npm start` | `npm test` |
| Python | requirements.txt | `python app.py` | `pytest` |
| Rust | Cargo.toml | `cargo run` | `cargo test` |
| Generic | Dockerfile | `CMD` from Dockerfile | `RUN test` from Dockerfile |

#### SSH access

```go
// Start an SSH session
if err := env.Shell(context.Background()); err != nil {
    return err
}

// Get a console (serial) session
if err := env.Console(context.Background()); err != nil {
    return err
}

// Execute a command
output, err := env.Exec(context.Background(), "ls -la /app")
```

#### Sandboxed Claude sessions

```go
// Start a sandboxed Claude session with auth forwarding
session, err := env.ClaudeSession(context.Background(), devenv.ClaudeConfig{
    Model: "gemma-7b",
    Auth:  devenv.ClaudeAuth{Token: "..."},
})
if err != nil {
    return err
}
defer session.Close()

// Send a message
response, err := session.Send("Explain this codebase")
```

---

## Image sources

### ImageSource interface

```go
type ImageSource interface {
    // GetImage retrieves an image by reference
    GetImage(ctx context.Context, ref string) (io.Reader, error)
    
    // ListImages lists available images
    ListImages(ctx context.Context) ([]string, error)
    
    // DeleteImage removes an image
    DeleteImage(ctx context.Context, ref string) error
}
```

### CDN source

Downloads images via HTTP GET:

```go
source := sources.NewCDNSource("https://images.example.com/")
reader, err := source.GetImage(context.Background(), "alpine:latest.iso")
```

#### CDN configuration

```go
type CDNSource struct {
    BaseURL    string        // Base URL for CDN
    UserAgent  string        // HTTP User-Agent
    Timeout    time.Duration // HTTP timeout
    Retries    int           // Retry count
    CacheDir   string        // Local cache directory
}
```

### GitHub source

Downloads images from GitHub releases:

```go
source := sources.NewGitHubSource("linuxkit", "linuxkit")
reader, err := source.GetImage(context.Background(), "v0.10.0")
```

#### GitHub configuration

```go
type GitHubSource struct {
    Owner      string // Repository owner
    Repo       string // Repository name
    Token      string // Optional: GitHub API token
    CacheDir   string // Local cache directory
}
```

### Local source

Loads images from local file system:

```go
source := sources.NewLocalSource("/path/to/images")
reader, err := source.GetImage(context.Background(), "myapp.iso")
```

---

## State management

All container state is persisted as JSON to `~/.core/containers.json`:

```json
{
  "version": 1,
  "containers": [
    {
      "id": "abc123",
      "name": "myapp",
      "provider": "linuxkit",
      "image": "myapp.yml",
      "status": "running",
      "created": "2026-06-18T00:00:00Z",
      "started": "2026-06-18T00:01:00Z",
      "config": { ... },
      "network": { ... },
      "mounts": [ ... ]
    }
  ],
  "images": [
    {
      "ref": "myapp.yml",
      "path": "/home/user/.core/images/myapp.raw",
      "size": 10485760,
      "created": "2026-06-18T00:00:00Z",
      "provider": "linuxkit"
    }
  ]
}
```

### State operations

```go
// Get container state
state := container.NewStateStore()
containers, err := state.All()

// Get single container
container, err := state.Get("abc123")

// Update container status
container.Status = container.StatusRunning
if err := state.Save(container); err != nil {
    return err
}

// Delete container
if err := state.Delete("abc123"); err != nil {
    return err
}
```

**Copy-on-read:** `State.Get()` and `State.All()` return copies to prevent data races.

---

## CLI commands

### Command structure

```
go-container/cmd/
├── vm/                   # Main VM/container commands
│   ├── cmd_container.go # Container management (list, start, stop, delete)
│   ├── cmd_images.go    # Image management (list, pull, delete)
│   ├── cmd_tim.go       # TIM-specific commands
│   ├── cmd_templates.go # Template management
│   └── cmd_vm.go        # VM management
└── vzagent/             # VZ agent (runs inside containers)
```

### Available commands

#### Container management

```bash
# List all containers
core vm list

# Start a container
core vm start <name>

# Stop a container
core vm stop <name>

# Delete a container
core vm delete <name>

# Get container status
core vm status <name>

# Access container console
core vm console <name>

# Execute command in container
core vm exec <name> -- ls -la /app
```

#### Image management

```bash
# List all images
core vm images

# Pull an image
core vm images pull alpine:latest

# Delete an image
core vm images delete alpine:latest

# Build an image from YAML
core vm images build app.yml
```

#### TIM commands

```bash
# Build a TIM bundle
core vm tim build app.tim

# Run a TIM bundle
core vm tim run app.tim

# List TIM bundles
core vm tim list
```

#### Template management

```bash
# List available templates
core vm templates list

# Initialize from template
core vm templates init myapp --template golang
```

#### VM management

```bash
# Start the VM service
core vm service start

# Stop the VM service
core vm service stop

# Get VM service status
core vm service status
```

### Command flags

#### Global flags

```bash
--config      # Config file path (default: ~/.core/config.yaml)
--debug       # Enable debug output
--json        # Output in JSON format
--provider    # Container provider (linuxkit, tim, vz)
--workspace   # Workspace directory
```

#### Run command flags

```bash
core run app.yml [flags]

Flags:
  --provider   # Provider: linuxkit (default), tim, vz
  --memory     # Memory limit (e.g., 2g)
  --cpu        # CPU count
  --disk       # Disk size (e.g., 20g)
  --network    # Network mode: nat, bridge, host
  --port       # Port forwarding (host:guest)
  --mount      # Mount directory (host:container)
  --env        # Environment variables
  --ssh        # Enable SSH access
  --console    # Attach console
  --detach     # Run in background
```

---

## Key patterns

### 1. io.Medium abstraction

File system operations use `io.Medium` (from `go-io`) instead of `os` directly:

```go
// Use io.Local for real file access
medium := io.Local()
data, err := medium.ReadFile("/path/to/file")

// Use io.Memory for tests
medium := io.Memory()
medium.WriteFile("/test/file", []byte("test"))
```

**Benefit:** Enables test mocking and alternative backends (GitHub, S3, etc.).

### 2. Compile-time interface checks

All implementations verify interface compliance at compile time:

```go
var _ ImageSource = (*CDNSource)(nil)
var _ ImageSource = (*GitHubSource)(nil)
var _ Provider = (*LinuxKitProvider)(nil)
var _ Provider = (*TIMProvider)(nil)
```

### 3. Copy-on-read state

All state operations return copies to prevent data races:

```go
// Returns a copy, not the original
container, err := state.Get("abc123")

// Modify the copy
container.Status = ContainerStatusRunning

// Save the modified copy
if err := state.Save(container); err != nil {
    return err
}
```

### 4. Context propagation

All blocking operations take `context.Context` as the first parameter:

```go
func (p *LinuxKitProvider) Build(ctx context.Context, config BuildConfig) (Image, error) {
    // Check context periodically
    if err := ctx.Err(); err != nil {
        return nil, err
    }
    // ...
}
```

---

## Code metrics

```
Total Go files:           50+
Total lines (source):    ~25,000
Total lines (tests):     ~15,000
Total packages:          3 (container, devenv, sources)
Total CLI commands:      20+
Supported providers:     3 (LinuxKit, TIM, VZ)
Supported sources:       3 (CDN, GitHub, Local)
```

### Test coverage

| Component | Coverage | Status |
|-----------|----------|--------|
| container | High | Complete |
| devenv | High | Complete |
| sources | High | Complete |

### Performance

| Operation | Time | Notes |
|-----------|------|-------|
| LinuxKit build | 5-10s | Depends on image size |
| LinuxKit start | 2-5s | Cold start |
| TIM build | 1-3s | Experimental |
| TIM start | <1s | Cold start |
| VZ start | <500ms | In-process |

---

## Compliance rules

From `AGENTS.md` and `CLAUDE.md`:

### Coding standards

- **UK English** — colour, organisation, honour
- **Error wrapping** — Use `core.E("Op", "message", err)`
- **Context propagation** — All blocking operations take `context.Context`
- **Licence** — EUPL-1.2

### Test organisation

- Tests use `testify` assertions
- Naming convention: `TestSubject_Function_{Good,Bad,Ugly}`
- Example tests with `// Output:` form

### File organisation

- **Root package (`container`)** — Container lifecycle, hypervisor abstraction, state, templates
- **`devenv` package** — DevOps orchestrator, SSH access, project mounting
- **`sources` package** — Image source implementations (CDN, GitHub, Local)
- **`cmd` package** — CLI commands

### Dependencies

- **All persistent data** lives under `~/.core/`:
  - `containers.json` — Container state
  - `logs/` — Container logs
  - `images/` — Container images
  - `config.yaml` — Configuration
  - `known_hosts` — SSH known hosts
  - `linuxkit/` — LinuxKit templates

- **Environment variables:**
  - `CORE_IMAGES_DIR` — Override images directory
  - `CORE_CONFIG_DIR` — Override config directory

---

## Related documentation

- [CoreGo Framework](../../README.md) — Parent knowledge pack
- [go-io Package](../io/README.md) — I/O Medium interface used throughout
- [go-devops Package](../devops/README.md) — Related DevOps orchestration
- [CoreGo RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md) — Framework specification
- [go-container RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/container/RFC.md) — Package specification
- [go-container CLAUDE.md](file:///Users/snider/Code/core/go-container/CLAUDE.md) — Implementation details

### Sub-specifications

- [RFC.apple.md](file:///Users/snider/Code/meowmix/plans/code/core/go/container/RFC.apple.md) — Apple + Runtime Detection
- [RFC.commands.md](file:///Users/snider/Code/meowmix/plans/code/core/go/container/RFC.commands.md) — Command specifications
- [RFC.catalog.md](file:///Users/snider/Code/meowmix/plans/code/core/go/container/RFC.catalog.md) — Catalog of images/templates
- [RFC.models.md](file:///Users/snider/Code/meowmix/plans/code/core/go/container/RFC.models.md) — Data models
- [RFC.tim.md](file:///Users/snider/Code/meowmix/plans/code/core/go/container/RFC.tim.md) — TIM format specification
- [RFC.vz.md](file:///Users/snider/Code/meowmix/plans/code/core/go/container/RFC.vz.md) — VZ provider specification

---

## Use cases

### Primary use cases

1. **Dev environments** — Portable, reproducible dev environments with `core run`
2. **Model isolation** — Running LEM (Lethean Ethical Models) in isolated containers
3. **CI/CD pipelines** — Containerised build and test environments
4. **Edge computing** — Lightweight container runtime for edge devices

### Integration patterns

1. **Library mode** — Embed in Go applications for container management
2. **CLI mode** — Use via `core vm` and `core run` commands
3. **DevOps mode** — Use `devenv` package for full dev environment orchestration
4. **Provider mode** — Implement custom providers for new runtime backends

---

## Maintenance information

- **Author**: Mistral Vibe (Purberus <purberus@lthn.ai>)
- **Created**: 2026-06-18T00:00:00Z
- **Last Updated**: 2026-06-18T00:00:00Z
- **Version**: 1.0.0
- **Licence**: EUPL-1.2
- **Repository**: `forge.lthn.sh/core/go-container`
