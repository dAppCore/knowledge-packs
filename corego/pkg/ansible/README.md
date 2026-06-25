---
type: Package Deep Dive
title: go-ansible — Pure Go Ansible Engine
description: Complete documentation for go-ansible — a pure Go implementation of Ansible playbook orchestration
module: dappco.re/go/ansible
repo: core/go-ansible
tags: [ansible, orchestration, infrastructure, deployment, ssh, yaml, jinja2]
created: 2026-06-17T23:30:00Z
author: Mistral Vibe
version: 1.0.0
---

# go-ansible — Pure Go Ansible Playbook Engine

`dappco.re/go/ansible` is a **native Go implementation** of core Ansible playbook mechanics. It parses YAML playbooks, inventories, and roles, then executes tasks on remote hosts via SSH — **without any Python dependency**.

---

## Overview

### What it is

- **Pure Go Ansible engine** — No Python, no external dependencies
- **Library-first** — Designed as a library, CLI integration is optional
- **Production-ready** — Used by DevOps playbooks for fleet management (noc, de1, syd1)
- **45+ module handlers** — Supports ansible.builtin.*, ansible.posix.*, community.general.*, community.docker.*

### What it is NOT

- Not a drop-in replacement for Python Ansible (some advanced features may be missing)
- Not a standalone binary (CLI is part of `core` binary)
- Not a reimplementation of ALL Ansible modules (focus on most common ones)

---

## Architecture

```
Playbook YAML ──► Parser ──► []Play ──► Executor ──► Module Handlers ──► SSH/Local Client ──► Remote/Local Host
                              │                         │
Inventory YAML ──► Parser ──► Inventory        Callbacks (OnPlayStart, OnTaskEnd, OnPlayEnd)
```

### Core Layers

| Layer | File | Purpose |
|-------|------|---------|
| **Data Model** | `types.go` | Core structs: Playbook, Play, Task, RoleRef, Inventory, Host, Facts, TaskResult, TaskApply, LoopControl |
| **Parsing** | `parser.go` | YAML parsing (15 methods): playbooks, inventories, tasks, roles, vars files. Iterator support via `iter.Seq` |
| **Orchestration** | `executor.go` | Orchestration (68 methods): host resolution, play execution order, `when:` conditions, `{{ }}` templating, loops, block/rescue/always, handler notification, facts gathering |
| **Modules** | `modules.go` | 45 module handler implementations |
| **Transport** | `ssh.go` + `local_client.go` | SSH client and local execution |
| **Core Integration** | `core_primitives.go` | Core framework integrations: paths, strings, I/O, process |
| **CLI** | `cmd/ansible/` | CLI command registration via `dappco.re/go` command registry |

---

## Quick start

### Basic Usage

```go
import "dappco.re/go/ansible"

executor := ansible.NewExecutor("/srv/playbooks")
if err := executor.SetInventory("/srv/inventory.yml"); err != nil {
    return err
}
executor.SetVar("release", "2026.04")
return executor.Run(context.Background(), "/srv/playbooks/site.yml")
```

### CLI Integration

```go
app := core.New()
ansiblecmd.Register(app)
```

Then use:
```bash
core ansible playbook.yml -i inventory.yml -e release=2026.04
```

---

## Package structure

### Main Files

```
ansible/
├── types.go              # Core data structures
├── parser.go             # YAML parsing for playbooks, inventories, tasks
├── executor.go           # Main orchestration engine
├── modules.go            # Module handler implementations
├── ssh.go                # SSH client implementation
├── local_client.go       # Local execution client
├── core_primitives.go    # Core framework integrations
├── template_features.go  # Jinja2-style templating
├── async_features.go     # Async execution support
├── result_helpers.go     # Result handling utilities
├── service.go            # Service integration
└── cmd/
    └── ansible/
        ├── cmd.go         # CLI command wiring
        ├── ansible.go     # Playbook execution
        └── ...
```

### Test Files

```
ansible/
├── types_test.go         # YAML unmarshalling tests
├── parser_test.go        # Playbook/inventory parsing tests
├── executor_test.go      # Executor lifecycle, conditions, templating, loops
├── ssh_test.go           # SSH client tests
├── mock_ssh_test.go      # Mock SSH infrastructure
├── modules_cmd_test.go  # Command modules (shell, command, raw, script)
├── modules_file_test.go  # File modules (copy, template, file, etc.)
├── modules_svc_test.go   # Service modules (service, systemd, user, group)
├── modules_infra_test.go # Infrastructure modules (apt, pip, git, etc.)
├── modules_adv_test.go   # Advanced modules (debug, fail, assert, etc.)
└── ...
```

