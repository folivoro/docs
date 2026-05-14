# Roadmap

This page documents planned changes, open decisions and known issues for upcoming releases. It is intentionally transparent — if you're building on Sloth today, this tells you what's coming and what to expect.

---

## Before 2.0

These items are considered blockers or high priority for the 2.0 release.

### Config system overhaul

The current config system is inconsistent. Legacy projects use `Configure::write()` calls scattered across multiple files with ad-hoc key names. The goal for 2.0 is a clean, namespaced config structure in two files:

```
app/config/
├── app.php     # application-level config
└── theme.php   # theme-level config
```

**Key mapping — old → new:**

| Old | New |
|-----|-----|
| `twig.autoescape` | `theme.twig.autoescape` |
| `theme.image-sizes` | `theme.image_sizes` |
| `theme.menus` | `theme.menus` |
| `layotter_custom_classes` | `theme.layotter.custom_classes` |
| `layotter_prepare_fields` | `theme.layotter.prepare_fields` |
| `urls.relative` | `app.urls.relative` |
| `links.urls.relative` | `app.urls.links.relative` |
| `uploads.urls.relative` | `app.urls.uploads.relative` |
| `wp-json.baseUrl` | `app.wp_json.base_url` |
| `autosync_acf` | `app.acf.autosync` |
| `sloth.acf.process` | `app.acf.process` |

**Deprecated — no replacement:** `core.hide_updates`, `plugins.hide_updates`, `themes.hide_updates`, `theme.routes`, `plugins.autoactivate`, `plugins.autoactivate.blacklist`

### Configure deprecation

`Configure` will be deprecated and removed from `folivoro/sloth`. Until removal it continues to work as a fallback. New code should use `config()` directly.

A Rector rule (`MigrateConfigureToLaravelConfigRector`) will automate the migration.

### folivoro/climb

A standalone migration tool (built with Laravel Zero) that automates the config migration:

```bash
composer require folivoro/climb --dev
php climb modernize
composer remove folivoro/climb
```

`climb` will use PHP parsing (`nikic/php-parser`) to find and rewrite `Configure::write()` calls — including complex cases. For `theme.twig.filters` and `theme.twig.functions` it will warn and point to the new Extension system, as those require manual migration.

### View Extensions — Formatters and Helpers

Currently Twig filters and functions are registered via config arrays — tightly coupled to Twig and awkward to use. The new system uses auto-discovered classes:

```php
// theme/Extensions/View/Formatters/Currency.php
class Currency extends \Sloth\View\Formatter
{
    public function handle(float $value, string $currency = 'EUR'): string
    {
        return number_format($value, 2) . ' ' . $currency;
    }
}

// theme/Extensions/View/Helpers/CurrentYear.php
class CurrentYear extends \Sloth\View\Helper
{
    public function handle(): int
    {
        return (int) date('Y');
    }
}
```

**Formatters** → registered as template filters (`{{ value | currency }}`).
**Helpers** → registered as template functions (`{{ current_year() }}`).

The naming is engine-agnostic — a future Blade driver maps Formatters to directives and Helpers to Blade helpers without any changes to theme code.

### ~~feat/context~~ — done ✓

Context is now lazy and extensible. See [Context](context) for details.

`Context.php` will be rebuilt as a lazy, extensible system. Currently all context values are resolved eagerly on every request — even if the template never uses them.

- `LazyContext` — `ArrayAccess` + `IteratorAggregate`, resolves values on access
- `ContextProvider` base class
- `Context::register(string $key, callable $resolver)` API
- Auto-Discovery in `app/Context/` and `theme/Context/`
- All existing keys (`site.*`, `globals.*`, `post`, `taxonomy`, `author`, `options`) migrated to lazy providers
- `getContext()` returns `LazyContext` instead of `array` — backwards compatible

### feat/options

Clean Options abstraction over WordPress Core `get_option()` and ACF Options. No Illuminate overhead — WordPress' own object cache is sufficient.

- `OptionsProxy` — `get()`, `set()`, `delete()`, `has()`
- Automatic fallback: ACF `get_field($key, 'option')` when available, otherwise `get_option()`
- `OptionsServiceProvider` — binds `options` in the container
- `Options` Facade
- Registered as `options` key in LazyContext
- Twig: `{{ options.blogname }}`, `{{ options.primary_color }}`

### feat/assets

Vite Manifest Reader, `asset()` helper and WordPress `wp_enqueue_*` integration. `URL::asset()` is already prepared.

- `AssetsServiceProvider` — reads `public/manifest.json`
- `Manifest.php` — resolves hashed filenames
- `wp_enqueue_scripts` integration
- HMR support for development (Vite Dev Server)

---

## Planned packages

### folivoro/sloth-layotter

Extract `src/LayotterBridge/` into its own package.

- New repo `folivoro/sloth-layotter`
- Remove hardcoded `LayotterBridgeServiceProvider` from `Application.php`
- Publish on Packagist

### folivoro/acf-flexible-content

Port `FlexibleContentRenderer` + `ServiceProvider` from koalapress.

- Replace custom `ClassResolver` with Sloth's Manifest system
- Optional ACFE (ACF Extended) support

### folivoro/coding-standards

Shared `php-cs-fixer.php` + `phpstan.neon` base config for all folivoro packages. Analogous to `laravel/pint`.

---

## Code quality

### PHPStan

Currently at Level 1. Target is Level 5 on the full codebase without a baseline.

- Level 1 → 3: address new errors or justify suppressions
- Level 3 → 5: full coverage
- Include `src/Model` in analysis after Corcel removal
- Remove `src/Facades` from `excludePaths`, add `@mixin` to all facades
- Add Layotter PHPStan stubs

### Corcel removal

The `refactor/remove-corcel` branch (12 commits, tests present) is ready for review. `MenuItem.php` is the main blocker — estimated 2-3 days of work.

### Rector rules

- `NormalizeSlothRegistrationPropertiesRector`
- `MigrateConfigureToLaravelConfigRector`

### Facade audit

Several facades have zero usages in the framework itself — `Pagination`, `Module`, `Menu`. These will either be documented with clear use cases or removed.

---

## Versioning

| Version | After |
|---------|-------|
| `v1.1.0` | `feat/context` + `feat/options` |
| `v1.2.0` | `feat/assets` + koalapress migration possible |
| `v2.0.0` | Config overhaul, `Configure` removed, Extension system, package rename |
