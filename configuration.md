# Configuration

Sloth uses two complementary configuration systems: **Laravel's config repository** (the primary system) and **`Configure`** (a legacy store kept for backwards compatibility).

## Config files

Configuration files live in `app/config/` and are loaded automatically on boot. Each file must return an array and its filename becomes the config key. Create them as needed — there are no required files:

```
app/config/
├── theme.php         # config('theme')
├── events.php        # config('events')
└── layotter.php      # config('layotter')
```

```php
// app/config/theme.php
return [
    'twig' => [
        'autoescape' => false,
        'filters'    => [],
        'functions'  => [],
    ],
];
```

Read values anywhere with `config()`:

```php
config('theme.twig.autoescape');   // false
config('theme.twig.filters');      // []
config('missing.key', 'default'); // 'default'
```

## Environment variables

Environment variables are loaded from `.env` automatically before boot. Use `env()` to read them:

```php
env('WP_ENV', 'production');  // 'local', 'development', 'staging', 'production'
env('APP_SECRET');
env('DB_NAME');
```

The `.env` file lives at the project root — next to `composer.json` in Classic mode, or inside the theme in Theme mode.

### Environments

`WP_ENV` controls the environment:

| Value | `isLocal()` | `isProduction()` |
|-------|-------------|------------------|
| `local` | `true` | `false` |
| `development`, `develop`, `dev` | `true` | `false` |
| `testing` | `false` | `true` |
| `staging`, `production`, anything else | `false` | `true` |

`local` is the recommended value for local development.

```php
app()->isLocal();       // true when WP_ENV is local/development/develop/dev
app()->isProduction();  // true for everything else
app()->environment();   // returns WP_ENV value directly
```

## Overriding config at runtime

```php
// In a service provider or config file
config(['theme.twig.autoescape' => true]);

// Using dot notation
config(['my.nested.key' => 'value']);
```

## Theme config

The theme can also have a `config.php` directly in its root — loaded before other providers boot:

```php
// theme/config.php
use Twig\TwigFilter;

Configure::write('theme.twig.filters', [
    new TwigFilter('my_filter', fn($v) => transform($v)),
]);
```

## Configure (legacy)

`Configure` is a standalone key-value store that predates the Laravel config system. It's kept for backwards compatibility — new code should use `config()` directly.

```php
use Sloth\Configure\Configure;

// Write
Configure::write('theme.foo', 'bar');
Configure::write(['theme.foo' => 'bar', 'theme.baz' => 'qux']);

// Read
Configure::read('theme.foo');          // 'bar'
Configure::read('theme.foo', 'default'); // with default
Configure::read();                     // all values

// Check and delete
Configure::check('theme.foo');         // true
Configure::delete('theme.foo');
```

Values written via `Configure::write()` are merged into the Laravel config repository on boot, so `config('theme.foo')` and `Configure::read('theme.foo')` both work. Laravel config files take precedence over `Configure` values.

:::tip
Migrate `Configure::write()` calls to config files that return arrays — it's cleaner and integrates better with the rest of the framework.
:::

## Framework config

Framework components ship their own config files that can be extended in `app/config/`. For example, to add hooks to the Event Bridge:

```php
// app/config/events.php
return [
    'bridge' => [
        'save_post'        => 'action',
        'wp_nav_menu_args' => 'filter',
    ],
];
```

Framework defaults are merged with your values via `array_replace_recursive` — you only need to specify what you want to change.

## Reading config from the CLI

```bash
wp sloth config:get theme.twig.autoescape
wp sloth config:get app.name
```

Returns the value as JSON. Exits with code `1` if the key does not exist.

## Publishing framework config files

Framework components ship default config files that can be published into your project for customization:

```bash
wp sloth vendor:publish --provider="Sloth\Event\EventServiceProvider" --tag=config
```

This copies the default `events.php` into `app/config/` where you can extend it. Your values are merged with the framework defaults — you only need to specify what you want to change.
