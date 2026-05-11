# Advanced Custom Fields

Sloth integrates with [Advanced Custom Fields](https://www.advancedcustomfields.com/) (ACF) at the model level. ACF fields are resolved automatically on model retrieval — no extra configuration needed.

:::note
ACF must be installed and activated as a WordPress plugin for this integration to work.
:::

## Model integration

ACF fields are available directly as properties on any model:

```php
$project = Project::find(1);

// Direct access
$project->my_acf_field;

// Via ACF proxy
$project->acf->my_acf_field;
```

Fields are fetched via `get_fields()` on retrieval and cached per model instance. ACF field types are cast automatically — images return Image objects, relationships return model collections, and so on.

## Options pages

Sloth provides a dedicated Options API for ACF options pages. See [Options](options) for the full documentation.

```php
use Sloth\Facades\Options;

Options::get('my_option_field');
```

In Twig:

```twig
{{ options.my_option_field }}
```

## ACF in Twig

The `get_field()` function is available in all templates:

```twig
{{ get_field('my_field') }}
{{ get_field('my_field', post.ID) }}
```

## ACF and Modules (Layotter)

When used with Layotter, ACF field group values are passed to the module automatically as view variables. See [Layotter](layotter) for details.
