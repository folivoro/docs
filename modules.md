# Modules

Modules are reusable UI components — a PHP class paired with a Twig template. Drop a class extending `Sloth\Module\Module` into `app/Module/` or `theme/Module/` and Sloth discovers it automatically.

## Generating a module

```bash
wp sloth make:module HeroModule
```

This creates two files:

```
theme/Module/HeroModule/HeroModule.php
theme/Module/HeroModule/hero-module.twig
```

## Creating a module manually

```php
<?php

namespace App\Module;

use App\Model\Project;
use Sloth\Module\Module;

class FeaturedProjectsModule extends Module
{
    protected function beforeRender(): void
    {
        $this->set('projects', Project::limit(3)->get());
        $this->set('title', 'Featured Projects');
    }
}
```

The template is resolved automatically from the class name — `FeaturedProjectsModule` maps to `theme/View/Module/featured-projects.twig`:

```twig
<section class="featured-projects">
    <h2>{{ title }}</h2>
    {% for project in projects %}
        <article>
            <h3>{{ project.title }}</h3>
            <p>{{ project.excerpt }}</p>
            <a href="{{ project.permalink }}">Read more</a>
        </article>
    {% endfor %}
</section>
```

## Rendering

From Twig:

```twig
{{ module('featured-projects') }}

{# Pass data directly #}
{{ module('featured-projects', { title: 'Our Work' }) }}
```

From PHP:

```php
// Render and echo
module('featured-projects');
module('featured-projects', ['title' => 'Our Work']);

// Render and return
app('module.factory')->render('featured-projects', ['title' => 'Our Work']);

// Instantiate without rendering
$module = app('module.factory')->make('featured-projects');
$module->render();
```

## View variables

```php
protected function beforeRender(): void
{
    // Set a single variable
    $this->set('title', 'Hello');

    // Set multiple variables
    $this->set([
        'title'    => 'Hello',
        'subtitle' => 'World',
    ]);

    // Read a variable
    $title = $this->get('title');

    // Check if set
    if ($this->isSet('title')) { ... }

    // Unset
    $this->unset('title');
}
```

Variables passed via `module()` or `make()` are merged with those set in `beforeRender()`. Variables from `module()` do not override variables set in `beforeRender()` — the module always has the final say.

## Automatic context

Every module automatically receives the full Twig context — `site`, `globals`, `post`, and any other variables available in the current template. You don't need to pass them manually.

## Name resolution

Module names are resolved to class names in the `Theme\Module` namespace:

| Name | Class |
|------|-------|
| `hero` | `Theme\Module\HeroModule` |
| `hero-section` | `Theme\Module\HeroSectionModule` |
| `hero_section` | `Theme\Module\HeroSectionModule` |

## REST endpoint

Modules can expose a JSON endpoint automatically by setting `$json`:

```php
class ProjectsModule extends Module
{
    // Enable JSON endpoint
    public static $json = true;

    // With URL parameters
    public static $json = ['params' => ['category']];

    protected function beforeRender(): void
    {
        $category = $this->get('category');
        $this->set('projects', Project::taxonomy('category', $category)->get());
    }
}
```

This registers:
- `GET /wp-json/sloth/v1/module/projects` 
- `GET /wp-json/sloth/v1/module/projects/{category}` (with params)
- `wp_ajax_` and `wp_ajax_nopriv_` AJAX handlers

The endpoint returns the module's view variables as JSON.

## AJAX URL

Every module has a built-in AJAX URL available in its template:

```twig
<div data-ajax="{{ ajax_url }}">...</div>
```

## Layotter

Modules can be used as Layotter page builder elements. See [Layotter](layotter) for details.

:::tip Production deployment
After deploying new or renamed modules, run `wp sloth manifest:clear` to regenerate the manifest cache. See [Auto-Discovery](auto-discovery#clearing-the-manifest-cache).
:::
