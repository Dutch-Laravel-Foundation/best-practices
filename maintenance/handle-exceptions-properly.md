# Handle Exceptions Properly

<a name="introduction"></a>
## Introduction

Laravel provides flexible exception handling through two approaches: co-locating behavior on exception classes or centralizing it in `bootstrap/app.php`. Either approach works — the key is picking one and applying it consistently. Beyond that, Laravel offers tools for suppressing noise (throttling, deduplication), forcing correct API error formats, and attaching structured context to exceptions.

<a name="why"></a>
## Why

- **Consistent handling**: Choosing one approach (co-located or centralized) prevents scattered, hard-to-find exception behavior
- **Reduced noise**: Throttling high-volume exceptions and deduplicating reports protects log sinks and error tracking budgets
- **Correct API responses**: Explicitly declaring JSON rendering for API routes prevents HTML error pages from being sent to API clients
- **Better debugging**: Structured context on exception classes provides rich metadata in log entries without manual logging

<a name="suitable-for"></a>
## Suitable For

- Applications with custom exception types
- APIs that need consistent JSON error responses
- Applications integrating with error tracking services (Sentry, Flare, Bugsnag)
- High-traffic applications where error volume can overwhelm logging

<a name="less-suitable"></a>
## Less Suitable

- Simple applications where default exception handling is sufficient
- Early-stage prototypes where structured error handling adds overhead

<a name="examples"></a>
## Examples

### Choose One Approach and Be Consistent

**Co-location on the exception class** — keeps behavior alongside the definition:

```php
class InvalidOrderException extends Exception
{
    public function report(): void { /* custom reporting */ }

    public function render(Request $request): Response
    {
        return response()->view('errors.invalid-order', status: 422);
    }
}
```

**Centralized in `bootstrap/app.php`** — all exception handling in one place:

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (InvalidOrderException $e, Request $request) {
        return response()->view('errors.invalid-order', status: 422);
    });
})
```

Check the existing codebase and follow whichever pattern is already established.

### Use `ShouldntReport` for Exceptions That Should Never Log

```php
class PodcastProcessingException extends Exception implements ShouldntReport {}
```

### Force JSON Error Rendering for API Routes

Laravel auto-detects `Accept: application/json` but API clients may not set it:

```php
$exceptions->shouldRenderJsonWhen(function (Request $request, Throwable $e) {
    return $request->is('api/*') || $request->expectsJson();
});
```

### Add Structured Context to Exceptions

```php
class InvalidOrderException extends Exception
{
    public function context(): array
    {
        return ['order_id' => $this->orderId];
    }
}
```

Laravel automatically includes this data in the log entry.

### Throttle High-Volume Exceptions

A single failing integration can flood error tracking. Use `throttle()` to rate-limit per exception type.

### Enable `dontReportDuplicates()`

Prevents the same exception instance from being logged multiple times when `report($e)` is called in multiple catch blocks.

<a name="more-info"></a>
## More Info

- [Laravel Error Handling Documentation](https://laravel.com/docs/errors)
- [Laravel Boost Best Practices PR](https://github.com/laravel/boost/pull/628)
