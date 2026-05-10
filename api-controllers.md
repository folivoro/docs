# API Controllers

Sloth can auto-discover REST API controllers from `app/Api/` or `theme/Api/`. Drop a class extending `Sloth\Api\Controller` in either directory and Sloth registers the endpoints automatically under `/wp-json/sloth/v1/`.

:::tip
Use API controllers for structured, resource-oriented endpoints. For quick one-off endpoints or module data, [Modules](modules) with `$json` enabled may be a simpler fit.
:::

## Creating a controller

```php
<?php

namespace App\Api;

use App\Model\Project;
use Sloth\Api\Controller;

class ProjectController extends Controller
{
    public function index(): array
    {
        return Project::all()->toArray();
    }

    public function single(int $id): array
    {
        $project = Project::find($id);

        if (!$project) {
            $this->setStatusCode(404);
            return [];
        }

        return $project->toArray();
    }

    public function featured(): array
    {
        return Project::taxonomy('category', 'featured')->get()->toArray();
    }
}
```

## Route mapping

Routes are derived from the class and method names automatically. The class name determines the route prefix — `ProjectController` becomes `project`:

| Method | Route |
|--------|-------|
| `index()` | `GET /wp-json/sloth/v1/project` |
| `single()` | `GET /wp-json/sloth/v1/project/{id}` |
| `featured()` | `GET /wp-json/sloth/v1/project/featured/{id?}` |

The rules are:

- If the controller has a `single()` method: `index()` maps to `/{prefix}`, `single()` maps to `/{prefix}/{id?}`
- If no `single()` method: `index()` maps to `/{prefix}/{id?}`
- All other public methods map to `/{prefix}/{method-name}/{id?}`
- Methods starting with `_` are excluded

All routes accept `GET`, `POST`, `PUT` and `DELETE`. Inspect `$this->request->get_method()` inside the controller to branch by method.

## Accessing the request

```php
public function index(): array
{
    $method = $this->request->get_method();       // GET, POST, …
    $params = $this->request->get_query_params();  // ?foo=bar
    $body   = $this->request->get_json_params();   // JSON body
    $id     = $this->request->get_param('id');     // URL param
}
```

## Status codes

```php
public function single(int $id): array
{
    $project = Project::find($id);

    if (!$project) {
        $this->setStatusCode(404);
        return [];
    }

    return $project->toArray();
}
```

Error responses automatically include the HTTP status text:

```json
{ "code": 404, "message": "Not Found" }
```

## URL generation

Generate URLs to controller endpoints from PHP:

```php
$controller = new ProjectController();
$controller->getUrl('featured');            // /wp-json/sloth/v1/project/featured
$controller->getUrl('featured', ['page' => 2]); // …?page=2
```

:::tip Production deployment
After deploying new or renamed API controllers, run `wp sloth manifest:clear` to regenerate the manifest cache. See [Auto-Discovery](auto-discovery#clearing-the-manifest-cache).
:::
