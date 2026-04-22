---
title: CLI Commands
---

# WP-CLI Commands

Sloth provides Artisan-style console commands available via `wp sloth`:

```bash
# List all available commands
wp sloth list

# Display a welcome message
wp sloth inspire

# Clear all manifest files
wp sloth manifest:clear
```

## Available Commands

| Command | Description |
|---------|-------------|
| `wp sloth inspire` | Display a welcome message |
| `wp sloth manifest:clear` | Clear all Sloth manifest files |
| `wp sloth list` | List all available commands |

## Creating Custom Commands

Create commands in your theme's `app/Console/Commands/` directory:

```php
<?php

namespace App\Console\Commands;

use Sloth\Console\Command;

class MyCommand extends Command
{
    protected $signature = 'my:command';
    protected $description = 'My custom command';

    public function handle(): int
    {
        $this->info('Done!');
        return self::SUCCESS;
    }
}
```