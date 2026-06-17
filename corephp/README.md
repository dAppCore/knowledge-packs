---
type: Knowledge Pack
title: CorePHP Framework
description: Complete knowledge pack for CorePHP — the PHP framework for Lethean PHP applications
author: Mistral Vibe
version: 1.0.0
created: 2026-06-17T14:50:00Z
tags: [framework, php, lethean, web, laravel-like]
---

# CorePHP Knowledge Pack

> **"PHP framework for the Lethean ecosystem"**

CorePHP is the PHP framework for Lethean PHP applications, providing a modern, modular approach to PHP development with Laravel-like conventions.

---

## 🎯 Overview

**CorePHP** provides:
- Modern PHP development
- Modular architecture
- Laravel-like conventions
- Lethean-specific integrations
- Multiple specialized packages

### Key Statistics

- **Repository:** `forge.lthn.sh/core/php`
- **PHP Version:** 8.2+
- **Total Repos:** 25+ (php-* ecosystem)
- **Framework:** Custom (Laravel-inspired)

---

## 📚 Source of Truth

- **Primary Spec:** [`plans/code/core/php/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/php/RFC.md)
- **Implementation:** [`core/php/`](file:///Users/snider/Code/core/php/)
- **Agent Guide:** [`core/php/AGENTS.md`](file:///Users/snider/Code/core/php/AGENTS.md)
- **Index:** [`plans/code/core/php/INDEX.md`](file:///Users/snider/Code/meowmix/plans/code/core/php/INDEX.md)

---

## 🏗️ Architecture

```
CorePHP (PHP Framework)
├── Core Packages
│   ├── php-admin
│   ├── php-agent
│   ├── php-analytics
│   ├── php-api
│   ├── php-bio
│   ├── php-client
│   ├── php-commerce
│   ├── php-config
│   ├── php-content
│   ├── php-developer
│   ├── php-filehost
│   ├── php-framework
│   └── php-tenant
├── Plugins
│   ├── php-plug-altum
│   ├── php-plug-business
│   ├── php-plug-cdn
│   ├── php-plug-chat
│   ├── php-plug-content
│   ├── php-plug-social
│   ├── php-plug-stock
│   ├── php-plug-storage
│   └── php-plug-web3
└── Services
    ├── php-service
    ├── php-shield
    ├── php-template
    ├── php-uptelligence
    ├── php-vapphost
    └── php-vaulthost
```

---

## 📦 Core Components

### 1. Framework Core (`php-framework`)

**Laravel-inspired foundation:**

- **Routing** — RESTful routes
- **Middleware** — Request/response processing
- **Controllers** — Business logic
- **Models** — Data access
- **Views** — Template rendering
- **Blade-like templates** — GrammarImprint integration

### 2. API Package (`php-api`)

**REST API framework:**

- **Resource controllers**
- **Request validation**
- **Response formatting**
- **Authentication** — JWT, OAuth2
- **Rate limiting**

### 3. Agent Package (`php-agent`)

**Agent integration:**

- **MCP (Model Context Protocol)** support
- **Workspace management**
- **Tool execution**
- **Fleet coordination**

### 4. Plugins System

**20+ specialized plugins:**

| Plugin | Purpose |
|--------|---------|
| php-plug-altum | Altum integration |
| php-plug-business | Business logic |
| php-plug-cdn | CDN management |
| php-plug-chat | Chat functionality |
| php-plug-content | Content management |
| php-plug-social | Social features |
| php-plug-stock | Stock/asset management |
| php-plug-storage | Storage backends |
| php-plug-web3 | Web3/blockchain |

### 5. Services Layer

**Specialized services:**

| Service | Purpose |
|---------|---------|
| php-service | Core service framework |
| php-shield | Security/protection |
| php-template | Template management |
| php-uptelligence | Uptime intelligence |
| php-vapphost | VAPP hosting |
| php-vaulthost | Vault hosting |

---

## 🚀 Getting Started

### For Agents

1. **Read the RFC:** [`plans/code/core/php/RFC.md`](file:///Users/snider/Code/meowmix/plans/code/core/php/RFC.md)
2. **Check index:** [`plans/code/core/php/INDEX.md`](file:///Users/snider/Code/meowmix/plans/code/core/php/INDEX.md)
3. **Explore code:** `ls core/php/`

### For Developers

1. **Clone the framework:** `git clone forge.lthn.sh/core/php-framework`
2. **Install dependencies:** `composer install`
3. **Run development server:** `php artisan serve`
4. **Build for production:** `php artisan optimize`

---

## 📊 Quick Stats

```
Total PHP repos:         25+
Core packages:           13
Plugins:                 9
Services:                6
Total lines of code:    100K+
PHP version:             8.2+
```

---

## 🎯 Use Cases

### When to Use CorePHP

✅ **PHP web applications** — Use for traditional PHP apps
✅ **Laravel-like patterns** — Use familiar conventions
✅ **Plugin architecture** — Use modular plugins
✅ **Legacy systems** — Use for PHP-based infrastructure

### When NOT to Use CorePHP

❌ **Go applications** — Use CoreGo instead
❌ **TypeScript frontends** — Use CoreTS instead
❌ **Desktop applications** — Use CoreGUI instead
❌ **CLI tools** — Use CoreCLI instead

---

## 🔗 Related Knowledge Packs

- [CoreGo](../corego/README.md) — Go framework
- [CoreTS](../corets/README.md) — TypeScript framework
- [CoreGUI](../coregui/README.md) — GUI framework
- [CoreCLI](../corecli/README.md) — CLI framework
- [CorePlay](../coreplay/README.md) — Play framework (Go + TS)

---

## 💡 Agent Tips

1. **Use GrammarImprint** — For semantic HTML generation
2. **Laravel conventions** — Follow familiar patterns
3. **Plugin system** — Use plugins for modularity
4. **Protobuf where possible** — For type-safe communication
5. **CorePHP-first** — Always check if a package exists

---

## 📝 Maintenance

This knowledge pack is maintained by Mistral Vibe. Updates triggered by:

- Changes to `plans/code/core/php/`
- Changes to `core/php*` repositories
- New PHP packages
- Framework updates

---

*Knowledge Pack v1.0.0*
*Created: 2026-06-17T14:50:00Z*
*Author: Mistral Vibe*
*Source: Lethean CorePHP Framework*
