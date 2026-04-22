---
title: Configuration
---

# Configuration

## Environment Variables

Create a `.env` file in your theme root:

```env
APP_ENV=local
WP_DEBUG=true
DATABASE_HOST=localhost
DATABASE_NAME=wordpress
DATABASE_USER=root
DATABASE_PASSWORD=
```

## Custom Configuration

Add configuration files in `src/config/`:

```php
<?php

return [
    'setting_name' => 'value',
    'another_setting' => true,
];
```

Access configuration via:

```php
<?php

use Sloth\Facades\Configure;

$value = Configure::get('config_file.setting_name');
```