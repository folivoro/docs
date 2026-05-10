# Events

Sloth provides two complementary ways to react to WordPress hooks: the **declarative hook system** in service providers, and the **WordPress Event Bridge** which connects WordPress hooks to Laravel's event dispatcher.

## Hooks and filters vs. events — when to use which

WordPress has two native extension points: actions (fire-and-forget) and filters (modify a value). Sloth wraps both in a declarative API via `getHooks()` and `getFilters()` in service providers.

The **Event Bridge** goes one step further — it re-dispatches selected WordPress hooks as Laravel events. This enables a more decoupled architecture where multiple listeners can react to the same hook independently, without knowing about each other.

| | `getHooks()` / `getFilters()` | Event Bridge |
|---|---|---|
| **Best for** | Provider-local concerns | Shared, cross-cutting concerns |
| **Coupling** | Callback registered directly | Listener registered independently |
| **Multiple listeners** | Possible but manual | Built-in via Laravel's dispatcher |
| **Filter modification** | Return value from callback | Set `$event->result` |
| **Performance** | Direct `add_action()` call | Near-zero if no listeners |

**Use `getHooks()` / `getFilters()`** when only one provider cares about the hook — registering a post type on `init`, enqueueing scripts on `wp_enqueue_scripts`, and so on.

**Use the Event Bridge** when multiple components need to react to the same hook, or when you want to decouple the reaction from the hook registration entirely.

## getHooks() and getFilters()

The simplest way to hook into WordPress. Declare hooks in a service provider and Sloth registers them after all providers have booted:

```php
public function getHooks(): array
{
    return [
        'init'      => fn() => $this->registerPostTypes(),
        'wp_loaded' => fn() => $this->setup(),
    ];
}

public function getFilters(): array
{
    return [
        'the_title'   => fn(string $title) => strtoupper($title),
        'body_class'  => fn(array $classes) => [...$classes, 'my-class'],
    ];
}
```

See [Service Providers](service-providers) for all supported formats.

## The Event Bridge

The Event Bridge wraps a set of WordPress hooks with `add_action()` / `add_filter()` and re-dispatches them as Laravel events named `wp:{hook}`.

### Listening to actions

```php
use Sloth\Event\WpHookFired;
use Sloth\Facades\Event;

// In a service provider's boot() method
Event::listen('wp:init', function (WpHookFired $event) {
    // Runs when WordPress 'init' fires
});

Event::listen('wp:wp_loaded', function (WpHookFired $event) {
    dump($event->hook);  // 'wp_loaded'
    dump($event->args);  // arguments passed to do_action()
    dump($event->type);  // 'action'
});
```

### Modifying filters

For filter hooks, set `$event->result` to change the value returned by `apply_filters()`:

```php
Event::listen('wp:the_content', function (WpHookFired $event) {
    $event->result = '<div class="content">' . $event->result . '</div>';
});

Event::listen('wp:body_class', function (WpHookFired $event) {
    $event->result[] = 'my-global-class';
});
```

### Bridged hooks

These WordPress hooks are bridged by default:

**Actions**

| Hook | When it fires |
|------|---------------|
| `muplugins_loaded` | MU-plugins loaded |
| `plugins_loaded` | All plugins loaded |
| `after_setup_theme` | Theme `functions.php` loaded |
| `init` | WordPress fully initialized |
| `wp_loaded` | All WordPress setup complete |
| `template_redirect` | Before template is loaded |
| `wp_head` | Inside `<head>` |
| `wp_footer` | Before `</body>` |
| `shutdown` | PHP shutdown |

**Filters**

| Hook | What it filters |
|------|-----------------|
| `the_content` | Post content before display |
| `the_title` | Post title before display |
| `the_excerpt` | Post excerpt before display |
| `body_class` | HTML body classes |

### Adding custom hooks to the bridge

Register additional hooks at runtime via the bridge directly:

```php
use Sloth\Event\WordPressEventBridge;

public function boot(): void
{
    $bridge = app(WordPressEventBridge::class);

    // Bridge a custom action
    $bridge->addHook('my_custom_action', 'action');

    // Bridge a custom filter
    $bridge->addHook('my_custom_filter', 'filter');
}
```

Or add them to `app/config/events.php`:

```php
return [
    'bridge' => [
        'save_post'        => 'action',
        'wp_nav_menu_args' => 'filter',
    ],
];
```

### Performance

The bridge only dispatches events when there are active Laravel listeners. If nothing is listening to `wp:the_content`, the WordPress callback returns immediately without creating any objects. Adding hooks to the bridge configuration has near-zero performance impact.

## Laravel Events

Beyond WordPress hooks, you can dispatch and listen to your own Laravel events:

```php
// Define an event
class ProjectPublished
{
    public function __construct(
        public readonly Project $project,
    ) {}
}

// Dispatch it
Event::dispatch(new ProjectPublished($project));

// Listen to it
Event::listen(ProjectPublished::class, function (ProjectPublished $event) {
    // Send notification, invalidate cache, etc.
});
```

## Further reading

- [Laravel Events](https://laravel.com/docs/events) — listeners, subscribers, queued events
