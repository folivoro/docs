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
│   └── Providers/        # Service providers
├── public/
│   └── wp-content/
│       ├── mu-plugins/
│       │   └── cecropia.php   # Bootstraps Sloth before the theme loads
│       └── themes/
│           └── my-theme/      # The active theme (see Theme structure below)
├── vendor/
├── composer.json
└── .env
```

The project root — where `composer.json` lives — is the base path. `app/` holds application logic that is shared across themes and independent of any single theme.

## Theme mode

```
my-theme/
├── app/                  # Same conventions as Classic mode's app/ — optional
├── Includes/
├── Model/
├── Taxonomy/
├── Module/
├── Api/
├── Providers/
├── config/
├── View/
│   ├── Layout/           # Twig templates matching the WordPress template hierarchy
│   └── Module/           # Module templates
├── cache/                # Auto-created by Sloth
├── logs/                 # Auto-created by Sloth
├── functions.php         # Bootstraps Sloth
├── composer.json
└── .env
```

The theme directory is the base path. Everything lives inside the theme.

## Theme structure (both modes)

The active theme directory always follows this layout:

```
my-theme/
├── View/
│   ├── Layout/           # Template hierarchy templates
│   │   ├── index.twig
│   │   ├── single.twig
│   │   ├── single-project.twig
│   │   ├── archive-project.twig
│   │   ├── page.twig
│   │   └── 404.twig
│   └── Module/           # Module templates
│       ├── hero.twig
│       └── featured-projects.twig
├── cache/                # Auto-created — add to .gitignore
├── logs/                 # Auto-created — add to .gitignore
└── functions.php
```

## Auto-discovery

Sloth scans both `app/{Directory}/` and `theme/{Directory}/` for each concern. This means in Classic mode you can split classes between `app/` and `theme/` freely — a model shared across themes lives in `app/Model/`, a theme-specific module lives in `theme/Module/`.

| Directory | What Sloth discovers |
|-----------|----------------------|
| `Model/` | Custom post type models |
| `Taxonomy/` | Custom taxonomies |
| `Module/` | UI modules |
| `Api/` | REST API controllers |
| `Providers/` | Service providers |
| `Includes/` | PHP files required automatically on `init` |
| `config/` | Configuration files loaded into `config()` |

## Registered paths

All paths are accessible at runtime via `app()->path()`:

```php
app()->path();              // project root (basePath)
app()->path('app');         // app/ directory
app()->path('theme');       // active theme directory
app()->path('cache');       // theme/cache/
app()->path('logs');        // theme/logs/
app()->path('uploads');     // WordPress uploads directory
app()->path('vendor');      // vendor/ directory
app()->path('cms');         // WordPress ABSPATH

// Subdirectories
app()->path('Model');              // app/Model/
app()->path('Model', 'theme');     // theme/Model/
app()->path('config', 'app');      // app/config/
```