---

## Playbook execution flow

### Per-Play Order

1. **Gather facts** (via `setup` module, unless `gather_facts: no`)
2. **`pre_tasks`**
3. **`roles`** (with `tasks/main.yml`, `handlers/main.yml`, `defaults/main.yml`, `vars/main.yml`)
4. **`tasks`**
5. **`post_tasks`**
6. **Notified `handlers`** (deduplicated, run once at end)

### Batch Execution

- `serial:` — Execute play on hosts in batches (int or percentage)
- `max_fail_percentage:` — Halt batch if failure % exceeded
- `any_errors_fatal:` — Stop entire play on first failure
- `force_handlers:` — Run handlers even on failure

---

## Module handlers (45 total)

### Command Execution (4)

| Module | FQCN | Description |
|--------|------|-------------|
| `shell` | `ansible.builtin.shell` | Execute shell commands |
| `command` | `ansible.builtin.command` | Execute commands without shell |
| `raw` | `ansible.builtin.raw` | Execute raw SSH commands (no fact gathering) |
| `script` | `ansible.builtin.script` | Execute local scripts on remote host |

### File Operations (8)

| Module | FQCN | Description |
|--------|------|-------------|
| `copy` | `ansible.builtin.copy` | Copy files to remote host |
| `template` | `ansible.builtin.template` | Template and copy files (Jinja2 support) |
| `file` | `ansible.builtin.file` | Manage file attributes (touch, symlink, mode, owner) |
| `lineinfile` | `ansible.builtin.lineinfile` | Ensure line in file (insert, replace, remove) |
| `stat` | `ansible.builtin.stat` | Get file stats (exists, isdir, mode, owner, size) |
| `slurp` | `ansible.builtin.slurp` | Read file contents |
| `fetch` | `ansible.builtin.fetch` | Download files from remote |
| `get_url` | `ansible.builtin.get_url` | Download URLs |

### Package Management (7)

| Module | FQCN | Description |
|--------|------|-------------|
| `apt` | `ansible.builtin.apt` | Debian/Ubuntu packages |
| `apt_key` | `ansible.builtin.apt_key` | APT repository keys |
| `apt_repository` | `ansible.builtin.apt_repository` | APT repository management |
| `yum` | `ansible.builtin.yum` | RedHat/CentOS packages |
| `dnf` | `ansible.builtin.dnf` | Fedora/RHEL 8+ packages |
| `rpm` | `ansible.builtin.rpm` | RPM package operations |
| `pip` | `ansible.builtin.pip` | Python pip packages |
| `package` | `ansible.builtin.package` | Generic package manager abstraction |

### Service Management (4)

| Module | FQCN | Description |
|--------|------|-------------|
| `service` | `ansible.builtin.service` | Manage init.d/upstart services |
| `systemd` | `ansible.builtin.systemd` | Manage systemd services |
| `user` | `ansible.builtin.user` | Manage user accounts |
| `group` | `ansible.builtin.group` | Manage groups |

### Network & HTTP (1)

| Module | FQCN | Description |
|--------|------|-------------|
| `uri` | `ansible.builtin.uri` | HTTP requests (GET, POST, PUT, DELETE) |

### Control Flow & Facts (9)

| Module | FQCN | Description |
|--------|------|-------------|
| `debug` | `ansible.builtin.debug` | Print variables and messages |
| `fail` | `ansible.builtin.fail` | Fail the play with a message |
| `assert` | `ansible.builtin.assert` | Assert conditions |
| `ping` | `ansible.builtin.ping` | Test connectivity |
| `set_fact` | `ansible.builtin.set_fact` | Set variables |
| `add_host` | `ansible.builtin.add_host` | Dynamically add hosts to inventory |
| `group_by` | `ansible.builtin.group_by` | Group hosts dynamically |
| `pause` | `ansible.builtin.pause` | Pause playbook execution |
| `wait_for` | `ansible.builtin.wait_for` | Wait for condition (port, file, string) |

### Infrastructure (7)

