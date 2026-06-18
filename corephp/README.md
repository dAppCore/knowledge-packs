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

> **"The CorePHP module framework"** — CorePHP RFC

**Module:** `dappco.re/php/*`
**Repository:** `core-php` + `core-{module}` per module

CorePHP is the PHP framework for the Lethean ecosystem, providing a modern, modular approach with Laravel-like conventions. **Everything is a module. Every module is self-contained. Every module has one `Boot.php`.**

---

## 🎯 Overview

From the [CorePHP RFC](file:///Users/snider/Code/meowmix/plans/code/core/php/RFC.md):

**CorePHP** provides the framework architecture for PHP development in the Lethean ecosystem.

**Key principles:**
- Everything is a module
- Every module is self-contained
- Every module has one `Boot.php` (ServiceProvider entry point)
- Modules are only instantiated when their events fire — **lazy loading**

### Module Architecture

```
app/Mod/{ModuleName}/
├── Boot.php              # Single ServiceProvider entry point
├── Console/              # Artisan Commands
├── Controllers/          # HTTP Controllers (Web & API)
├── Database/             # Migrations, Factories, Seeders
├── Events/               # Events & Listeners
├── Jobs/                 # Queueable Jobs
├── Mcp/                  # MCP Tools & Servers
├── Models/               # Eloquent Models (Namespace: Mod\{Mod}\Models)
├── Services/             # Business Logic
├── Tests/                # Pest Tests
└── View/                 # UI Layer (MVVM)
    ├── Modal/            # Livewire Components (ViewModels)
    │   ├── Admin/        # Admin-facing
    │   └── Web/          # Public-facing
    ├── Blade/            # Dumb templates (no logic)
    │   ├── admin/
    │   └── web/
    ├── Components/       # Reusable Blade Components
    └── routes/           # Route definitions
```

### Event-Driven Boot

```php
class Boot extends ServiceProvider
{
    public static array $listens = [
        AdminPanelBooting::class => 'onAdminPanel',
        WebRoutesRegistering::class => 'onWebRoutes',
        ApiRoutesRegistering::class => 'onApiRoutes',
        ConsoleBooting::class => 'onConsole',
        McpToolsRegistering::class => 'onMcpTools',
    ];
}
```

Modules are only instantiated when their events fire — **lazy loading**.

### MVVM View Protocol (Modern Flexy)

- **Controller** = traffic cop (delegates, never touches HTML)
- **View Modal** = the interface (Livewire Component — formats, prepares, manages state)
- **Template** = dumb (HTML + simple interpolation, no `@php` blocks, no DB queries)

**Rule:** Modules provide DATA, Core provides RENDERING.

### Three-Tier Frontend

```
Core/Front/Web    → Public (anonymous, read-only)
Core/Front/Client → SaaS customer (authenticated, namespace owner)
Core/Front/Admin  → Backend admin (privileged)
Core/Hub          → SaaS operator (Host.uk.com control plane)
```

**Namespace** = identity, tied to URI/handle (lt.hn/you, you.lthn).
**Workspace** = management container (org that can own multiple namespaces).

### Sub-Specs

The CorePHP RFC includes these sub-specifications:
- [Architecture](file:///Users/snider/Code/meowmix/plans/code/core/php/RFC.architecture.md)
- [Commands](file:///Users/snider/Code/meowmix/plans/code/core/php/RFC.commands.md)
- [Endpoints](file:///Users/snider/Code/meowmix/plans/code/core/php/RFC.endpoints.md)
- [Events](file:///Users/snider/Code/meowmix/plans/code/core/php/RFC.events.md)
- [Models](file:///Users/snider/Code/meowmix/plans/code/core/php/RFC.models.md)
- [Patterns](file:///Users/snider/Code/meowmix/plans/code/core/php/RFC.patterns.md)
- [Release](file:///Users/snider/Code/meowmix/plans/code/core/php/RFC.release.md)
- [Scaffolding](file:///Users/snider/Code/meowmix/plans/code/core/php/RFC.scaffolding.md)
- [UI](file:///Users/snider/Code/meowmix/plans/code/core/php/RFC.ui.md)

### Key Statistics

- **Repository:** `forge.lthn.sh/core/php`
- **Module:** `dappco.re/php/*`
- **PHP Version:** 8.2+
- **Total Repos:** 25+ (php-* ecosystem)
- **Framework:** Custom (Laravel-inspired)
- **Testing:** Pest PHP framework
- **Total Modules:** 25+ (see INDEX.md)

### Component Pattern

- **Anonymous** (no PHP class) — pure presentation (`<admin:panel>`, `<admin:nav-item>`)
- **Class-backed** — when logic needed (`<admin:data-table>`, `<admin:sidemenu>`)

**Rule:** Modules provide DATA, Core provides RENDERING.

### Table Naming Convention

`[module_snake_case]_[table_name]` — e.g., `agentic_plans`, `social_posts`, `analytics_visitors`.

### Enforcement Rules

1. No Eloquent in Blade — template uses View Modal getters
2. No `@php` blocks with business logic
3. Modules don't query other modules' tables directly — use Service contracts or Events
4. A module should be extractable as a Composer package with minimal effort

### Shared Services

Cross-cutting services available to all modules:

| Service | Purpose |
|---------|---------|
| `DeviceDetectionService` | Device type, OS, browser, bot detection, in-app browser detection (15 platforms) |
| `GeoIpService` | IP geolocation from CDN headers or MaxMind |
| `PrivacyHelper` | IP anonymisation and hashing |
| `UtmHelper` | UTM parameter extraction |

#### In-App Browser Detection

Detects social media in-app browsers (Instagram, TikTok, Facebook, Twitter, Snapchat, LinkedIn, Threads, Pinterest, Reddit, WeChat, LINE, Telegram, Discord, WhatsApp).

```php
$dd = app(DeviceDetectionService::class);
$dd->isInAppBrowser($ua);          // any in-app browser
$dd->isStrictContentPlatform($ua); // platforms that enforce content policies
$dd->isMetaPlatform($ua);          // Instagram, Facebook, Threads
$dd->getPlatformDisplayName($ua);  // "Instagram", "TikTok", etc.
```

Used by BioHost for 18+ content warnings when accessed from strict platforms.

---

### Inter-Module Communication

Modules communicate via Laravel events — **lifecycle events** (`WebRoutesRegistering`, `AdminPanelBooting`, etc.) for route/panel registration, and **domain events** (`DomainResolving`) for cross-module data flows.

```php
// Fire a domain event (not lifecycle)
event(new MemoryStored($memory));

// Any module can listen
MemoryStored::class => [UpdateTrainingData::class, TrackRecallLatency::class]
```

Shared event types in `Core\Events\` namespace:

```php
// src/Core/Events/DataEvents.php
namespace Core\Events;

class MemoryStored { public function __construct(public BrainMemory $memory) {} }
class MemoryRecalled { public function __construct(public string $query, public array $results) {} }
class WorkspaceCreated { public function __construct(public Workspace $workspace) {} }
class SubscriptionActivated { public function __construct(public Workspace $workspace, public Plan $plan) {} }
class ApiRequestCompleted { public function __construct(public string $endpoint, public float $duration) {} }
```

---

### Testing Strategy

Tests use **Pest** with Orchestra Testbench and in-memory SQLite. Tests are co-located in `src/Core/{Package}/Tests/` and `src/Mod/{Module}/Tests/`.

CI matrix covers PHP 8.2/8.3/8.4 × Laravel 11/12.

Every module has:
- Unit tests for Actions and Services
- Feature tests for API endpoints (happy path, 401, 403, 422)
- Livewire component tests
- Migration tests (up + down)

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

## 💻 How-Tos

### 1. Creating a New Module

```bash
# Create module directory
mkdir -p app/Mod/MyModule

# Create Boot.php
cat > app/Mod/MyModule/Boot.php <<EOF
<?php

namespace App\Mod\MyModule;

use Core\ServiceProvider;
use Core\Events\WebRoutesRegistering;

class Boot extends ServiceProvider
{
    public static array \$listens = [
        WebRoutesRegistering::class => 'onWebRoutes',
    ];
    
    public function onWebRoutes(WebRoutesRegistering \$event): void
    {
        // Register routes
    }
}
EOF
```

### 2. Creating a Livewire Component

```php
// app/Mod/MyModule/Modal/Web/MyComponent.php
namespace App\Mod\MyModule\Modal\Web;

use Livewire\Component;

class MyComponent extends Component
{
    public string \$name = '';
    
    public function render()
    {
        return view('mymodule::web.my-component');
    }
}
```

### 3. Using Device Detection

```php
// In a controller or service
\$dd = app(DeviceDetectionService::class);

if (\$dd->isInAppBrowser(request()->userAgent())) {
    // Show warning for in-app browser
    return view('warnings.in-app-browser');
}

if (\$dd->isStrictContentPlatform(request()->userAgent())) {
    // Show 18+ content warning
    return view('warnings.strict-platform');
}
```

### 4. Firing Domain Events

```php
// In a service
use Core\Events\MemoryStored;

event(new MemoryStored($memory));

// Listeners are registered in Boot.php
MemoryStored::class => [
    UpdateTrainingData::class,
    TrackRecallLatency::class,
]
```

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

## 📖 Quick Reference

### Module Structure

| Directory | Purpose | Namespace |
|-----------|---------|-----------|
| `Boot.php` | ServiceProvider entry | `App\Mod\{Module}` |
| `Console/` | Artisan Commands | `App\Mod\{Module}\Console` |
| `Controllers/` | HTTP Controllers | `App\Mod\{Module}\Controllers` |
| `Database/` | Migrations, Factories | `App\Mod\{Module}\Database` |
| `Events/` | Events & Listeners | `App\Mod\{Module}\Events` |
| `Jobs/` | Queueable Jobs | `App\Mod\{Module}\Jobs` |
| `Mcp/` | MCP Tools | `App\Mod\{Module}\Mcp` |
| `Models/` | Eloquent Models | `App\Mod\{Module}\Models` |
| `Services/` | Business Logic | `App\Mod\{Module}\Services` |
| `Tests/` | Pest Tests | `App\Mod\{Module}\Tests` |
| `View/` | UI Layer | `App\Mod\{Module}\View` |

### Frontend Tiers

| Tier | Audience | Access |
|------|----------|--------|
| `Core/Front/Web` | Public | Anonymous, read-only |
| `Core/Front/Client` | SaaS customer | Authenticated, namespace owner |
| `Core/Front/Admin` | Backend admin | Privileged |
| `Core/Hub` | SaaS operator | Host.uk.com control plane |

### Device Detection Methods

| Method | Returns | Example |
|--------|---------|---------|
| `isInAppBrowser($ua)` | `bool` | Detect any in-app browser |
| `isStrictContentPlatform($ua)` | `bool` | Strict content policy platforms |
| `isMetaPlatform($ua)` | `bool` | Instagram, Facebook, Threads |
| `getPlatformDisplayName($ua)` | `string` | "Instagram", "TikTok" |

### Testing Strategy

| Test Type | Purpose | Location |
|-----------|---------|----------|
| Unit tests | Actions and Services | `Tests/Unit/` |
| Feature tests | API endpoints | `Tests/Feature/` |
| Livewire tests | Component tests | `Tests/Livewire/` |
| Migration tests | Database migrations | `Tests/Migration/` |

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
