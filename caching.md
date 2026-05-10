# Caching

Sloth provides Laravel's Cache system with two drivers out of the box: **file** (default) and **WordPress Transients**.

## Basic usage

```php
use Sloth\Facades\Cache;

// Store for N seconds
Cache::put('my-key', $value, 3600);

// Store forever
Cache::forever('my-key', $value);

// Retrieve
Cache::get('my-key');
Cache::get('my-key', 'default');

// Retrieve or store
Cache::remember('my-key', 3600, fn() => expensive());
Cache::rememberForever('my-key', fn() => expensive());

// Check and forget
Cache::has('my-key');
Cache::forget('my-key');
Cache::flush();

// Increment / decrement
Cache::increment('counter');
Cache::decrement('counter', 5);
```

## Drivers

### File (default)

Cache files are stored in `theme/cache/Cache/`. This directory is created automatically by Sloth.

```php
Cache::put('key', $value, 3600); // uses file driver by default
```

### WordPress Transients

Stores cache data in the WordPress database via the Transients API. Useful when filesystem caching is unavailable or when you want cache to survive deployments.

Publish the cache config and set the driver:

```bash
wp sloth vendor:publish --provider="Sloth\Cache\CacheServiceProvider" --tag=config
```

```php
// app/config/cache.php
return [
    'driver' => 'wp-transients',
    'prefix' => 'my_project_', // optional — defaults to 'sloth_'
];
```

Or use per call:

```php
Cache::driver('wp-transients')->rememberForever('key', fn() => expensive());
```

All keys are prefixed with `sloth_` by default to avoid collisions with other plugins. Override via the `prefix` config key.

All keys are prefixed with `sloth_` to avoid collisions with other plugins.

### Array

In-memory only — not persisted between requests. Useful in tests.

```php
Cache::driver('array')->put('key', $value, 3600);
```

## Caching in production

Sloth uses the cache internally for manifest resolution. In local development manifests are always rebuilt from disk. In production they are cached forever — which is why `wp sloth manifest:clear` is needed after deployments.

For your own code, the same pattern is recommended:

```php
public static function resolve(): Collection
{
    if (app()->isLocal()) {
        return static::fetch(); // always fresh in development
    }

    return Cache::rememberForever(static::$cacheKey, fn() => static::fetch());
}
```

## Further reading

- [Laravel Cache](https://laravel.com/docs/cache) — full Cache API reference
