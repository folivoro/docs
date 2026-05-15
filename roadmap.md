# Roadmap

This page documents planned changes, open decisions and known issues for upcoming releases. It is intentionally transparent — if you're building on Sloth today, this tells you what's coming and what to expect.

---

## Before 2.0

These items are considered blockers or high priority for the 2.0 release.

### feat/assets

Vite Manifest Reader, `asset()` helper and WordPress `wp_enqueue_*` integration.

- `AssetsServiceProvider` — reads `public/manifest.json`
- `Manifest.php` — resolves hashed filenames
- `wp_enqueue_scripts` integration
- HMR support for development (Vite Dev Server)
- `asset()` helper available in Twig and PHP

### Corcel removal

Replace `jgrossi/corcel` with a native Sloth implementation built directly on `illuminate/database`. Corcel is no longer actively maintained and conflicts with PHP 8.4. See the [GitHub issue](https://github.com/folivoro/sloth/issues) for full details.

The `refactor/remove-corcel` branch has a start. `MenuItem.php` is the main blocker — estimated 2-3 days of work.

### PHPStan level increase

Currently at Level 1. Corcel removal unblocks this significantly — Corcel's untyped properties are the main source of suppressions.

Target: Level 5 on the full codebase without a baseline.

---

## Planned

### Blade support

`BladeAdapter` for projects that prefer Blade over Twig. Engine-agnostic `AbstractViewExtension` system already supports this.

### folivoro/climb

A standalone migration tool (built with Laravel Zero) that automates the upgrade from 1.x to 2.0:

```bash
composer require folivoro/climb --dev
wp sloth modernize
composer remove folivoro/climb
```

`climb` uses PHP parsing (`nikic/php-parser`) to find and rewrite legacy code. For `theme.twig.filters` and `theme.twig.functions` it will warn and point to the new Extension system, as those require manual migration.

### folivoro/acf-flexible-content

Port `FlexibleContentRenderer` + `ServiceProvider` from koalapress.

- Replace custom `ClassResolver` with Sloth's Manifest system
- Optional ACFE (ACF Extended) support

### folivoro/coding-standards

Shared `php-cs-fixer.php` + `phpstan.neon` base config for all folivoro packages. Analogous to `laravel/pint`.

---

## Done in 2.0

- ✅ Config system overhaul — `Configure` removed, `app.php` / `theme.php` / `admin.php`
- ✅ View Extension system — `AbstractViewExtension`, `getHelpers()`, `getDirectives()`, `share()`
- ✅ `TwigAdapter` — engine-agnostic view layer
- ✅ `feat/context` — `LazyContext`, auto-discovery, `ContextProvider`
- ✅ `feat/options` — `Options` facade, ACF fallback
- ✅ `ApplicationPathTrait` — typed path accessors, `isThemeMode()`
- ✅ `UrlServiceProvider` — relative URL handling extracted from Media
- ✅ `make:*` commands — `make:module`, `make:model`, `make:provider`, `make:extension`, `make:command`, `make:api-controller`, `make:layout`
- ✅ `stub:publish` — publishable stubs
- ✅ `folivoro/layotter-bridge` — extracted as standalone package
- ✅ Package rename — `sixmonkey/sloth` → `folivoro/sloth`
- ✅ PHP 8.4 minimum

---

## Versioning

| Version | Milestone |
|---------|-----------|
| `v2.0.0` | Config overhaul, Extension system, package rename, `folivoro/layotter-bridge` |
| `v2.1.0` | `feat/assets`, Blade support |
| `v2.2.0` | Corcel removal, PHPStan Level 5 |
