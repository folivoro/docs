---
title: Quick Start
---

# Quick Start

## Creating a Module

Use the scaffolder to create a new module:

```bash
php sloth-cli.php make:module MyModule
```

## Defining Routes

Create a `routes.php` file in your theme root:

```php
<?php

use Sloth\Facades\Route;

Route::get('/about', [
    'controller' => 'PageController',
    'action' => 'about',
]);
```

## Using Models

```php
<?php

use Sloth\Model\Post;

$post = Post::find(123);
$posts = Post::where('category', 'news')->get();
```

## Validation

```php
<?php

use Sloth\Facades\Validation;

$validator = Validation::make($request->all(), [
    'name'  => 'required|min:3|max:255',
    'email' => 'required|email',
]);

if ($validator->fails()) {
    $errors = $validator->errors();
}
```