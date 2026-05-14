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

---

### Config system overhaul

All legacy `Configure::write()` keys have been replaced with namespaced Laravel config files. Publish the new config files first:

```bash
wp sloth vendor:publish --tag=config
```

This creates `config/app.php`, `config/theme.php`, `config/admin.php` and others in your project.

#### Key mapping

| Old (`Configure::write`) | New (`config/`) | File |
|--------------------------|-----------------|------|
| `twig.autoescape` | `view.autoescape` | `config/view.php` |
| `theme.twig.filters` | — | Use [View Extensions](views#extending-twig) |
| `theme.twig.functions` | — | Use [View Extensions](views#extending-twig) |
| `theme.image-sizes` | `theme.image_sizes` | `config/theme.php` |
| `theme.menus` | `theme.menus` | `config/theme.php` |
| `theme.routes` | — | Use [Routes](routing) |
| `theme.layotter.row_layouts` | `layotter.row_layouts` | `config/layotter.php` |
| `layotter_prepare_fields` | `layotter.prepare_fields` | `config/layotter.php` |
| `layotter_custom_classes` | `layotter.custom_classes` | `config/layotter.php` |
| `autosync_acf` | — | No longer part of Sloth |
| `sloth.acf.process` | `theme.process_acf` | `config/theme.php` |
| `urls.relative` | `app.relative_urls` | `config/app.php` |
| `links.urls.relative` | `app.relative_links` | `config/app.php` |
| `uploads.urls.relative` | `app.relative_uploads` | `config/app.php` |
| `wp-json.baseUrl` | `app.wp_json.base_url` | `config/app.php` |
| `core.hide_updates` | `admin.hide_updates.core` | `config/admin.php` |
| `plugins.hide_updates` | `admin.hide_updates.plugins` | `config/admin.php` |
| `themes.hide_updates` | `admin.hide_updates.themes` | `config/admin.php` |
| `plugins.autoactivate` | — | No longer part of Sloth |
| `plugins.autoactivate.blacklist` | — | No longer part of Sloth |

#### Before

```php
// functions.php or app/config/config.php
Configure::write('theme.menus', [
    'primary' => 'Primary Navigation',
    'footer'  => 'Footer Navigation',
]);

Configure::write('theme.image-sizes', [
    'hero' => [1920, 1080, true],
]);

Configure::write('urls.relative', true);
```

#### After

```php
// config/theme.php
return [
    'menus' => [
        'primary' => 'Primary Navigation',
        'footer'  => 'Footer Navigation',
    ],
    'image_sizes' => [
        'hero' => [1920, 1080, true],
    ],
];

// config/app.php
return [
    'relative_urls' => true,
];
```

#### Theme supports

`add_theme_support()` calls in `functions.php` move to `config/theme.php`:

```php
// Before — functions.php
add_action('after_setup_theme', function() {
    add_theme_support('title-tag');
    add_theme_support('post-thumbnails');
});

// After — config/theme.php
return [
    'supports' => [
        'title-tag',
        'post-thumbnails',
        'html5' => ['search-form', 'comment-form', 'gallery'],
    ],
];
```

#### Twig filters and functions

`Configure::write('theme.twig.filters', [...])` and `Configure::write('theme.twig.functions', [...])` are replaced by the View Extension system:

```php
// Before
Configure::write('theme.twig.filters', [
    new TwigFilter('format_phone', fn($n) => preg_replace('/[^0-9\+]/', '', $n)),
]);

// After — theme/Extensions/View/PhoneExtension.php
class PhoneExtension extends \Sloth\View\Extensions\AbstractViewExtension
{
    public function getHelpers(): array
    {
        return [
            'format_phone' => fn($n) => preg_replace('/[^0-9\+]/', '', $n),
        ];
    }
}
```

:::tip folivoro/climb
`folivoro/climb` will warn about `theme.twig.filters` and `theme.twig.functions` entries but cannot migrate them automatically — the callbacks contain business logic that requires manual review.
:::

#### `Configure` is deprecated

`Configure::write()` and `Configure::read()` still work but trigger deprecation warnings and will be removed in a future release. Run `wp sloth modernize` to migrate automatically (requires `folivoro/climb`).

---

### `.env` location

Sloth now walks up from the App-Root to find `.env` automatically. In Classic mode, `.env` should live next to `composer.json` in the **project root** — not inside `app/`:

```
# Classic mode
my-project/
├── .env           ← here
├── composer.json
└── app/

# Theme mode — unchanged
my-theme/
├── .env           ← here, next to composer.json
└── composer.json
```

---

### View layer restructure

Internal View classes have moved. If you extended them directly, update your imports:

| Old | New |
|-----|-----|
| `Sloth\View\Engines\TwigEngine` | `Sloth\View\Engines\Twig\TwigEngine` |
| `Sloth\View\Extensions\SlothTwigExtension` | `Sloth\View\Engines\Twig\TwigExtension` |
| `Sloth\View\Formatter` | `Sloth\View\Extensions\AbstractViewExtension` |
| `Sloth\View\Helper` | `Sloth\View\Extensions\AbstractViewExtension` |

These are internal framework classes. Use the new [View Extension](views#extending-twig) auto-discovery system instead.

---

### `storage/` directory

Cache and log files now live in `storage/` relative to the App-Root:

```
# Before
theme/cache/
theme/logs/

# After
theme/storage/cache/
theme/storage/logs/
```

Update your `.gitignore`:

```diff
- /cache/
- /logs/
+ /storage/
```

---

### WP_ENV

`local` is now the recommended value for local development. `development`, `develop` and `dev` continue to work for backwards compatibility.

---

### Removed

- `InspireCommand`
- `bootstrap.php` — dotenv is now loaded automatically
- `theme.routes` config — use [Routes](routing) instead
- `plugins.autoactivate` config — no longer part of Sloth
- `autosync_acf` config — no longer part of Sloth
