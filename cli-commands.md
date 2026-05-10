# CLI Commands

Sloth integrates with WP-CLI and exposes all commands under the `wp sloth` namespace. Commands are discovered automatically from `app/Console/`, `theme/Console/` and the framework itself.

## Built-in commands

| Command | Description | See also |
|---------|-------------|----------|
| `wp sloth manifest:clear` | Clear all manifest files — regenerated on next request | [Auto-Discovery](auto-discovery) |
| `wp sloth config:get <key>` | Read a config value by dot-notation key | [Configuration](configuration) |
| `wp sloth vendor:publish` | Publish config and view files from a package | [Service Providers](service-providers), [Configuration](configuration) |

## Writing custom commands

Drop a class extending `Sloth\Console\Command` into `app/Console/` or `theme/Console/` and Sloth discovers it automatically.

```php
<?php

namespace App\Console;

use App\Model\Project;
use Sloth\Console\Command;

class SyncProjectsCommand extends Command
{
    protected $signature = 'projects:sync
        {--dry-run : Show what would be synced without making changes}';

    protected $description = 'Sync projects from the external API';

    public function handle(): int
    {
        $this->info('Syncing projects...');

        $projects = fetchFromApi();

        if ($this->option('dry-run')) {
            $this->warn('Dry run — no changes made.');
            $this->table(['ID', 'Title'], $projects->map(fn($p) => [$p->id, $p->title]));

            return self::SUCCESS;
        }

        foreach ($projects as $project) {
            Project::updateOrCreate(['post_name' => $project->slug], [
                'post_title' => $project->title,
            ]);

            $this->line("  <fg=green>✓</> {$project->title}");
        }

        $this->newLine();
        $this->info("Synced {$projects->count()} project(s).");

        return self::SUCCESS;
    }
}
```

```bash
wp sloth projects:sync
wp sloth projects:sync --dry-run
```

## Registering commands from a service provider

Packages can register commands via `$this->commands()` in `boot()`. Commands registered this way are picked up automatically by `wp sloth` — no directory conventions required:

```php
class MyPackageServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        $this->commands([
            MyPackageCommand::class,
        ]);
    }
}
```

## Command API

Commands extend `Sloth\Console\Command` which extends Laravel's `Illuminate\Console\Command`. The full [Laravel console documentation](https://laravel.com/docs/artisan#writing-commands) applies:

```php
// Arguments and options
$this->argument('name');
$this->option('dry-run');

// Output
$this->info('Done!');
$this->warn('Watch out!');
$this->error('Something went wrong.');
$this->line('Plain text');
$this->table(['Col 1', 'Col 2'], $rows);

// Progress bar
$bar = $this->output->createProgressBar(count($items));
$bar->finish();

// Exit codes
return self::SUCCESS;  // 0
return self::FAILURE;  // 1
```
