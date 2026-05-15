# CLI Commands

Sloth integrates with WP-CLI and exposes all commands under the `wp sloth` namespace. Commands are discovered automatically from `app/Console/`, `theme/Console/` and the framework itself.

## Built-in commands

| Command | Description |
|---------|-------------|
| `wp sloth make:module` | Generate a Module class + Twig template |
| `wp sloth make:model` | Generate a Model class |
| `wp sloth make:provider` | Generate a Service Provider |
| `wp sloth make:extension` | Generate a View Extension |
| `wp sloth make:layout` | Generate a Layout Twig template |
| `wp sloth make:command` | Generate a WP-CLI Command |
| `wp sloth make:api-controller` | Generate an API Controller |
| `wp sloth stub:publish` | Publish stubs to the project for customisation |
| `wp sloth manifest:clear` | Clear all manifest caches |
| `wp sloth config:get <key>` | Read a config value by dot-notation key |
| `wp sloth vendor:publish` | Publish config and view files from a package |

## Scaffolding

The `make:*` commands generate boilerplate files. Each command is documented on the relevant page:

- `make:module` → [Modules](modules#generating-a-module)
- `make:model` → [Models](models#generating-a-model)
- `make:provider` → [Service Providers](service-providers#generating-a-provider)
- `make:extension` → [Views](views#generating-an-extension)
- `make:layout` → [Views](views#generating-a-layout)

### make:command

Generates a new WP-CLI Command class:

```bash
wp sloth make:command SyncProjectsCommand
```

Creates `app/Console/Commands/SyncProjectsCommand.php` (or `theme/Console/Commands/` in Theme mode).

### make:api-controller

Generates a new REST API Controller:

```bash
wp sloth make:api-controller ProjectController
```

Creates `app/Api/ProjectController.php` with a default `index()` endpoint.

### stub:publish

Publishes all framework stubs to `stubs/` in the project root so you can customise them. Once published, all `make:*` commands use your local stubs instead of the framework defaults:

```bash
wp sloth stub:publish

# Overwrite existing stubs
wp sloth stub:publish --force
```

## Writing custom commands

Drop a class extending `Sloth\Console\Command` into `app/Console/Commands/` or `theme/Console/Commands/` and Sloth discovers it automatically. Or generate one:

```bash
wp sloth make:command SyncProjectsCommand
```

```php
<?php

namespace App\Console\Commands;

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

Packages can register commands via `$this->commands()` in `boot()`:

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
