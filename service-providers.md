# Service Providers

A service provider is a class that tells the application how to wire things together. It answers two questions: *what exists* (`register`) and *what happens when the app boots* (`boot`).

If you've never worked with this pattern before, think of it as a structured replacement for the scattered `functions.php` logic you'd normally write in WordPress — but with a clear lifecycle, dependency injection, and access to the full container.

Sloth's service provider system is built directly on Laravel's. If you want to understand the pattern in depth before diving in, [Laravel's documentation](https://laravel.com/docs/providers) is the best resource available.

## Generating a provider

```bash
# Theme mode — always goes to theme/Providers/
wp sloth make:provider ThemeSetupProvider

# Classic mode — choose destination
wp sloth make:provider ThemeSetupProvider --theme   # → theme/Providers/
wp sloth make:provider AppServiceProvider --app     # → app/Providers/
```

## Anatomy of a provider

```php
<?php

namespace App\Providers;

use Sloth\Core\ServiceProvider;

class MyServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Bind services into the container.
        // Runs before boot() — other providers may not be available yet.
        $this->app->singleton('my-service', fn() => new MyService());
    }

    public function boot(): void
    {
        // Runs after all providers have been registered.
        // Safe to resolve other services from the container here.
    }

    public function getHooks(): array
    {
        // Declarative WordPress actions — no add_action() calls needed.
        return [
            'init' => fn() => $this->doSomething(),
        ];
    }

    public function getFilters(): array
    {
        // Declarative WordPress filters — no add_filter() calls needed.
        return [
            'the_title' => fn(string $title) => '[' . $title . ']',
        ];
    }
}
```

Drop the class into `app/Providers/` or `theme/Providers/` and Sloth discovers it automatically — no registration required.

## Boot lifecycle

Sloth boots on the `after_setup_theme` WordPress hook. The sequence is:

1. Framework providers register (infrastructure, database, views, routing…)
2. Your providers in `app/Providers/` and `theme/Providers/` register
3. Vendor package providers register
4. All providers boot in registration order
5. Hooks and filters from `getHooks()` / `getFilters()` are registered

This two-pass approach — register all, then boot all — means every binding is available in the container before any hook fires.

### Where does this sit in the WordPress lifecycle?

```
mu-plugins load          ← Classic mode: Cecropia bootstraps Sloth here
    ↓
plugins load
    ↓
after_setup_theme        ← Theme mode: functions.php bootstraps Sloth here
    ↓                       All providers register() and boot() here
    ↓                       getHooks() / getFilters() are registered here
init                     ← First hook your providers can reliably use
    ↓
wp_loaded
    ↓
template_redirect
    ↓
wp_head / the_content / …
```

**`register()`** runs during `after_setup_theme` — before `init`. Don't call WordPress functions that depend on the full WordPress environment being ready (e.g. `get_posts()`, `current_user_can()`). Stick to container bindings.

**`boot()`** also runs during `after_setup_theme`, but after all providers have registered. You can safely resolve services from the container. For anything that needs WordPress to be fully initialized, use `getHooks()` with `init` or later hooks.

**`getHooks()` / `getFilters()`** are registered during `after_setup_theme` but the callbacks themselves fire at their respective hook — `init`, `wp_loaded`, etc. This is where the bulk of your WordPress integration lives.

## Hook and filter registration

Instead of calling `add_action()` and `add_filter()` directly, return them from `getHooks()` and `getFilters()`. Sloth registers them after all providers have booted.

Three formats are supported:

```php
public function getHooks(): array
{
    return [
        // 1. Single callable — default priority 10
        'init' => fn() => $this->setup(),

        // 2. Multiple callbacks for the same hook
        'wp_loaded' => [
            fn() => $this->stepOne(),
            fn() => $this->stepTwo(),
        ],

        // 3. With explicit priority
        'save_post' => ['callback' => fn($id) => $this->onSave($id), 'priority' => 20],

        // 4. Multiple callbacks with different priorities
        'wp_head' => [
            ['callback' => fn() => $this->early(), 'priority' => 5],
            ['callback' => fn() => $this->late(),  'priority' => 99],
        ],
    ];
}
```

The same formats apply to `getFilters()`. Filter callbacks receive the filtered value as the first argument and must return it:

```php
public function getFilters(): array
{
    return [
        'the_content' => fn(string $content) => $this->transform($content),
    ];
}
```

## Binding services

Use `register()` to bind services into the container:

```php
public function register(): void
{
    // New instance every time
    $this->app->bind('my-service', fn() => new MyService());

    // Single shared instance
    $this->app->singleton('my-service', fn() => new MyService());

    // Specific implementation for an interface
    $this->app->bind(MyInterface::class, MyImplementation::class);
}
```

Resolve them anywhere:

```php
app('my-service');
app(MyInterface::class);
app()->make(MyService::class);
```

## When to use the EventBridge instead

`getHooks()` and `getFilters()` are ideal for provider-local concerns. For shared WordPress lifecycle hooks that multiple parts of your application might listen to, the [EventBridge](events) is more appropriate:

```php
use Sloth\Event\WpHookFired;
use Sloth\Facades\Event;

public function boot(): void
{
    Event::listen('wp:the_content', function (WpHookFired $event) {
        $event->result = transform($event->result);
    });
}
```

## Further reading

- [Laravel Service Providers](https://laravel.com/docs/providers) — lifecycle, binding, deferred providers
- [Laravel Service Container](https://laravel.com/docs/container) — binding, resolving, contextual binding

## Publishing config files

Packages and framework providers can ship default config files that users publish into their project:

```bash
wp sloth vendor:publish --provider="MyPackage\MyServiceProvider" --tag=config
wp sloth vendor:publish --tag=config   # publish all config files
```

To make your own provider publishable, declare files in `boot()`:

```php
public function boot(): void
{
    $this->publishes([
        __DIR__ . '/../config/my-package.php' => app()->path('config', 'app') . '/my-package.php',
    ], 'config');
}
```

:::tip Production deployment
After deploying new or renamed service providers, run `wp sloth manifest:clear` to regenerate the manifest cache. See [Auto-Discovery](auto-discovery#clearing-the-manifest-cache).
:::
