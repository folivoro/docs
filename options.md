# Options

Sloth provides a unified API over WordPress Core `get_option()` and ACF Options pages. The resolution order is always: ACF option field first (if ACF is active), WordPress core option as fallback.

WordPress' own object cache handles caching — no additional layer needed.

## PHP

```php
use Sloth\Facades\Options;

// Get
Options::get('blogname');
Options::get('primary_color');           // ACF option field
Options::get('my_option', 'fallback');   // with default

// Set
Options::set('my_option', 'value');

// Check
Options::has('my_option');

// Delete
Options::delete('my_option');
```

Via the global helper:

```php
options('blogname');
options('primary_color', '#000');  // with default
options();                          // Options instance
```

Via the container:

```php
app('options')->get('blogname');
app('options')->primary_color;     // magic property access
```

## Twig

```twig
{# Via the options global — dot notation #}
{{ options.blogname }}
{{ options.primary_color }}

{# Via the options() function — with optional default #}
{{ options('blogname') }}
{{ options('primary_color', '#000') }}
```

## Resolution order

For every key, Sloth resolves in this order:

1. **ACF option field** — `get_field($key, 'option')` — if ACF is active and the field exists
2. **WordPress core option** — `get_option($key)`
3. **Default value** — the value passed as second argument, or `null`

This means standard WordPress options (`blogname`, `blogdescription`, `admin_email` etc.) are accessible through the same API as ACF option fields.

## Writing options

`Options::set()` always uses `update_option()` — ACF option fields are managed via ACF's own save mechanism and should not be written directly.

```php
Options::set('my_option', 'value');   // update_option()
Options::delete('my_option');         // delete_option()
```

## ACF Options pages

To use ACF option fields, register an ACF options page as usual:

```php
// In a ServiceProvider boot() or functions.php
acf_add_options_page([
    'page_title' => 'Theme Settings',
    'menu_title' => 'Theme Settings',
    'menu_slug'  => 'theme-settings',
]);
```

Fields registered on the options page are then accessible via `Options::get()` or `options()` automatically.
