# Upgrade Guide

## Upgrading to 2.0

Sloth 2.0 introduces breaking changes as part of a major cleanup. This guide covers what's changing and how to migrate.

### Package rename

The package has moved from `sixmonkey/sloth` to `folivoro/sloth`:

```bash
composer remove sixmonkey/sloth
composer require folivoro/sloth
```

### PHP requirement

Minimum PHP version is now **8.4**. Ensure your server meets this requirement before upgrading.

### Config system

The legacy `Configure::write()` system is deprecated. Configuration now lives in two files:

```
app/config/app.php    # application config
app/config/theme.php  # theme config
```

Run the modernize command to migrate automatically:

```bash
wp sloth modernize
```

See the [Roadmap](roadmap) for the full config key mapping.

### Twig Extensions

`Configure::write('theme.twig.filters', [...])` and `Configure::write('theme.twig.functions', [...])` are no longer supported. Migrate to the new Extension system:

```php
// Before — in functions.php or a config file
Configure::write('theme.twig.filters', [
    new TwigFilter('currency', fn($v) => format($v)),
]);

// After — theme/Extensions/View/Formatters/Currency.php
class Currency extends \Sloth\View\Formatter
{
    public function handle(float $value): string
    {
        return format($value);
    }
}
```

See [Views](views) for details on the new Extension system.

### WP_ENV

`local` is now the recommended value for local development. `development`, `develop` and `dev` continue to work for backwards compatibility.

### Removed

- `Configure` facade — use `config()` directly
- `InspireCommand` — moved to the starters
- `bootstrap.php` — dotenv is now loaded automatically by Sloth

### View layer restructure

The View layer has been reorganised. If you referenced internal View classes directly, update your imports:

| Old | New |
|-----|-----|
| `Sloth\View\Engines\TwigEngine` | `Sloth\View\Engines\Twig\Engine` |
| `Sloth\View\Extensions\SlothTwigExtension` | `Sloth\View\Engines\Twig\Extension` |
| `Sloth\View\Formatter` | `Sloth\View\Extensions\Formatter` |
| `Sloth\View\Helper` | `Sloth\View\Extensions\Helper` |

These are internal framework classes — if you extended them directly, consider using the new auto-discovery system via `theme/Extensions/View/Formatters/` and `theme/Extensions/View/Helpers/` instead.
