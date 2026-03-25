# Prevent N+1 Queries

<a name="introduction"></a>
## Introduction

N+1 query problems are one of the most common performance issues in Laravel applications. They occur when code loads a collection of models and then accesses a relationship on each model individually, resulting in one query for the collection plus one query per model. Laravel provides several tools to prevent this, including eager loading with `with()`, `preventLazyLoading()`, `withCount()`, and selective column loading.

<a name="why"></a>
## Why

- **Dramatically fewer queries**: Eager loading with `with()` reduces N+1 queries down to just 2 queries total, regardless of collection size
- **Early detection**: Enabling `preventLazyLoading()` in development catches N+1 issues before they reach production
- **Efficient counting**: Using `withCount()` avoids loading entire relationships just to count them
- **Reduced memory usage**: Selecting only needed columns avoids loading large text or JSON fields unnecessarily

<a name="suitable-for"></a>
## Suitable For

- Any application that loads relationships on collections of models
- Views or API responses that display related model data
- Applications with growing datasets where query count matters
- Projects with multiple developers where N+1 issues are easily introduced

<a name="less-suitable"></a>
## Less Suitable

- Simple applications with minimal relationships
- Single-model lookups where only one related query is executed

<a name="examples"></a>
## Examples

### Always Eager Load Relationships

```php
// Bad: N+1 — executes 1 + N queries
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->author->name;
}

// Good: 2 queries total
$posts = Post::with('author')->get();
foreach ($posts as $post) {
    echo $post->author->name;
}
```

Constrain eager loads to select only needed columns (always include the foreign key):

```php
$users = User::with(['posts' => function ($query) {
    $query->select('id', 'user_id', 'title')
          ->where('published', true)
          ->latest()
          ->limit(10);
}])->get();
```

### Prevent Lazy Loading in Development

Enable this in `AppServiceProvider::boot()` to catch N+1 issues during development:

```php
public function boot(): void
{
    Model::preventLazyLoading(! app()->isProduction());
}
```

This throws a `LazyLoadingViolationException` when a relationship is accessed without being eager-loaded.

### Use `withCount()` for Counting Relations

```php
// Bad: loads entire collections just to count them
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->comments->count();
}

// Good: adds a comments_count attribute via a single subquery
$posts = Post::withCount('comments')->get();
foreach ($posts as $post) {
    echo $post->comments_count;
}
```

### Select Only Needed Columns

```php
// Bad: SELECT * on both tables
$posts = Post::with('author')->get();

// Good: only fetches what you need
$posts = Post::select('id', 'title', 'user_id', 'created_at')
    ->with(['author:id,name,avatar'])
    ->get();
```

When selecting columns on eager-loaded relationships, always include the foreign key column or the relationship won't match.

<a name="more-info"></a>
## More Info

- [Laravel Eager Loading Documentation](https://laravel.com/docs/eloquent-relationships#eager-loading)
- [Laravel `preventLazyLoading` Documentation](https://laravel.com/docs/eloquent-relationships#preventing-lazy-loading)
- [Laravel Counting Related Models](https://laravel.com/docs/eloquent-relationships#counting-related-models)
- [Use Chunking for Large Datasets](./use-chunking-for-large-datasets.md) — for processing large collections efficiently
- [Use Eloquent Scopes and Casts](./use-eloquent-scopes-and-casts.md) — for reusable query constraints
- [Laravel Boost Best Practices PR](https://github.com/laravel/boost/pull/628)
