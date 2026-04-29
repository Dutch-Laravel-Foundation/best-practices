# Use Caching Effectively

<a name="introduction"></a>
## Introduction

Laravel's caching layer provides several patterns beyond simple get/put operations. Using `Cache::remember()`, `Cache::flexible()`, `Cache::memo()`, `once()`, cache tags, and failover stores properly can dramatically improve application performance while avoiding common pitfalls like race conditions and stale data.

<a name="why"></a>
## Why

- **Reduced boilerplate**: `Cache::remember()` replaces manual get/check/put patterns with an atomic operation
- **Better user experience**: `Cache::flexible()` serves stale data while refreshing in the background, so no user takes the slow-path hit
- **Fewer round-trips**: `Cache::memo()` and `once()` eliminate redundant cache or computation calls within a single request
- **Clean invalidation**: Cache tags allow flushing related groups of entries atomically
- **Resilience**: Failover cache stores keep the application running when the primary cache goes down

<a name="suitable-for"></a>
## Suitable For

- Applications with expensive database queries or API calls that don't need to be real-time
- High-traffic endpoints where cache stampedes are a concern
- Applications with related data that needs to be invalidated together
- Production environments where cache store reliability matters

<a name="less-suitable"></a>
## Less Suitable

- Data that must always be fresh (financial transactions, real-time inventory)
- Development environments where caching obscures bugs
- Simple applications with fast queries and low traffic

<a name="examples"></a>
## Examples

### Use `Cache::remember()` Instead of Manual Get/Put

```php
// Bad: race condition, boilerplate
$val = Cache::get('stats');
if (! $val) {
    $val = $this->computeStats();
    Cache::put('stats', $val, 60);
}

// Good: atomic pattern
$val = Cache::remember('stats', 60, fn () => $this->computeStats());
```

### Use `Cache::flexible()` for Stale-While-Revalidate

On high-traffic keys, one user always gets a slow response when the cache expires. `flexible()` serves slightly stale data while refreshing in the background:

```php
// Bad: one unlucky user waits for recomputation
Cache::remember('users', 300, fn () => User::all());

// Good: fresh for 5 min, stale-but-served up to 10 min, refreshes via deferred function
Cache::flexible('users', [300, 600], fn () => User::all());
```

### Use `Cache::memo()` to Avoid Redundant Hits

If the same cache key is read multiple times per request, `memo()` stores the resolved value in memory:

```php
Cache::memo()->get('settings'); // 5 calls = 1 Redis round-trip instead of 5
```

### Use Cache Tags to Invalidate Related Groups

```php
Cache::tags(['user-1'])->flush();
```

> **Note:** Tags only work with `redis`, `memcached`, `dynamodb` — not `file` or `database`.

### Use `once()` for Per-Request Memoization

`once()` memoizes a function's return value for the lifetime of the object — pure in-memory, no cache store hit:

```php
public function roles(): Collection
{
    return once(fn () => $this->loadRoles());
}
```

### Use `Cache::add()` for Atomic Conditional Writes

```php
// Bad: race condition between check and write
if (! Cache::has('lock')) {
    Cache::put('lock', true, 10);
}

// Good: atomic — only writes if key doesn't exist
Cache::add('lock', true, 10);
```

### Configure Failover Cache Stores in Production

```php
// config/cache.php
'failover' => [
    'driver' => 'failover',
    'stores' => ['redis', 'database'],
],
```

<a name="more-info"></a>
## More Info

- [Laravel Cache Documentation](https://laravel.com/docs/cache)
- [Laravel Boost Best Practices PR](https://github.com/laravel/boost/pull/628)
