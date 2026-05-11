# Context

The template context is the set of variables available in every Twig template. Sloth builds it lazily — values are only resolved when the template actually accesses them.

## Built-in context keys

| Key | Content | Condition |
|-----|---------|-----------|
| `site` | Site information — name, url, description, language, … | Always |
| `globals` | Global URLs — home_url, theme_url, images_url | Always |
| `sloth` | Sloth internals — current_layout | Always |
| `wp_title` | Current page title | Always |
| `options` | Options accessor — `{{ options.my_field }}` | Always |
| `post` | Current post model | Single posts and pages only |
| `{post_type}` | Alias for `post` — e.g. `{{ project.title }}` | Single posts and pages only |
| `taxonomy` | Current taxonomy term | Taxonomy archives only |
| `{taxonomy_slug}` | Alias for `taxonomy` | Taxonomy archives only |
| `author` | Current author | Author archives only |
| `user` | Alias for `author` | Author archives only |

## Usage in Twig

```twig
{{ site.name }}
{{ site.description }}
{{ globals.theme_url }}
{{ wp_title }}

{# On single posts #}
{{ post.title }}
{{ post.content }}
{{ post.my_acf_field }}

{# On single posts — post type alias #}
{{ project.title }}

{# On taxonomy archives #}
{{ taxonomy.name }}
{{ projectcategory.name }}

{# Options #}
{{ options.primary_color }}
{{ options.blogname }}
```

## Adding custom context

### Auto-discovery

Drop a class extending `Sloth\Context\ContextProvider` into `app/Context/` or `theme/Context/` and Sloth discovers it automatically:

```php
<?php

namespace App\Context;

use Sloth\Context\ContextProvider;

class NavigationContextProvider extends ContextProvider
{
    public function key(): string
    {
        return 'navigation';
    }

    public function resolve(): array
    {
        return [
            'primary' => wp_get_nav_menu_items('primary'),
            'footer'  => wp_get_nav_menu_items('footer'),
        ];
    }
}
```

Available in Twig immediately:

```twig
{% for item in navigation.primary %}
    <a href="{{ item.url }}">{{ item.title }}</a>
{% endfor %}
```

### Manual registration

Register a provider in any service provider's `boot()` method:

```php
public function boot(): void
{
    app('context')->register(new MyContextProvider());
}
```

### Conditional providers

Override `shouldResolve()` to only include the key on certain pages:

```php
class SalesBannerContextProvider extends ContextProvider
{
    public function key(): string
    {
        return 'sales_banner';
    }

    public function shouldResolve(): bool
    {
        return is_front_page() || is_shop();
    }

    public function resolve(): array
    {
        return [
            'text'  => options('sales_banner_text'),
            'color' => options('sales_banner_color', '#ff0000'),
        ];
    }
}
```

### Static values

For one-off values that don't need a full provider:

```php
app('context')->set('current_step', 3);
app('context')->set('show_sidebar', false);
```

## Provider API

```php
abstract class ContextProvider
{
    // The key under which this value is available in Twig
    abstract public function key(): string;

    // Resolve and return the value — called lazily on first access
    abstract public function resolve(): mixed;

    // Whether to include this key — defaults to true
    public function shouldResolve(): bool
    {
        return true;
    }
}
```

## Context in PHP

```php
// Access a single key (lazy)
app('context')['post'];
app('context')['site'];

// Set a value
app('context')['my_key'] = 'value';
app('context')->set('my_key', 'value');

// Register a provider
app('context')->register(new MyContextProvider());

// Get full context as array (resolves all providers)
app('context')->toArray();
```

:::tip Production deployment
After adding new context providers, run `wp sloth manifest:clear` to regenerate the manifest cache. See [Auto-Discovery](auto-discovery#clearing-the-manifest-cache).
:::
