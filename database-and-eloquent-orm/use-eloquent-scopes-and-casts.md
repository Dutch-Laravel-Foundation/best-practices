# Use Eloquent Scopes and Casts

<a name="introduction"></a>
## Introduction

Eloquent provides local scopes for reusable query constraints and attribute casts for automatic type conversion. Using these features keeps query logic DRY, ensures consistent data types, and makes code more expressive. Combined with helpers like `whereBelongsTo()`, they make Eloquent queries cleaner and less error-prone.

<a name="why"></a>
## Why

- **DRY queries**: Local scopes extract reusable query constraints, eliminating duplicated `where` clauses across the codebase
- **Type safety**: Attribute casts automatically convert database values to the correct PHP types (booleans, arrays, dates, decimals)
- **Cleaner queries**: `whereBelongsTo()` eliminates hardcoded foreign key references, making relationship queries more readable
- **Better templates**: Casting date columns means you can use Carbon methods directly in Blade templates instead of manually parsing strings

<a name="suitable-for"></a>
## Suitable For

- Any model with repeated query patterns (active users, published posts, etc.)
- Models with JSON, boolean, decimal, or date columns
- Applications using Blade or API responses that format dates

<a name="less-suitable"></a>
## Less Suitable

- One-off queries that aren't reused anywhere
- Global scopes should be used sparingly — prefer local scopes for most filtering needs

<a name="examples"></a>
## Examples

### Local Scopes

```php
// Bad: duplicated query logic
$active = User::where('verified', true)->whereNotNull('activated_at')->get();
$articles = Article::whereHas('user', function ($q) {
    $q->where('verified', true)->whereNotNull('activated_at');
})->get();

// Good: reusable local scope
public function scopeActive(Builder $query): Builder
{
    return $query->where('verified', true)->whereNotNull('activated_at');
}

$active = User::active()->get();
$articles = Article::whereHas('user', fn ($q) => $q->active())->get();
```

### Global Scopes — Use Sparingly

Global scopes silently modify every query on the model, making debugging difficult. Reserve them for truly universal constraints like soft deletes or multi-tenancy. Prefer local scopes for everything else.

### Attribute Casts

Use the `casts()` method for automatic type conversion:

```php
protected function casts(): array
{
    return [
        'is_active' => 'boolean',
        'metadata' => 'array',
        'total' => 'decimal:2',
    ];
}
```

### Cast Date Columns Properly

```php
// Bad: manual date parsing in templates
{{ Carbon::createFromFormat('Y-d-m H-i', $order->ordered_at)->toDateString() }}

// Good: cast in the model
protected function casts(): array
{
    return [
        'ordered_at' => 'datetime',
    ];
}

// Then use directly in Blade
{{ $order->ordered_at->toDateString() }}
{{ $order->ordered_at->format('m-d') }}
```

### Use `whereBelongsTo()`

```php
// Bad: hardcoded foreign key
Post::where('user_id', $user->id)->get();

// Good: cleaner and relationship-aware
Post::whereBelongsTo($user)->get();
Post::whereBelongsTo($user, 'author')->get();
```

<a name="more-info"></a>
## More Info

- [Laravel Local Scopes Documentation](https://laravel.com/docs/eloquent#local-scopes)
- [Laravel Attribute Casting Documentation](https://laravel.com/docs/eloquent-mutators#attribute-casting)
- [Laravel Boost Best Practices PR](https://github.com/laravel/boost/pull/628)
