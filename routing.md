# Routing

Sloth comes with two complementary routing mechanisms: the **WordPress Template Hierarchy** for standard content pages, and a **custom router** for anything that goes beyond what WordPress offers.

## WordPress Template Hierarchy

For most pages — posts, archives, taxonomies, custom post types — you don't need to define routes at all. Place a Twig file with the right name in `theme/View/Layout/` and Sloth picks it up automatically via the WordPress template hierarchy.

```
theme/View/Layout/
├── index.twig              # Fallback for everything
├── single.twig             # Single posts
├── single-project.twig     # Single post of type "project"
├── archive.twig            # Archives
├── archive-project.twig    # Archive for post type "project"
├── page.twig               # Pages
├── page-about.twig         # Page with slug "about"
├── taxonomy.twig           # Taxonomy archives
├── search.twig             # Search results
└── 404.twig                # Not found
```

See the [WordPress Template Hierarchy](https://developer.wordpress.org/themes/classic-themes/basics/template-hierarchy/) for the full list of template names.

## Custom Routes

When you need custom URLs or parameters that don't map to WordPress content, use Sloth's router.

Define routes in `app/routes/web.php` or `theme/routes/web.php`:

```php
<?php

use Sloth\Facades\Response;
use Sloth\Facades\Route;

Route::get('/feed/custom', function () {
    return Response::make(renderFeed(), 200)
        ->header('Content-Type', 'application/xml');
});
```

Both files are loaded if present — `app/routes/web.php` first, then `theme/routes/web.php`.

### HTTP methods

```php
Route::get('/projects', fn() => Response::view('projects/index'));
Route::post('/contact', fn() => handleForm());
Route::put('/projects/{id}', fn(string $id) => update($id));
Route::delete('/projects/{id}', fn(string $id) => delete($id));
```

### URL parameters

Parameters in curly braces are passed to the callback in order:

```php
Route::get('/projects/{slug}', function (string $slug) {
    $project = Project::where('post_name', $slug)->firstOrFail();

    return Response::view('projects/show', ['project' => $project]);
});
```

### Named routes

Assign a name by chaining `->name()`:

```php
Route::get('/projects', fn() => Response::view('projects/index'))
    ->name('projects.index');

Route::get('/projects/{slug}', fn(string $slug) => showProject($slug))
    ->name('projects.show');
```

### Dispatch

Routes are matched and dispatched on the `template_redirect` hook at priority 1 — before WordPress loads its own templates. If a route matches, WordPress template loading is bypassed entirely.

## Responses

Route callbacks return a `Response` instance. Sloth's `Response` class extends Illuminate's with static factory methods:

```php
use Sloth\Facades\Response;

// Render a Twig template
Route::get('/projects', function () {
    return Response::view('Layout/projects', [
        'projects' => Project::all(),
    ]);
});

// With status code and headers
Route::get('/feed', function () {
    return Response::make(renderFeed(), 200)
        ->header('Content-Type', 'application/xml');
});

// JSON
Route::get('/api/projects', function () {
    return Response::json(Project::all()->toArray());
});

Route::get('/api/projects/{id}', function (string $id) {
    $project = Project::find($id);
    if (!$project) {
        return Response::json(['error' => 'Not found'], 404);
    }
    return Response::json($project->toArray());
});

// Redirect
Route::get('/old-url', function () {
    return Response::redirect('/new-url');        // 302
    return Response::redirect('/new-url', 301);   // permanent
});

// File download
Route::get('/download', function () {
    return Response::download('/path/to/file.pdf', 'report.pdf');
});

// Inline file (display in browser)
Route::get('/preview', function () {
    return Response::file('/path/to/file.pdf');
});

// 204 No Content
Route::delete('/projects/{id}', function (string $id) {
    Project::find($id)?->delete();
    return Response::noContent();
});
```

## URL Generation

The `URL` facade and `url()` helper generate URLs for named routes and WordPress locations:

```php
use Sloth\Facades\URL;

// Named routes
URL::route('projects.index');                           // /projects
URL::route('projects.show', ['slug' => 'my-project']); // /projects/my-project

// WordPress URLs
URL::home();                    // https://example.com
URL::to('/about');              // https://example.com/about
URL::theme();                   // https://example.com/wp-content/themes/my-theme
URL::theme('css/app.css');      // https://example.com/.../my-theme/css/app.css
URL::asset('css/app.css');      // https://example.com/.../my-theme/public/css/app.css
URL::content();                 // https://example.com/wp-content
URL::uploads();                 // https://example.com/wp-content/uploads
URL::current();                 // /current/path (no host)
URL::full();                    // https://example.com/current/path
```

In Twig:

```twig
{{ url().route('projects.index') }}
{{ url().theme('css/app.css') }}
{{ url().asset('js/app.js') }}
```

Or via the global `url()` helper:

```php
url('/about');                          // https://example.com/about
url()->route('projects.show', [...]);   // named route
```
