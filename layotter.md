# Layotter

[Layotter](https://github.com/layotter/layotter) is a WordPress page builder plugin, originally developed by Dennis Hingst at [Quäntchen + Glück](https://www.qug.de/). Built before Gutenberg existed, it was an innovative approach to structured content editing — and one of the core inspirations for Sloth itself. Sloth originally grew out of Layotter and [Modulator](https://github.com/qundg/modulator), both largely his work.

:::note
Layotter must be installed and activated as a WordPress plugin for this integration to work.
:::

## Enabling Layotter for a post type

Set `$layotter` on your model:

```php
class Project extends Model
{
    // Enable with default settings
    public static $layotter = true;

    // Enable with custom row layouts
    public static $layotter = [
        'allowed_row_layouts' => ['full', 'half', 'third'],
    ];

    // Disable (default)
    public static $layotter = false;
}
```

## Turning a module into a Layotter element

Set `$layotter` on your module with an ACF field group and display settings:

```php
class HeroModule extends Module
{
    public static $layotter = [
        'field_group' => 'group_hero',   // ACF field group key
        'title'       => 'Hero',
        'description' => 'Full-width hero section',
        'icon'        => 'star',         // Dashicon name
    ];
}
```

Sloth registers the module as a Layotter element automatically. The ACF fields defined in `field_group` are passed to the module as view variables when rendered.

## Backend preview

By default, Layotter shows a table of field values as the backend preview. To use a custom Twig template instead, create a `_layotter` variant of the module template:

```
theme/View/Module/hero.twig           # Frontend template
theme/View/Module/hero_layotter.twig  # Backend preview template
```

## Column classes

Column classes default to a 12-column Bootstrap grid (`col-lg-1` through `col-lg-12`). Publish the config and override via `custom_classes`:

```bash
wp sloth vendor:publish --provider="Sloth\LayotterBridge\LayotterBridgeServiceProvider" --tag=config
```

```php
// app/config/layotter.php
return [
    'row_layouts' => ['full', '1/2', '1/3', '2/3'],

    'custom_classes' => [
        '1/2' => 'col-md-6',
        '1/3' => 'col-md-4',
        '2/3' => 'col-md-8',
    ],
];
```
