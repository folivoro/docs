# Container

Sloth's service container is Laravel's container — the same DI system that powers Laravel applications. It manages class dependencies and performs dependency injection automatically.

If you're new to the concept, [Laravel's container documentation](https://laravel.com/docs/container) covers it in depth. Everything there applies to Sloth.

## Accessing the container

```php
// Global helper
app();

// Resolve a binding
app('view');
app(MyService::class);

// With arguments
app()->make(MyService::class, ['option' => 'value']);
```

## Binding

Bind services in a service provider's `register()` method:

```php
// New instance every time
$this->app->bind(MyService::class, fn() => new MyService());

// Shared instance — same object returned every time
$this->app->singleton(MyService::class, fn() => new MyService());

// Bind an interface to a concrete implementation
$this->app->bind(MyInterface::class, MyImplementation::class);

// Store an existing instance
$this->app->instance('my-key', new MyService());
```

## Automatic injection

The container resolves constructor dependencies automatically:

```php
class MyServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // The container resolves MyDependency automatically
        $service = app()->make(MyService::class);
    }
}

class MyService
{
    public function __construct(
        private readonly MyDependency $dependency,
    ) {}
}
```

## Facades

Facades provide a static interface to services bound in the container. Sloth ships with the following facades:

| Facade | What it resolves |
|--------|-----------------|
| `Cache` | Laravel CacheManager — get, put, remember, forget |
| `File` | Illuminate Filesystem — read, write, copy, delete |
| `View` | Twig view factory — make, render |
| `Route` | Sloth Router — get, post, put, delete, match |
| `URL` | UrlGenerator — home, theme, asset, route, current |
| `Response` | HTTP response factory — make, json, redirect, download |
| `Options` | WordPress options + ACF options API |
| `Pagination` | Laravel Paginator |
| `Validation` | Laravel Validator |
| `Customizer` | WordPress Customizer integration |
| `Deployment` | Deployment utilities |

Use them with their full namespace or via the registered class alias:

```php
use Sloth\Facades\Cache;
use Sloth\Facades\View;
use Sloth\Facades\URL;

// Full namespace
Cache::remember('my-key', 3600, fn() => expensive());

// Class alias (registered automatically by Sloth)
View::make('Layout/single');
URL::route('projects.index');
```

## Helpers

Sloth also registers global helper functions:

```php
app()           // The application container
app('key')      // Resolve a binding
config('key')   // Read configuration
env('KEY')      // Read an environment variable
url()           // UrlGenerator instance
url('/path')    // Generate a URL
module('name')  // Render a module
debug($value)   // Dump a value (DebugBar or var_dump)
```

## Path helpers

```php
app()->path()                    // Project root (basePath)
app()->path('app')               // app/ directory
app()->path('theme')             // Active theme directory
app()->path('Model')             // app/Model/
app()->path('Model', 'theme')    // theme/Model/
app()->uri('theme')              // Theme URL
app()->uri('asset', 'theme')     // Theme asset URL
```
