# Use the HTTP Client Correctly

<a name="introduction"></a>
## Introduction

Laravel's HTTP Client (built on Guzzle) provides a fluent interface for making HTTP requests to external APIs. Using it correctly means setting explicit timeouts, implementing retry with backoff, handling errors properly, using request pooling for concurrent calls, and faking requests in tests.

<a name="why"></a>
## Why

- **Fail fast**: Explicit timeouts prevent requests from hanging for 30+ seconds on unresponsive APIs
- **Resilience**: Retry with exponential backoff handles transient failures gracefully without overwhelming external services
- **Correctness**: The HTTP Client does not throw on 4xx/5xx by default — errors must be handled explicitly to avoid silently using error response bodies as data
- **Performance**: `Http::pool()` runs independent requests concurrently, eliminating sequential wait times
- **Test reliability**: `Http::fake()` with `preventStrayRequests()` ensures tests never hit real APIs and catches unmocked calls

<a name="suitable-for"></a>
## Suitable For

- Any application that communicates with external APIs
- Services that integrate with payment providers, email services, or third-party data sources
- Applications where API reliability and performance matter

<a name="less-suitable"></a>
## Less Suitable

- Internal service-to-service communication where a dedicated client library is provided
- Simple file downloads or one-off scripts where robustness isn't critical

<a name="examples"></a>
## Examples

### Always Set Explicit Timeouts

```php
// Bad: default 30-second timeout
$response = Http::get('https://api.example.com/users');

// Good: explicit timeouts
$response = Http::timeout(5)
    ->connectTimeout(3)
    ->get('https://api.example.com/users');
```

For service-specific clients, define timeouts in a macro:

```php
Http::macro('github', function () {
    return Http::baseUrl('https://api.github.com')
        ->timeout(10)
        ->connectTimeout(3)
        ->withToken(config('services.github.token'));
});

$response = Http::github()->get('/repos/laravel/framework');
```

### Use Retry with Backoff for External APIs

```php
// Bad: no retry on transient failure
$response = Http::post('https://api.stripe.com/v1/charges', $data);

// Good: exponential backoff
$response = Http::retry([100, 500, 1000])
    ->timeout(10)
    ->post('https://api.stripe.com/v1/charges', $data);
```

### Handle Errors Explicitly

```php
// Bad: could be using an error response body as data
$response = Http::get('https://api.example.com/users/1');
$user = $response->json();

// Good: throw on failure
$response = Http::timeout(5)
    ->get('https://api.example.com/users/1')
    ->throw();

$user = $response->json();

// Good: graceful degradation
$response = Http::get('https://api.example.com/users/1');

if ($response->successful()) {
    return $response->json();
}

if ($response->notFound()) {
    return null;
}

$response->throw();
```

### Use Request Pooling for Concurrent Requests

```php
// Bad: sequential requests
$users = Http::get('https://api.example.com/users')->json();
$posts = Http::get('https://api.example.com/posts')->json();

// Good: concurrent requests
use Illuminate\Http\Client\Pool;

$responses = Http::pool(fn (Pool $pool) => [
    $pool->as('users')->get('https://api.example.com/users'),
    $pool->as('posts')->get('https://api.example.com/posts'),
]);

$users = $responses['users']->json();
$posts = $responses['posts']->json();
```

### Fake HTTP Calls in Tests

```php
it('syncs user from API', function () {
    Http::preventStrayRequests();

    Http::fake([
        'api.example.com/users/1' => Http::response([
            'name' => 'John Doe',
            'email' => 'john@example.com',
        ]),
    ]);

    $service = new UserSyncService;
    $service->sync(1);

    Http::assertSent(function (Request $request) {
        return $request->url() === 'https://api.example.com/users/1';
    });
});
```

<a name="more-info"></a>
## More Info

- [Laravel HTTP Client Documentation](https://laravel.com/docs/http-client)
- [Laravel HTTP Client Testing Documentation](https://laravel.com/docs/http-client#testing)
- [Follow Testing Best Practices](../testing/follow-testing-best-practices.md) — for general testing patterns
- [Laravel Boost Best Practices PR](https://github.com/laravel/boost/pull/628)
