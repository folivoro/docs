# Facades

Facades provide a static interface to services bound in the container. Sloth registers all facades automatically — no configuration needed.

## Available facades

| Facade | Container key | What it does |
|--------|--------------|--------------|
| `Cache` | `cache` | Laravel Cache — get, put, remember, forget |
| `File` | `files` | Illuminate Filesystem — read, write, copy, delete |
| `View` | `view` | Twig view factory — make, render |
| `Route` | `router` | Sloth Router — get, post, put, delete |
| `URL` | `url` | URL generation — home, theme, asset, route |
| `Response` | `response` | HTTP responses — make, json, view, redirect, download |
| `Options` | `options` | WordPress options + ACF options API |
| `Pagination` | `pagination` | Laravel Paginator |
| `Validation` | `validator` | Laravel Validator |
| `Menu` | `menu` | WordPress nav menus |
| `Module` | `module` | Module rendering |
| `Layotter` | `layotter` | Layotter page builder |
| `Customizer` | `customizer` | WordPress Customizer |
| `Deployment` | `deployment` | Deployment utilities |

## Usage

```php
use Sloth\Facades\Cache;
use Sloth\Facades\View;
use Sloth\Facades\URL;
use Sloth\Facades\Route;
use Sloth\Facades\Response;
use Sloth\Facades\Options;
use Sloth\Facades\File;

// Cache
Cache::remember('key', 3600, fn() => expensive());
Cache::forget('key');

// View
View::make('Layout/single')->with(['post' => $post])->render();

// URL
URL::route('projects.index');
URL::theme('css/app.css');
URL::asset('js/app.js');

// Route
Route::get('/projects', fn() => Response::view('Layout/projects'));

// Response
Response::view('Layout/page', ['post' => $post]);
Response::json(['key' => 'value']);
Response::redirect('/new-url');
Response::download('/path/to/file.pdf', 'report.pdf');

// Options
Options::get('blogname');
Options::get('primary_color', '#000');
options('blogname');        // global helper
options('primary_color');   // with default

// File
File::get('/path/to/file.txt');
File::put('/path/to/file.txt', 'contents');
File::exists('/path/to/file.txt');
```

## Writing your own facade

```php
<?php

namespace App\Facades;

use Sloth\Facades\Facade;

class MyService extends Facade
{
    protected static function getFacadeAccessor(): string
    {
        return 'my-service'; // container binding key
    }
}
```

Register the binding in a service provider:

```php
public function register(): void
{
    $this->app->singleton('my-service', fn() => new MyService());
}
```

## Further reading

- [Laravel Facades](https://laravel.com/docs/facades) — how facades work, real-time facades, testing