| Module | FQCN | Description |
|--------|------|-------------|
| `git` | `ansible.builtin.git` | Git clone, pull, checkout |
| `unarchive` | `ansible.builtin.unarchive` | Extract archives (zip, tar, tar.gz, etc.) |
| `archive` | `ansible.builtin.archive` | Create archives |
| `hostname` | `ansible.builtin.hostname` | Set hostname |
| `sysctl` | `ansible.builtin.sysctl` | Manage sysctl parameters |
| `cron` | `ansible.builtin.cron` | Manage cron jobs |
| `blockinfile` | `ansible.builtin.blockinfile` | Manage text blocks in files |
| `include_vars` | `ansible.builtin.include_vars` | Load variables from files |
| `meta` | `ansible.builtin.meta` | Execute meta tasks (flush handlers, etc.) |
| `setup` | `ansible.builtin.setup` | Gather facts from remote host |
| `reboot` | `ansible.builtin.reboot` | Reboot remote host |

### Community Modules (3)

| Module | FQCN | Description |
|--------|------|-------------|
| `ufw` | `community.general.ufw` | Manage UFW firewall rules |
| `authorized_key` | `ansible.posix.authorized_key` | Manage SSH authorized_keys |
| `docker_compose` | `community.docker.docker_compose` | Run docker-compose |

---

## Parser

### Custom YAML Parsing

Ansible uses a non-standard YAML format where module names are YAML keys:

```yaml
- name: Install nginx
  apt:                    # ← "apt" is the module name (a YAML key, not a field)
    name: nginx
    state: present
```

`Task.UnmarshalYAML` scans map keys against `KnownModules` to extract the module name and args. Free-form syntax (`shell: echo hello`) stored as `Args["_raw_params"]`.

### Parser Methods (15)

- `NewParser(basePath)` — Create parser with base directory for relative paths
- `ParsePlaybook(path)` — Parse playbook file, expand `import_playbook` directives
- `ParsePlaybookIter(path)` — Streaming parser returning `iter.Seq[Play]`
- `ParseInventory(path)` — Parse inventory file (YAML format)
- `ParseRoles(roleDir)` — Load role tasks, defaults, vars, handlers
- `ParseTasksFromDir(dir)` — Load tasks from directory (with `main.yml` fallback)
- `ParseVarsFiles(pattern)` — Load vars from files with glob support
- Iterator variants support streaming for large playbooks

### Inventory Format

YAML inventory supports:
- Flat host lists (all, webservers, databases groups)
- Nested groups with `children:` field
- Host variables via `host_vars:` object
- Group variables via `vars:` object
- Connection metadata: `ansible_host`, `ansible_user`, `ansible_port`, `ansible_password`

---

## Templating & variables

### Jinja2-Compatible Templating

```yaml
- name: Configure service
  template:
    src: "{{ config_dir }}/nginx.conf.j2"
    dest: /etc/nginx/nginx.conf
```

### Variable Resolution Order (Highest → Lowest Priority)

1. Task-level `vars:`
2. `set_fact` results from current host
3. Play `vars:`
4. `vars_files:` (play-level)
5. Inventory host variables
6. Inventory group variables
7. Facts (gathered via `setup` module)
8. Magic variables (`inventory_hostname`, `groups`, `group_names`, `ansible_host`, etc.)

### Supported Filters

- `default()` — Provide default value if undefined
- `upper()`, `lower()` — Case conversion
- `basename()`, `dirname()` — Path manipulation
- `join()` — Join list elements
- `split()` — Split strings
- `bool()` — Convert to boolean
- `int()` — Convert to integer
- `abs()` — Absolute value
- `min()`, `max()` — Min/max of list
- `length()` — Length of string/list

### Loops

Executor supports:
- `with_items:` — Loop over list
- `with_dict:` — Loop over dict key-value pairs
- `with_fileglob:` — Loop over file glob patterns
- `with_sequence:` — Loop over numeric sequence
- `with_together:` — Loop over multiple lists in parallel
- `with_subelements:` — Loop over nested structures
- `loop:` + `loop_control:` — Modern loop syntax with index, item control

### Conditions

`when:` field supports:
- Simple boolean: `when: ansible_os_family == "Debian"`
- Operators: `==`, `!=`, `<`, `>`, `<=`, `>=`, `in`, `not in`, `contains`, `is defined`, `is not defined`
- Compound: `and`, `or` (space-separated)
- Variable references: `{{ variable_name }}`

---

## SSH & local execution

### SSH Client

- **Lazy connection** — Connects on first use, cached per host
- **Auth chain** — Explicit key file → `~/.ssh/id_*` (id_rsa, id_ed25519, etc.) → password
- **`known_hosts` verification** — Strict host key checking
- **Become/sudo wrapping** — Privilege escalation (password and passwordless modes)
- **File transfer** — Via `cat >` piped through stdin
- **Port forwarding** — SSH config respects `ansible_port` per host
- **Timeout** — Default 30s, configurable

