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

Register custom functions and filters in `app/config/theme.php` or `theme/config/theme.php`:

```php
<?php

use Twig\TwigFilter;
use Twig\TwigFunction;

return [
    'twig' => [
        'functions' => [
            new TwigFunction('my_function', fn() => my_function()),
        ],
        'filters' => [
            new TwigFilter('my_filter', fn($value) => transform($value)),
        ],
        'autoescape' => false,
    ],
];
```

## Polylang

If Polylang is active, `pll__()` and `pll_e()` are registered as Twig functions automatically. See [Polylang](polylang) for details.

```twig
{{ pll__('Hello') }}
```

## Further reading

- [Twig documentation](https://twig.symfony.com/doc/) — syntax, filters, functions, tags
