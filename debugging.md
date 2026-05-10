# Debugging

Sloth integrates [PHP DebugBar](https://github.com/maximebf/php-debugbar) for local development. It is only active when the package is installed and `WP_ENV` is `local`.

## Installation

DebugBar is a dev dependency — install it via Composer:

```bash
composer require php-debugbar/php-debugbar --dev
```

Sloth detects the package automatically. No further configuration needed.

## The toolbar

In local environments, the DebugBar toolbar is injected at the bottom of every HTML page. It includes:

| Collector | What it shows |
|-----------|---------------|
| **Messages** | `debug()` calls and dumps |
| **Sloth** | Loaded models, modules, providers |
| **ACF** | ACF field resolution |
| **WordPress** | WordPress hooks timeline |
| **Queries** | Database queries with timing |
| **PHP Info** | PHP version, memory limit |
| **Memory** | Peak memory usage |

## Dumping values

Use the global `debug()` helper to dump values into the DebugBar:

```php
debug($post);
debug($projects->toArray());
debug('checkpoint reached');
```

In Twig:

```twig
{{ post | debug }}
{{ debug(projects) }}
```

In local environments `debug()` sends to the DebugBar Messages collector. In production it falls back to `var_dump()`.

## JSON responses

When a route returns a JSON response, DebugBar injects a `__debug` key into the response body:

```json
{
    "__debug": [
        { "file": "app/Model/Project.php:42", "message": "..." }
    ],
    "data": [...]
}
```

## Configuration

Publish the config file to customize the DebugBar:

```bash
wp sloth vendor:publish --provider="Sloth\Debug\DebugServiceProvider" --tag=config
```

```php
// app/config/debugger.php
return [
    'editor' => 'phpstorm',   // or 'vscode', 'sublime', etc.
    'bar' => [
        'display'    => true, // show the toolbar
        'dump_all'   => true, // dump all debug() calls
        'collector_providers' => [
            // Add or remove collectors
        ],
    ],
    'json' => [
        'prepend' => true,       // inject __debug into JSON responses
        'key'     => '__debug',  // the key name
    ],
];
```

## Custom collectors

Add your own DebugBar collector via the `collector_providers` config key. A collector provider is a class that implements `CollectorProviderInterface`:

```php
use Sloth\Debug\CollectorProviders\CollectorProviderInterface;
use DebugBar\DebugBar;

class MyCollectorProvider implements CollectorProviderInterface
{
    public function register(DebugBar $debugBar): void
    {
        $debugBar->addCollector(new MyCollector());
    }
}
```

Then add it to the config:

```php
'collector_providers' => [
    // framework collectors...
    MyCollectorProvider::class,
],
```
