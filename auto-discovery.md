# Auto-Discovery

Sloth uses a manifest-based auto-discovery system to find your classes automatically. Drop a file in the right directory and Sloth picks it up on the next request — no registration, no configuration.

## How it works

On boot, Sloth scans a set of directories and builds a manifest — a cached PHP file that maps class names to file paths. This manifest is rebuilt whenever files change. The directories scanned are always a pair: `app/{Directory}/` and `theme/{Directory}/`.

## Discovered directories

| Directory | What gets discovered |
|-----------|----------------------|
| `Providers/` | Classes extending `Sloth\Core\ServiceProvider` |
| `Model/` | Classes extending `Sloth\Model\Model` |
| `Taxonomy/` | Classes extending `Sloth\Model\Taxonomy` |
| `Module/` | Classes extending `Sloth\Module\Module` |
| `Api/` | Classes extending `Sloth\Api\Controller` |
| `Context/` | Classes extending `Sloth\Context\ContextProvider` |
| `Includes/` | All `.php` files — loaded via `require_once` on every request |

In Classic mode, Sloth scans both `app/{Directory}/` and `theme/{Directory}/`. In Theme mode, only `theme/{Directory}/` exists, but the scanner handles this gracefully — a missing directory is simply skipped.

## Providers

Service providers are discovered from `app/Providers/` and `theme/Providers/`. Any non-abstract class extending `Sloth\Core\ServiceProvider` is registered automatically.

Framework providers always register first — your providers come after, so you can safely override any framework binding.

See [Service Providers](service-providers) for the full lifecycle.

## Models and Taxonomies

Classes extending `Sloth\Model\Model` are discovered from `app/Model/` and `theme/Model/` and registered as WordPress custom post types automatically. The same applies to taxonomies in `app/Taxonomy/` and `theme/Taxonomy/`.

See [Models](models) and the [Directory Structure](directory-structure) for naming conventions.

## Modules

Module classes are discovered from `app/Module/` and `theme/Module/`. The template is resolved automatically from the class name — `FeaturedProjectsModule` maps to `Module/featured-projects.twig`.

See [Modules](modules) for details.

## API Controllers

Controllers in `app/Api/` and `theme/Api/` are auto-registered as WordPress REST API endpoints under `/wp-json/sloth/v1/`.

See [API Controllers](api-controllers) for details.

## Includes

PHP files in `app/Includes/` and `theme/Includes/` are required on every request via `require_once`. This is a compatibility layer for legacy code that hasn't been migrated to service providers yet.

:::warning
We strongly recommend against using `Includes/` for new code, and encourage migrating existing includes to service providers. PHP files loaded via `Includes/` run outside Sloth's boot lifecycle — hooks registered inside them fire at unpredictable times relative to other providers, which can lead to subtle ordering bugs that are hard to debug.

If your include registers WordPress hooks or filters, move it to a service provider and use `getHooks()` / `getFilters()` instead. This gives Sloth full control over registration order and timing.
:::

## Vendor packages

Sloth also discovers service providers from installed Composer packages. Any package that declares providers in its `composer.json` under `extra.folivoro.providers` is picked up automatically:

```json
{
    "extra": {
        "folivoro": {
            "providers": [
                "MyPackage\\MyServiceProvider"
            ]
        }
    }
}
```

To exclude a package from discovery, add it to `extra.folivoro.dont-discover` in your project's `composer.json`:

```json
{
    "extra": {
        "folivoro": {
            "dont-discover": [
                "vendor/some-package"
            ]
        }
    }
}
```

## Clearing the manifest cache

Manifests are rebuilt automatically when files change during development. In production — where file change detection may be disabled — run this after deploying new classes:

```bash
wp sloth manifest:clear
```

This deletes all cached manifests from `cache/Manifest/`. They are regenerated on the next request.
