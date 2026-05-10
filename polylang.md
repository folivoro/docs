# Polylang

[Polylang](https://polylang.pro/) is a WordPress multilingual plugin. Sloth detects it automatically. When the plugin is active, its translation functions are registered in Twig without any additional configuration.

:::note
Polylang must be installed and activated as a WordPress plugin for this integration to work.
:::

## Twig functions

```twig
{# Translate a string #}
{{ pll__('Hello') }}

{# Translate and echo #}
{{ pll_e('Hello') }}
```

These functions are only registered when Polylang is active — they are silently skipped otherwise.

## PHP

Use Polylang's functions directly in PHP as usual:

```php
pll__('Hello');
pll_e('Hello');
pll_current_language();
```

No Sloth-specific wrapper is needed.
