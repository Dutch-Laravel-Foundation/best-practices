# Use Route Model Binding

<a name="introduction"></a>
## Introduction

Laravel's implicit route model binding automatically resolves Eloquent models from route parameters, eliminating manual `findOrFail()` calls. Combined with scoped bindings for nested resources and resource controllers, this keeps routing code concise, consistent, and less error-prone.

<a name="why"></a>
## Why

- **Less boilerplate**: No need for manual `findOrFail()` or `find()` calls — Laravel resolves the model automatically
- **Automatic 404 handling**: If the model isn't found, Laravel returns a 404 response without any extra code
- **Parent-child enforcement**: Scoped bindings ensure nested resources actually belong to their parent, preventing unauthorized access
- **RESTful consistency**: Resource controllers enforce standard CRUD naming conventions across the application

<a name="suitable-for"></a>
## Suitable For

- Any route that operates on a specific model instance
- Nested resource routes (e.g., `/users/{user}/posts/{post}`)
- RESTful APIs and CRUD controllers
- Applications where consistent URL patterns improve developer experience

<a name="less-suitable"></a>
## Less Suitable

- Routes that need custom resolution logic beyond simple key lookups
- Endpoints that don't operate on specific model instances
- Legacy routes with non-standard parameter naming

<a name="examples"></a>
## Examples

### Implicit Route Model Binding

```php
// Bad: manual resolution
public function show(int $id)
{
    $post = Post::findOrFail($id);
}

// Good: automatic resolution with type-hinting
public function show(Post $post)
{
    return view('posts.show', ['post' => $post]);
}
```

### Scoped Bindings for Nested Resources

Enforce parent-child relationships automatically:

```php
Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    // $post is automatically scoped to $user
})->scopeBindings();
```

### Use Resource Controllers

```php
Route::resource('posts', PostController::class);
Route::apiResource('api/posts', Api\PostController::class);
```

<a name="more-info"></a>
## More Info

- [Laravel Route Model Binding Documentation](https://laravel.com/docs/routing#route-model-binding)
- [Laravel Resource Controllers Documentation](https://laravel.com/docs/controllers#resource-controllers)
- [Use Action Classes for Business Logic](../project-structure-and-code-architecture/use-action-classes-for-business-logic.md) — keep controllers thin by extracting logic to action classes
- [Use Form Request Classes](./use-form-request-classes.md) — extract validation from controllers into Form Requests
- [Laravel Boost Best Practices PR](https://github.com/laravel/boost/pull/628)
