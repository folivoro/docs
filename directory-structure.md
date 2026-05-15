# Directory Structure

Sloth scans two directories for classes: `app/` and `theme/`. Which directories actually exist depends on the mode.

## Classic mode

```
my-project/
├── app/
│   ├── config/           # Configuration files
│   ├── Includes/         # PHP files loaded automatically on init
│   ├── Model/            # Custom post type models
│   ├── Taxonomy/         # Custom taxonomies
│   ├── Module/           # Reusable UI modules
│   ├── Api/              # REST API controllers
│   ├── Console/Commands/ # WP-CLI commands
│   ├── Extensions/View/  # Twig helpers and directives
│   └── Providers/        # Service providers
├── public/
│   └── wp-content/
│       ├── mu-plugins/
│       │   └── cecropia.php   # Bootstraps Sloth before the theme loads
│       └── themes/
│           └── my-theme/      # The active theme (see Theme structure below)
├── storage/              # Auto-created — add to .gitignore
│   ├── cache/            # Twig and manifest cache
│   └── logs/             # Application logs
├── stubs/                # Optional — published via wp sloth stub:publish
├── vendor/
├── composer.json
└── .env
```

The project root — where `composer.json` lives — is the base path. `app/` holds application logic shared across themes. `storage/` always lives at the project root, never inside `app/`.

## Theme mode

```
my-theme/
├── config/               # Configuration files
├── Includes/
├── Model/
├── Taxonomy/
├── Module/
├── Api/
├── Console/Commands/
├── Extensions/View/
├── Providers/
├── View/
│   ├── Layout/           # Twig templates matching the WordPress template hierarchy
│   └── Module/           # Module templates
├── storage/              # Auto-created — add to .gitignore
│   ├── cache/
│   └── logs/
├── stubs/                # Optional — published via wp sloth stub:publish
├── functions.php         # Bootstraps Sloth
├── composer.json
└── .env
```

The theme directory is both the project root and the app root. Everything lives inside the theme.

## Theme structure (both modes)

The active theme always has a `View/` directory:

```
my-theme/
└── View/
    ├── Layout/           # Template hierarchy templates
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

## Auto-discovery

Sloth scans both `app/{Directory}/` and `theme/{Directory}/` for each concern:

| Directory | What Sloth discovers |
|-----------|----------------------|
| `Model/` | Custom post type models |
| `Taxonomy/` | Custom taxonomies |
| `Module/` | UI modules |
| `Api/` | REST API controllers |
| `Providers/` | Service providers |
| `Context/` | Template context providers |
| `Extensions/View/` | Twig helpers and directives |
| `Console/Commands/` | WP-CLI commands |
| `Includes/` | PHP files required on `init` |
| `config/` | Configuration files |

## Registered paths

All paths are accessible via typed accessors:

```php
app()->basePath();          // project root (where composer.json lives)
app()->appPath();           // app/ in Classic, theme root in Theme mode
app()->themePath();         // active theme directory
app()->configPath();        // app/config/ or theme/config/
app()->storagePath();       // project root storage/
app()->cachePath();         // storage/cache/
app()->logsPath();          // storage/logs/
app()->uploadsPath();       // WordPress uploads directory
app()->cmsPath();           // WordPress ABSPATH
app()->pluginsPath();       // WordPress plugins directory

// With subdirectory
app()->appPath('Model');        // app/Model/
app()->themePath('View');       // theme/View/
app()->cachePath('Twig');       // storage/cache/Twig/
```