### Local Execution

`connection: local` tasks execute on controller via `localClient`:
- Direct file I/O (no SSH needed)
- Command execution via local shell (`bash -lc`)
- Become via sudo (with password piping)
- File upload/download via filesystem operations

### Client Contract

Both SSH and local clients implement `sshExecutorClient`:
- `Run(ctx, cmd)` — Execute command, return stdout/stderr/exitCode
- `RunScript(ctx, script)` — Execute bash script
- `Upload(ctx, reader, remote, mode)` — Upload file with permissions
- `Download(ctx, remote)` — Download file
- `FileExists(ctx, path)` — Check file existence
- `Stat(ctx, path)` — Get file metadata
- `BecomeState()`, `SetBecome(bool, user, pass)` — Privilege escalation control

---

## Executor API

### Main Methods (68 total)

#### Construction

- `NewExecutor(basePath)` — Create executor with playbook base directory
- `SetInventory(path)` — Load inventory from YAML file
- `SetInventoryDirect(inv)` — Set inventory programmatically
- `SetVar(key, value)` — Set play-scoped variable

#### Execution

- `Run(ctx, playbookPath)` — Execute playbook (main entry point)
- `RunPlay(ctx, play)` — Execute a single play
- `RunTask(ctx, host, task)` — Execute a single task on a host

#### Callbacks

Executor exposes callbacks for integration:
- `OnPlayStart(play)` — Called before play executes
- `OnTaskStart(host, task)` — Called before task on host
- `OnTaskEnd(host, task, result)` — Called after task completes
- `OnPlayEnd(play)` — Called after play completes

### Options

- `Limit` — Restrict hosts (comma-separated or regex)
- `Tags` — Run only tasks with matching tags
- `SkipTags` — Skip tasks with matching tags
- `CheckMode` — Dry-run (no changes made)
- `Diff` — Show file differences
- `Verbose` — Verbosity level (0-4)

### Task Fields Supported

- `name:` — Descriptive name
- `module_name:` — Any registered module
- `args:` — Module arguments
- `vars:` — Task-scoped variables
- `when:` — Conditional execution
- `tags:` — Task tags for selective execution
- `register:` — Store result in variable
- `notify:` — Notify handlers
- `loop:` — Iterate over items
- `block:` / `rescue:` / `always:` — Exception handling
- `retries:` / `delay:` — Retry logic
- `until:` — Retry until condition met
- `become:` / `become_user:` — Privilege escalation
- `ignore_errors:` — Continue even on failure
- `async:` / `poll:` — Background execution (polling mode)
- `connection:` — Override play connection (ssh/local)

---

## CLI integration

Registered via `core/cli`:

```bash
core ansible <playbook.yml> [flags]
core ansible test <host> [flags]
```

### Playbook Command Flags

- `--inventory`, `-i` — Path to inventory file
- `--limit`, `-l` — Limit hosts (comma-separated pattern)
- `--tags`, `-t` — Run only tasks with matching tags
- `--skip-tags` — Skip tasks with matching tags
- `--extra-vars`, `-e` — Extra variables (JSON or key=value format)
- `--verbose`, `-v` — Verbosity level (0-4, stackable)
- `--check` — Dry-run mode (no changes)
- `--diff` — Show file differences

### SSH Test Command

`ansible test <host>` validates SSH connectivity:

- `--user`, `-u` — SSH user (default: root)
- `--password` — SSH password
- `--key`, `-i` — Private key file
- `--port` — SSH port (default: 22)

Returns exit code 0 on success, 1 on failure.

---

## Advanced features

### Roles

Roles provide reusable task collections:

```
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml      # Tasks executed for this role
    ├── handlers/
    │   └── main.yml      # Handlers triggered by role tasks
    ├── defaults/
    │   └── main.yml      # Default variables (lowest priority)
    └── vars/
        └── main.yml      # Role variables
```

### Handlers

Handlers are deferred tasks triggered by `notify:` directives. Only notified handlers execute, and only once per play (deduplicated by name). Handlers run at end of play unless `force_handlers: yes` (runs even on failure).

### Block/Rescue/Always

Error handling:

```yaml
- block:                 # Try block
    - task1
    - task2
  rescue:               # Catch block
    - task_on_failure
  always:               # Finally block
    - task_always_runs
```

### Facts Gathering

`setup` module gathers facts via remote execution:
- OS/distribution (Linux, Ubuntu, CentOS, etc.)
- Network interfaces and IP addresses
- Memory, CPU, disk info
- System uptime, hostname

