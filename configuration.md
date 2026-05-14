# Configuration

Sloth uses Laravel's config system. Config files live in `config/` and are loaded automatically on boot.

:::warning Deprecated
`Configure::write()` and `Configure::read()` are deprecated as of Sloth 2.0 and will be removed in a future release. Use `config()` and publishable config files instead. See the [upgrade guide](upgrade#config-system-overhaul).
:::

## Config files

Publish Sloth's built-in config files to your project:

```bash
wp sloth vendor:publish --tag=config
```

This creates:

```
config/
├── app.php        # Relative URLs, WP JSON
├── theme.php      # Menus, image sizes, theme supports
├── admin.php      # Hide updates, admin footer
├── cache.php      # Cache driver and prefix
├── debugger.php   # DebugBar settings
├── events.php     # Event bridge settings
└── layotter.php   # Layotter bridge settings
```

Add your own config files as needed — any file in `config/` is loaded automatically:

```php
// config/payment.php
return [
    'provider' => env('PAYMENT_PROVIDER', 'stripe'),
    'key'      => env('PAYMENT_KEY'),
];
```

```php
config('payment.provider'); // 'stripe'
config('payment.key');      // null
config('missing', 'default'); // 'default'
```

## Core config files

### `config/theme.php`

```php
return [
    'menus' => [
        'primary' => 'Primary Navigation',
        'footer'  => 'Footer Navigation',
    ],

    'image_sizes' => [
        'hero' => [1920, 1080, true],
    ],

    'supports' => [
        'title-tag',
        'post-thumbnails',
        'html5' => ['search-form', 'comment-form', 'gallery'],
    ],

    'process_acf' => true,
];
```

### `config/app.php`

```php
return [
    'relative_urls'    => false,  // Convert all URLs to relative
    'relative_links'   => false,  // Convert link hrefs only
    'relative_uploads' => false,  // Convert upload srcs only

    'wp_json' => [
        'base_url' => 'wp-json', // REST API base URL prefix
    ],
];
```

### `config/admin.php`

```php
return [
    'hide_updates' => [
        'core'    => false,
        'plugins' => false,
        'themes'  => false,
    ],
    'footer'       => true,
    'cleanup_menu' => true,
];
```

## Environment variables

Environment variables are loaded from `.env` before boot. Use `env()` to read them:

```php
env('WP_ENV', 'production');
env('APP_SECRET');
env('DB_NAME');
```

The `.env` file lives at the project root — next to `composer.json` in both modes. In Theme mode that's the theme directory, in Classic mode that's the project root (not inside `app/`).

### Environments

`WP_ENV` controls the environment:

| Value | `app()->isLocal()` | `app()->isProduction()` |
|-------|-------------------|------------------------|
| `local` | `true` | `false` |
| `development` | `true` | `false` |
| `staging` | `false` | `false` |
| `production` | `false` | `true` |

## Writing config values at runtime

```php
config(['theme.menus.primary' => 'Main Menu']);
```

## Overriding framework defaults

Published config files take precedence over Sloth's built-in defaults. Unpublished keys always fall back to Sloth's defaults — you only need to publish and edit the keys you want to change.
