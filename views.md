# Views

Sloth uses [Twig](https://twig.symfony.com/) as its templating engine. Templates live in `theme/View/` and receive a rich context automatically — no manual setup needed.

## Directory structure

```
theme/View/
├── Layout/           # Template hierarchy templates (matched by WordPress)
│   ├── index.twig
│   ├── single.twig
│   ├── single-project.twig
│   ├── archive-project.twig
│   ├── page.twig
│   └── 404.twig
└── Module/           # Module templates
    ├── hero.twig
    └── featured-projects.twig
```

## Generating a layout

```bash
# Interactive — select from the full WordPress template hierarchy
wp sloth make:layout

# Direct — create a specific template immediately
wp sloth make:layout single
wp sloth make:layout single-project
wp sloth make:layout archive-event
wp sloth make:layout page-contact
```

The interactive mode shows the full WordPress template hierarchy and prompts for any placeholders — post type, taxonomy, term or slug — using suggestions from your registered models and taxonomies.

All layouts extend `Partials/wrapper.twig` by default:

```twig
{% extends 'Partials/wrapper.twig' %}

{% block content %}

{% endblock %}
```

## Automatic context

Every Twig template receives these variables automatically:

```twig
{# Site information #}
```twig
{# Site information — values are resolved via get_bloginfo() and apply its filters #}
{{ site.name }}
{{ site.description }}
{{ site.url }}
{{ site.language }}
{{ site.canonical_url }}
```

:::note
`site.*` values come from `get_bloginfo()` — not raw `get_option()`. This means WordPress filters like `bloginfo` are applied, and plugins that modify these values will be respected. If you need the raw option value, use `{{ options.blogname }}` instead.
:::

```twig
{# Global URLs #}
{{ globals.home_url }}
{{ globals.theme_url }}
{{ globals.images_url }}

{# Current post — available on single posts and pages #}
{{ post.title }}
{{ post.content }}
{{ post.excerpt }}
{{ post.permalink }}
{{ post.image }}
{{ post.my_acf_field }}

{# On taxonomy archives #}
{{ taxonomy.name }}
{{ taxonomy.slug }}

{# On author archives #}
{{ author.display_name }}
```

The current post is also available under its post type name:

```twig
{# single-project.twig #}
{{ project.title }}
{{ project.my_acf_field }}
```

## WordPress functions in Twig

Any PHP or WordPress function can be called via the `fn` global:

```twig
{{ fn.get_the_title() }}
{{ fn.bloginfo('name') }}
{{ fn.get_option('blogname') }}
```

Or use the built-in Twig functions Sloth registers:

```twig
{# Required in your base layout #}
{{ wp_head() }}
{{ wp_footer() }}

{# Body and post classes #}
<body class="{{ body_class() }}">
<article class="{{ post_class() }}">

{# ACF #}
{{ get_field('my_field') }}

{# Meta #}
{{ meta('my_key') }}
{{ meta('my_key', post.ID) }}

{# i18n #}
{{ __('Hello', 'my-theme') }}
{{ _n('One item', '%d items', count, 'my-theme') }}

{# URLs #}
{{ url().route('projects.index') }}
{{ url().theme('css/app.css') }}
{{ url().asset('js/app.js') }}

{# Modules #}
{{ module('hero', { title: 'Hello' }) }}
```

## Twig filters

```twig
{# Wrap a phone number in a tel: URI #}
{{ '+49 123 456789' | tel }}         {# tel:+49123456789 #}

{# Sanitize a string as a WordPress slug #}
{{ 'My Title' | sanitize }}          {# my-title #}

{# Debug #}
{{ post | debug }}
```

## Rendering from PHP

```php
use Sloth\Facades\View;

// Render a template
$html = View::make('Layout/single')->with(['post' => $post])->render();

// Via Response
return Response::view('Layout/projects', ['projects' => $projects]);
```

## Extending Twig

Sloth uses a `AbstractViewExtension` system for registering custom helpers and directives. Drop a subclass in `app/Extensions/View/` or `theme/Extensions/View/` and Sloth discovers it automatically.

### Generating an extension

```bash
wp sloth make:extension FormatExtension
```

Creates `app/Extensions/View/FormatExtension.php` (Classic) or `theme/Extensions/View/FormatExtension.php` (Theme mode) with `getHelpers()`, `getDirectives()` and `share()` ready to fill in.

### Helpers

Helpers transform a value — registered as Twig filters:

```php
// theme/Extensions/View/PhoneExtension.php
class PhoneExtension extends \Sloth\View\Extensions\AbstractViewExtension
{
    public function getHelpers(): array
    {
        return [
            'format_phone' => fn($number) => preg_replace('/[^0-9\+]/', '', $number),
            'tel'          => fn($number) => 'tel:' . preg_replace('/[^0-9\+]/', '', $number),
        ];
    }
}
```

```twig
{{ '+49 123 456 789' | format_phone }}
{{ '+49 123 456 789' | tel }}
```

### Directives

Directives generate output or call actions — registered as Twig functions:

```php
// theme/Extensions/View/PriceExtension.php
class PriceExtension extends \Sloth\View\Extensions\AbstractViewExtension
{
    public function getDirectives(): array
    {
        return [
            'format_price' => fn(float $value, string $currency = 'EUR') => 
                number_format($value, 2) . ' ' . $currency,
        ];
    }
}
```

```twig
{{ format_price(9.99) }}
{{ format_price(9.99, 'USD') }}
```

### Shared variables

Variables available in all templates:

```php
public function share(): array
{
    return [
        'my_global' => 'value',
    ];
}
```

```twig
{{ my_global }}
```

### Custom names

Use string keys to control the name in templates — useful for names that aren't valid PHP identifiers:

```php
public function getDirectives(): array
{
    return [
        'pll__' => fn(string $s) => pll__($s),  // {{ pll__('Hello') }}
    ];
}
```

### Array formats

All three methods support the same formats — analogous to `getHooks()`/`getFilters()` in service providers:

```php
return [
    'wp_head',                          // string — callable of same name
    'pll__'  => 'pll__',               // alias → callable string
    'format' => fn($v) => transform($v), // closure
];
```

## Polylang

If Polylang is active, `pll__()` and `pll_e()` are registered as Twig functions automatically. See [Polylang](polylang) for details.

```twig
{{ pll__('Hello') }}
```

## Further reading

- [Twig documentation](https://twig.symfony.com/doc/) — syntax, filters, functions, tags
