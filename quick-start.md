# Quick Start

This guide walks you through the three core building blocks of a Sloth application: a Model, a Route, and a Module. By the end you'll have a working page that queries data and renders it with Twig.

It assumes you've already installed Sloth via one of the starters. If not, see [Introduction](introduction).

## 1. Create a Model

Models represent WordPress custom post types. Drop a class into `app/Model/` (Classic mode) or `theme/Model/` (Theme mode) and Sloth registers it automatically.

```php
<?php

namespace App\Model;

use Sloth\Model\Model;

class Project extends Model
{
    public static $names = [
        'singular' => 'Project',
        'plural'   => 'Projects',
    ];

    public static $icon = 'dashicons-portfolio';
}
```

That's it. Sloth registers the `project` post type with WordPress, generates labels from `$names`, and makes the model queryable via Eloquent.

```php
// Fetch all projects — Sloth automatically limits to published posts for guests
$projects = Project::all();

// Fetch a single project by slug
$project = Project::where('post_name', 'my-project')->first();

// Access fields
$project->title;
$project->content;
$project->permalink;
$project->image;        // featured image as Image object
$project->my_acf_field; // ACF fields resolved automatically
```

:::tip
Sloth applies a global scope that restricts queries to published posts for non-logged-in users automatically. You don't need to add `->published()` in your templates.
:::

## 2. Template Hierarchy

Sloth uses the [WordPress Template Hierarchy](https://developer.wordpress.org/themes/classic-themes/basics/template-hierarchy/) to resolve templates automatically. Just place a Twig file with the right name in your `View/Layout/` directory and Sloth picks it up — no routing needed.

For example, to render a single project:

```
theme/View/Layout/single-project.twig
```

The file naming follows the WordPress template hierarchy exactly — `single-project.twig` for single posts of type `project`, `archive-project.twig` for the archive, `page.twig` for pages, and so on.

The current post is available in every template automatically:

```twig
{# single-project.twig #}
<h1>{{ post.title }}</h1>
<div>{{ post.content }}</div>
<a href="{{ post.permalink }}">{{ post.title }}</a>

{% if post.image %}
    <img src="{{ post.image.url }}" alt="{{ post.title }}">
{% endif %}
```

If you need more than WordPress routing offers — custom endpoints, URL parameters, API routes — Sloth has a routing layer on top. See [Routing](routing).

## 3. Create a Module

Modules are reusable UI components that pair a PHP class with a Twig template. Drop a class into `app/Module/` or `theme/Module/` and Sloth discovers it automatically.

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
    }
}
```

The template lives at `theme/View/Module/featured-projects.twig` — the filename is derived from the class name automatically:

```twig
<section class="featured-projects">
    {% for project in projects %}
        <article>
            <h2>{{ project.title }}</h2>
            <p>{{ project.excerpt }}</p>
            <a href="{{ project.permalink }}">Read more</a>
        </article>
    {% endfor %}
</section>
```

Render the module from any Twig template:

```twig
{{ module('featured-projects') }}
```

Or from PHP:

```php
module('featured-projects');
```

## Next steps

- [Directory Structure](directory-structure) — where to put things in Classic and Theme mode
- [Models](models) — custom post types, ACF, taxonomies, relationships
- [Routing](routing) — custom routes, parameters, named routes, URL generation
- [Modules](modules) — view variables, AJAX, REST endpoints