Facts cached per-play and available as `ansible_*` variables. Disabled with `gather_facts: no`.

### Serialisation & Async

- `serial:` — Execute play on hosts in batches
- `max_fail_percentage:` — Halt batch if failure % exceeded
- `async:` / `poll:` — Background task execution with polling
- `any_errors_fatal:` — Stop entire play on first failure

### Dynamic Inventory

- `add_host:` — Add host to inventory mid-play
- `group_by:` — Group hosts by fact/variable value
- Results available immediately in subsequent tasks

### Tags & Selective Execution

- Play/task/role tags enable selective execution
- `--tags` / `--skip-tags` filter execution
- Special tags: `always` (always runs), `never` (never auto-run)

---

## I/O Medium integration

Playbooks and inventory load from `io.Medium` — local files, Git repos, or S3. See `code/core/go/io/RFC.md §Medium`.

```go
ansible.Run(ansible.WithInventory(io.GitHub("core/devops")))
```

---

## Adding a new module

1. Add both FQCN and short form to `KnownModules` in `types.go`
2. Add the dispatch case in `executeModule()` switch in `modules.go`
3. Implement `module{Name}(ctx, client, args)` method on `Executor`
4. Write tests using mock SSH infrastructure (`mock_ssh_test.go`)

If adding new YAML keys to `Task`, update the `knownKeys` map in `Task.UnmarshalYAML` (`parser.go`) to prevent them being mistaken for module names.

---

## Compliance rules

From `AGENTS.md`:

- **File boundaries**: Keep changes within existing file boundaries
  - Parser behavior → `parser.go`
  - Execution flow → `executor.go`
  - Module handlers → `modules.go`
  - SSH transport → `ssh.go`
  - Local transport → `local_client.go`
  - Command wiring → `cmd/ansible`

- **Import restrictions**: Use `dappco.re/go` wrappers for:
  - Formatting: `format` package
  - JSON: `json` package
  - Filesystem: `io` package
  - Path: `path` package
  - String: `string` package
  - Bytes: `bytes` package
  - Errors: `error` package
  - Process: `process` package

- **Do NOT** add direct imports of:
  - `fmt`, `errors`, `strings`, `path`, `path/filepath`
  - `os`, `os/exec`, `encoding/json`, `bytes`, `log`

- **Test coverage**: Every public symbol needs:
  - `Test<File>_<Symbol>_Good` — Happy path
  - `Test<File>_<Symbol>_Bad` — Expected errors
  - `Test<File>_<Symbol>_Ugly` — Edge cases/panics
  - `Example<Symbol>` — Example usage

- **Naming**: UK English (colour, organisation, centre)
- **Licence**: EUPL-1.2

---

## Examples

### Simple Playbook

```yaml
---
- name: Ensure nginx installed
  hosts: webservers
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

### Using Variables

```yaml
---
- name: Deploy application
  hosts: app_servers
  vars:
    app_version: "1.2.3"
  tasks:
    - name: Download release
      get_url:
        url: "https://github.com/org/app/releases/download/v{{ app_version }}/app.tar.gz"
        dest: /tmp/app.tar.gz
    - name: Extract archive
      unarchive:
        src: /tmp/app.tar.gz
        dest: /opt/app
        remote_src: yes
```

### Using Roles

```yaml
---
- name: Deploy web stack
  hosts: webservers
  roles:
    - nginx
    - php
    - { role: app, tags: [app] }
  post_tasks:
    - name: Verify deployment
      uri:
        url: http://localhost
        return_content: yes
```

### Conditional Execution

```yaml
---
- name: Configure firewall
  hosts: all
  tasks:
    - name: Enable UFW on Ubuntu
      ufw:
        state: enabled
        policy: deny
      when: ansible_os_family == "Debian"
```

---

## Related documentation

- [CoreGo Framework](../../README.md) — Parent knowledge pack
- [go-io Package](../io/README.md) — I/O Medium interface
- [go-log Package](../log/README.md) — Structured logging
- [CoreGo RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/RFC.md) — Framework specification
- [go-ansible RFC](file:///Users/snider/Code/meowmix/plans/code/core/go/ansible/RFC.md) — Package specification
- [go-ansible AGENTS.md](file:///Users/snider/Code/core/go-ansible/AGENTS.md) — Agent guidance
- [go-ansible CLAUDE.md](file:///Users/snider/Code/core/go-ansible/CLAUDE.md) — Implementation details

