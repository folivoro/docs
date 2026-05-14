# Introduction

Sloth is a polymorphic WordPress application framework. It brings Laravel patterns — Dependency Injection, Service Providers, Facades, Eloquent, Events, Cache — into WordPress, without requiring the full Laravel stack.

The word *polymorphic* is deliberate: Sloth adapts to how your project is structured, rather than enforcing a fixed directory layout.

## Two modes

### Classic mode

The application lives outside the theme. A shared `app/` directory holds models, APIs and business logic that are independent of any theme. The WordPress theme handles presentation only.

Bootstrapping happens via [Cecropia](https://github.com/folivoro/cecropia), a must-use plugin that boots Sloth before WordPress loads any theme.

Use this mode when you want a clean separation between application logic and theme — or when multiple themes share the same business logic.

**Starter:** `folivoro/folivoro`

### Theme mode

Everything lives inside the theme. Sloth is bootstrapped directly in `functions.php`.

Use this mode when you're delivering a standalone theme — a simpler setup with no dependencies outside the theme directory.

**Starter:** `folivoro/theme`

## Installation

### With a starter (recommended)

The fastest way to get started. The starters set up the full directory structure, environment configuration and bootstrapping for you.

**Classic mode:**

```bash
composer create-project folivoro/folivoro my-project
```

**Theme mode:**

```bash
composer create-project folivoro/theme my-theme
```

Place the theme in `wp-content/themes/` and activate it in the WordPress admin.

### Without a starter

**Theme mode** — add Sloth to an existing theme:

```bash
composer require folivoro/sloth
```

Then bootstrap in `functions.php`:

```php
use Sloth\Core\Application;

Application::configure(basePath: __DIR__)->boot();
```

**Classic mode** — add Sloth and Cecropia to an existing WordPress project:

```bash
composer require folivoro/sloth
composer require folivoro/cecropia
```

Cecropia has type `wordpress-muplugin` and will be placed automatically if your project uses Composer installer paths. If not, copy `cecropia.php` manually to `wp-content/mu-plugins/`.

### Requirements

- PHP 8.4 or higher
- WordPress 5.0 or higher
- Composer 2.0 or higher

## How Sloth detects the mode

Pass the base path explicitly — no guessing needed:

```php
// Theme mode — __DIR__ is the theme root = App-Root
Application::configure(basePath: __DIR__)->boot();

// Classic mode — explicit App-Root
Application::configure(basePath: __DIR__ . '/app')->boot();
```

If no `basePath` is passed, Sloth resolves the App-Root automatically:

1. `SLOTH_BASE_PATH` constant — must be defined before WordPress bootstraps, used as-is
2. Walk up from `ABSPATH` until a `composer.json` outside `vendor/` is found → **Classic mode** → App-Root = `${project}/app/`
3. Fall back to `get_template_directory()` → **Theme mode** → App-Root = theme directory

In both modes, all auto-discovered classes (`Model/`, `Module/`, `Providers/` etc.) and `storage/` live relative to the App-Root.

## Environment

Both modes use a `.env` file for environment-specific configuration:

```env
WP_ENV=local
WP_HOME=https://my-project.test
WP_SITEURL=https://my-project.test/cms

DB_NAME=my_project
DB_USER=root
DB_PASSWORD=
DB_HOST=localhost

APP_SECRET=a-long-random-string
```

`WP_ENV` controls the environment. Use `local` for local development — `development`, `develop` and `dev` are also accepted for backwards compatibility. Anything else is treated as production. Use `testing` when running tests.

## Next steps

- [Quick Start](quick-start) — create your first model, route and module
- [Directory Structure](directory-structure) — what goes where in Classic and Theme mode
- [Service Providers](service-providers) — how Sloth bootstraps and how to hook in
