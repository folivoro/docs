# Models

Sloth models represent WordPress custom post types as Eloquent models. Under the hood, Sloth builds on [Corcel](https://github.com/corcel/corcel) — a Laravel package that maps WordPress database tables to Eloquent models. If you've used Eloquent before, models will feel immediately familiar.

## Creating a model

Drop a class extending `Sloth\Model\Model` into `app/Model/` or `theme/Model/` and Sloth registers it automatically as a WordPress custom post type.

```php
<?php

namespace App\Model;

use Sloth\Model\Model;

class Project extends Model
{
    // Singular and plural labels — used to generate all WordPress admin labels
    public static $names = [
        'singular' => 'Project',
        'plural'   => 'Projects',
    ];

    // Dashicon for the admin menu
    public static $icon = 'dashicons-portfolio';

    // Any valid register_post_type() argument
    public static $options = [
        'supports' => ['title', 'editor', 'thumbnail'],
        'has_archive' => true,
    ];
}
```

The post type slug is derived from the class name — `Project` becomes `project`. Override it explicitly if needed:

```php
public static $postType = 'my-project';
```

## Querying

Sloth applies a global scope automatically: non-logged-in users always see published posts only. You don't need to add `->published()` in your templates.

```php
// All projects (published for guests, all statuses for logged-in users)
Project::all();

// With explicit scopes
Project::published()->get();
Project::status('draft')->get();

// By slug
Project::slug('my-project')->first();
Project::findBySlugOrId('my-project')->first();

// By taxonomy
Project::taxonomy('category', 'featured')->get();
Project::taxonomy('post_tag', ['php', 'wordpress'])->get();

// Search across title, excerpt and content
Project::search('keyword')->get();

// Ordering
Project::orderBy('post_date', 'desc')->get();
Project::orderByMeta('priority', 'asc')->get();

// Pagination
Project::paginate(12);
```

## Accessing fields

```php
$project = Project::slug('my-project')->first();

// Built-in aliases
$project->title;        // post_title
$project->content;      // post_content (with the_content filters applied)
$project->excerpt;      // post_excerpt (shortcodes stripped)
$project->slug;         // post_name
$project->status;       // post_status
$project->created_at;   // post_date
$project->updated_at;   // post_modified
$project->permalink;    // get_permalink()
$project->image;        // featured image as Image object

// Post meta — direct access falls back to meta automatically
$project->my_key;

// ACF fields — resolved automatically
$project->my_acf_field;
$project->acf->my_acf_field; // via AcfProxy
```

## ACF integration

ACF fields are resolved automatically on model retrieval. No extra code needed — just access the field by name:

```php
$project->my_acf_field;
```

Fields are cached per model instance. To access the raw ACF proxy:

```php
$project->acf->my_acf_field;
```

## Meta fields

```php
// Read — direct access falls back to meta automatically
$project->my_key;

// Write
$project->saveMeta('my_key', 'value');
$project->saveMeta([
    'key_one' => 'value one',
    'key_two' => 'value two',
]);

// Query by meta
Project::hasMeta('featured', true)->get();
Project::hasMeta('status', 'active')->get();
Project::hasMetaLike('title', '%important%')->get();
```

## Relationships

```php
// Taxonomies
$project->taxonomies;
$project->terms;         // grouped by taxonomy: ['category' => ['slug' => 'name']]
$project->main_category; // first term name

// Author
$project->author;

// Hierarchical
$project->parent;
$project->children;

// Media
$project->attachment;
$project->thumbnail;
```

## Taxonomies

Taxonomies follow the same convention — drop a class into `app/Taxonomy/` or `theme/Taxonomy/` and Sloth registers it automatically.

```php
<?php

namespace App\Taxonomy;

use Sloth\Model\Taxonomy;

class ProjectCategory extends Taxonomy
{
    public static $names = [
        'singular' => 'Category',
        'plural'   => 'Categories',
    ];

    // Attach to one or more post types
    public static $postTypes = ['project'];

    // Only one term can be selected per post (radio instead of checkbox)
    public static $unique = false;

    public static $options = [
        'hierarchical' => true,
        'show_in_rest' => true,
    ];
}
```

Query posts by taxonomy term:

```php
Project::taxonomy('projectcategory', 'featured')->get();
```

Access taxonomy terms on a model:

```php
$project->terms;
// ['projectcategory' => ['featured' => 'Featured']]
```

## Registration options

| Property | Type | Description |
|----------|------|-------------|
| `$names` | `array` | `singular` and `plural` — used to generate WordPress admin labels |
| `$options` | `array` | Any valid `register_post_type()` / `register_taxonomy()` argument |
| `$labels` | `array` | Override auto-generated labels entirely |
| `$icon` | `string` | Dashicon for the admin menu (models only) |
| `$register` | `bool` | Set to `false` to use a class for querying only, without registration |
| `$postType` | `string` | Explicit post type slug (models only, defaults to lowercase class name) |
| `$postTypes` | `array` | Post types this taxonomy is attached to (taxonomies only) |
| `$unique` | `bool` | Single-value taxonomy — replaces checkbox metabox with radio buttons (taxonomies only) |
| `$layotter` | `array\|bool` | Layotter page builder config — see [Layotter](layotter) (models only) |

## Further reading

- [Corcel](https://github.com/corcel/corcel) — the Eloquent WordPress layer Sloth builds on
- [Laravel Eloquent](https://laravel.com/docs/eloquent) — full Eloquent documentation

:::tip Production deployment
After deploying new or renamed models or taxonomies, run `wp sloth manifest:clear` to regenerate the manifest cache. See [Auto-Discovery](auto-discovery#clearing-the-manifest-cache).
:::
