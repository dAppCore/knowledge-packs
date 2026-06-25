---
type: Package Index
title: go-ansible Package Index
description: Complete index of go-ansible package components and features
module: dappco.re/go/ansible
---

# go-ansible — Package Index

> **Repository:** `core/go-ansible`
> **Module:** `dappco.re/go/ansible`
> **Type:** Library
> **Status:** Production

---

## Quick links

- **[README.md](./README.md)** — Complete package documentation
- **[RFC Specification](file:///Users/snider/Code/meowmix/plans/code/core/go/ansible/RFC.md)** — Technical specification
- **[AGENTS.md](file:///Users/snider/Code/core/go-ansible/AGENTS.md)** — Agent guidance
- **[CLAUDE.md](file:///Users/snider/Code/core/go-ansible/CLAUDE.md)** — Implementation details

---

## File structure

### Source Files

| File | Lines | Purpose | Status |
|------|-------|---------|--------|
| `types.go` | 9492 | Core data structures (Playbook, Play, Task, Inventory, etc.) | ✅ Complete |
| `parser.go` | 40490 | YAML parsing for playbooks, inventories, tasks, roles | ✅ Complete |
| `executor.go` | 108148 | Orchestration engine with 68 methods | ✅ Complete |
| `modules.go` | 132306 | 45 module handler implementations | ✅ Complete |
| `ssh.go` | 12551 | SSH client with lazy connection, auth chain, known_hosts | ✅ Complete |
| `local_client.go` | 8992 | Local execution client for connection: local | ✅ Complete |
| `core_primitives.go` | 8688 | Core framework integrations | ✅ Complete |
| `template_features.go` | 9492 | Jinja2-style templating support | ✅ Complete |
| `async_features.go` | 11953 | Async execution support | ✅ Complete |
| `result_helpers.go` | 4049 | Result handling utilities | ✅ Complete |
| `service.go` | 3517 | Service integration | ✅ Complete |

### CLI Files

| File | Purpose |
|------|---------|
| `cmd/ansible/cmd.go` | CLI command registration |
| `cmd/ansible/ansible.go` | Playbook execution |
| `cmd/ansible/core_primitives.go` | Core primitives integration |
| `cmd/ansible/extravars_scalar_test.go` | Extra vars scalar tests |
| `cmd/ansible/command_result_test.go` | Command result tests |
| `cmd/ansible/cmd_test.go` | CLI tests |
| `cmd/ansible/cmd_example_test.go` | CLI examples |
| `cmd/ansible/ansible_test.go` | Ansible command tests |

### Test Files

| File | Purpose | Coverage |
|------|---------|----------|
| `types_test.go` | YAML unmarshalling tests | ✅ Good/Bad/Ugly |
| `parser_test.go` | Playbook/inventory parsing tests | ✅ Good/Bad/Ugly |
| `executor_test.go` | Executor lifecycle, conditions, templating, loops | ✅ Good/Bad/Ugly |
| `ssh_test.go` | SSH client tests | ✅ Good/Bad/Ugly |
| `mock_ssh_test.go` | Mock SSH infrastructure | ✅ Complete |
| `modules_cmd_test.go` | Command modules (shell, command, raw, script) | ✅ Good/Bad/Ugly |
| `modules_file_test.go` | File modules (copy, template, file, etc.) | ✅ Good/Bad/Ugly |
| `modules_svc_test.go` | Service modules (service, systemd, user, group) | ✅ Good/Bad/Ugly |
| `modules_infra_test.go` | Infrastructure modules (apt, pip, git, etc.) | ✅ Good/Bad/Ugly |
| `modules_adv_test.go` | Advanced modules (debug, fail, assert, etc.) | ✅ Good/Bad/Ugly |
| `executor_extra_test.go` | Additional executor tests | ✅ Good/Bad/Ugly |
| `executor_lookup_test.go` | Lookup functionality tests | ✅ Good/Bad/Ugly |
| `local_client_example_test.go` | Local client examples | ✅ Examples |
| `types_example_test.go` | Types examples | ✅ Examples |

### Example Files

| File | Purpose |
|------|---------|
| `types_example_test.go` | Type usage examples |
| `local_client_example_test.go` | Local client examples |

---

## Module catalogue (45 handlers)

### Command Execution (4)

| Short Name | FQCN | Status | Test File |
|------------|------|--------|-----------|
| `shell` | `ansible.builtin.shell` | ✅ Complete | `modules_cmd_test.go` |
| `command` | `ansible.builtin.command` | ✅ Complete | `modules_cmd_test.go` |
| `raw` | `ansible.builtin.raw` | ✅ Complete | `modules_cmd_test.go` |
| `script` | `ansible.builtin.script` | ✅ Complete | `modules_cmd_test.go` |

### File Operations (8)

| Short Name | FQCN | Status | Test File |
|------------|------|--------|-----------|
| `copy` | `ansible.builtin.copy` | ✅ Complete | `modules_file_test.go` |
| `template` | `ansible.builtin.template` | ✅ Complete | `modules_file_test.go` |
| `file` | `ansible.builtin.file` | ✅ Complete | `modules_file_test.go` |
| `lineinfile` | `ansible.builtin.lineinfile` | ✅ Complete | `modules_file_test.go` |
| `stat` | `ansible.builtin.stat` | ✅ Complete | `modules_file_test.go` |
| `slurp` | `ansible.builtin.slurp` | ✅ Complete | `modules_file_test.go` |
| `fetch` | `ansible.builtin.fetch` | ✅ Complete | `modules_file_test.go` |
| `get_url` | `ansible.builtin.get_url` | ✅ Complete | `modules_file_test.go` |

### Package Management (7)

| Short Name | FQCN | Status | Test File |
|------------|------|--------|-----------|
| `apt` | `ansible.builtin.apt` | ✅ Complete | `modules_infra_test.go` |
| `apt_key` | `ansible.builtin.apt_key` | ✅ Complete | `modules_infra_test.go` |
| `apt_repository` | `ansible.builtin.apt_repository` | ✅ Complete | `modules_infra_test.go` |
| `yum` | `ansible.builtin.yum` | ✅ Complete | `modules_infra_test.go` |
| `dnf` | `ansible.builtin.dnf` | ✅ Complete | `modules_infra_test.go` |
| `rpm` | `ansible.builtin.rpm` | ✅ Complete | `modules_infra_test.go` |
| `pip` | `ansible.builtin.pip` | ✅ Complete | `modules_infra_test.go` |
| `package` | `ansible.builtin.package` | ✅ Complete | `modules_infra_test.go` |

### Service Management (4)

| Short Name | FQCN | Status | Test File |
|------------|------|--------|-----------|
| `service` | `ansible.builtin.service` | ✅ Complete | `modules_svc_test.go` |
| `systemd` | `ansible.builtin.systemd` | ✅ Complete | `modules_svc_test.go` |
| `user` | `ansible.builtin.user` | ✅ Complete | `modules_svc_test.go` |
| `group` | `ansible.builtin.group` | ✅ Complete | `modules_svc_test.go` |

### Network & HTTP (1)

| Short Name | FQCN | Status | Test File |
|------------|------|--------|-----------|
| `uri` | `ansible.builtin.uri` | ✅ Complete | `modules_adv_test.go` |

### Control Flow & Facts (9)

| Short Name | FQCN | Status | Test File |
|------------|------|--------|-----------|
| `debug` | `ansible.builtin.debug` | ✅ Complete | `modules_adv_test.go` |
| `fail` | `ansible.builtin.fail` | ✅ Complete | `modules_adv_test.go` |
| `assert` | `ansible.builtin.assert` | ✅ Complete | `modules_adv_test.go` |
| `ping` | `ansible.builtin.ping` | ✅ Complete | `modules_adv_test.go` |
| `set_fact` | `ansible.builtin.set_fact` | ✅ Complete | `modules_adv_test.go` |
| `add_host` | `ansible.builtin.add_host` | ✅ Complete | `modules_adv_test.go` |
| `group_by` | `ansible.builtin.group_by` | ✅ Complete | `modules_adv_test.go` |
| `pause` | `ansible.builtin.pause` | ✅ Complete | `modules_adv_test.go` |
| `wait_for` | `ansible.builtin.wait_for` | ✅ Complete | `modules_adv_test.go` |

### Infrastructure (7)

| Short Name | FQCN | Status | Test File |
|------------|------|--------|-----------|
| `git` | `ansible.builtin.git` | ✅ Complete | `modules_infra_test.go` |
| `unarchive` | `ansible.builtin.unarchive` | ✅ Complete | `modules_infra_test.go` |
| `archive` | `ansible.builtin.archive` | ✅ Complete | `modules_infra_test.go` |
| `hostname` | `ansible.builtin.hostname` | ✅ Complete | `modules_infra_test.go` |
| `sysctl` | `ansible.builtin.sysctl` | ✅ Complete | `modules_infra_test.go` |
| `cron` | `ansible.builtin.cron` | ✅ Complete | `modules_infra_test.go` |
| `blockinfile` | `ansible.builtin.blockinfile` | ✅ Complete | `modules_infra_test.go` |
| `include_vars` | `ansible.builtin.include_vars` | ✅ Complete | `modules_infra_test.go` |
| `meta` | `ansible.builtin.meta` | ✅ Complete | `modules_adv_test.go` |
| `setup` | `ansible.builtin.setup` | ✅ Complete | `modules_adv_test.go` |
| `reboot` | `ansible.builtin.reboot` | ✅ Complete | `modules_adv_test.go` |

### Community Modules (3)

| Short Name | FQCN | Status | Test File |
|------------|------|--------|-----------|
| `ufw` | `community.general.ufw` | ✅ Complete | `modules_infra_test.go` |
| `authorized_key` | `ansible.posix.authorized_key` | ✅ Complete | `modules_svc_test.go` |
| `docker_compose` | `community.docker.docker_compose` | ✅ Complete | `modules_infra_test.go` |

---

## Tags & categories

### Module Categories

- **Command Execution** — 4 modules
- **File Operations** — 8 modules
- **Package Management** — 7 modules
- **Service Management** — 4 modules
- **Network & HTTP** — 1 module
- **Control Flow & Facts** — 9 modules
- **Infrastructure** — 7 modules
- **Community Modules** — 3 modules

### Technology Tags

- `ansible` — Primary tag
- `orchestration` — Deployment automation
- `infrastructure` — Infrastructure management
- `deployment` — Application deployment
- `ssh` — Remote execution
- `yaml` — Configuration format
- `jinja2` — Templating engine

---

## Dependencies

### Internal Dependencies

| Package | Purpose | Import |
|---------|---------|--------|
| `dappco.re/go` | Core framework | `core` |
| `dappco.re/go/log` | Structured logging | `log` |
| `dappco.re/go/io` | I/O Medium interface | `io` |
| `dappco.re/go/string` | String utilities | `string` |
| `dappco.re/go/path` | Path utilities | `path` |
| `dappco.re/go/format` | Formatting | `format` |
| `dappco.re/go/json` | JSON utilities | `json` |
| `dappco.re/go/process` | Process management | `process` |

### External Dependencies

- `golang.org/x/crypto/ssh` — SSH client implementation
- `gopkg.in/yaml.v3` — YAML parsing

---

## Related packages

| Package | Relationship | Location |
|---------|--------------|----------|
| [go-io](../io/README.md) | I/O Medium interface used for playbook loading | `/core/go-io` |
| [go-log](../log/README.md) | Structured logging used throughout | `/core/go-log` |
| [go-cli](../../pkg/cli/README.md) | CLI integration | `/core/cli` |
| [go-devops](../devops/README.md) | DevOps orchestration, uses go-ansible | `/core/go-devops` |

---

## Usage patterns

### Primary Use Cases

1. **Fleet Management** — Managing infrastructure across noc, de1, syd1
2. **Deployment Automation** — Deploying applications and services
3. **Configuration Management** — Ensuring consistent configuration
4. **Provisioning** — Setting up new hosts

### Integration Patterns

1. **Library Mode** — Embed in Go applications
2. **CLI Mode** — Use via `core ansible` command
3. **I/O Medium Mode** — Load playbooks from Git, S3, local files

---

## Maintenance information

- **Author**: Mistral Vibe (Purberus <purberus@lthn.ai>)
- **Created**: 2026-06-17T23:30:00Z
- **Last Updated**: 2026-06-17T23:30:00Z
- **Version**: 1.0.0
- **Licence**: EUPL-1.2
- **Repository**: `forge.lthn.sh/core/go-ansible`


