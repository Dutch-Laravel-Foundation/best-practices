# Configure Queued Jobs Properly

<a name="introduction"></a>
## Introduction

Beyond deferring business logic to jobs (see [Keep Commands Small and Defer to Jobs](./keep-commands-small-defer-to-jobs.md)), jobs themselves need proper configuration to be reliable in production. This includes setting correct timeout and retry values, implementing exponential backoff, preventing duplicate execution, handling failures explicitly, and rate limiting external API calls.

<a name="why"></a>
## Why

- **Prevents duplicate execution**: When `retry_after` is shorter than `timeout`, the queue worker re-dispatches the job while it's still running
- **Protects external services**: Exponential backoff and rate limiting prevent hammering failing APIs
- **Explicit failure handling**: Implementing `failed()` ensures errors are handled rather than silently ignored
- **Controlled concurrency**: `ShouldBeUnique` and `WithoutOverlapping` prevent duplicate and concurrent processing of the same data

<a name="suitable-for"></a>
## Suitable For

- Applications using queued jobs in production
- Jobs that call external APIs or process critical data
- Multi-worker or multi-server queue deployments
- Jobs processing user-facing operations where duplicates or failures are visible

<a name="less-suitable"></a>
## Less Suitable

- Jobs using the `sync` queue driver (development/testing only)
- Simple fire-and-forget jobs where failures are acceptable

<a name="examples"></a>
## Examples

### Set `retry_after` Greater Than `timeout`

```php
class ProcessReport implements ShouldQueue
{
    public $timeout = 120;
}

// config/queue.php — retry_after must be longer than any job timeout
// retry_after: 180 ← safely longer
```

### Use Exponential Backoff

```php
class SyncWithStripe implements ShouldQueue
{
    public $tries = 3;
    public $backoff = [1, 5, 10]; // seconds between retries
}
```

### Prevent Duplicate Job Processing

```php
class GenerateInvoice implements ShouldQueue, ShouldBeUnique
{
    public function uniqueId(): string
    {
        return $this->order->id;
    }

    public $uniqueFor = 3600;
}
```

### Always Implement `failed()`

```php
public function failed(?Throwable $exception): void
{
    $this->podcast->update(['status' => 'failed']);
    Log::error('Processing failed', [
        'id' => $this->podcast->id,
        'error' => $exception->getMessage(),
    ]);
}
```

### Rate Limit External API Calls

```php
public function middleware(): array
{
    return [new RateLimited('external-api')];
}
```

### Batch Related Jobs

```php
Bus::batch([
    new ImportCsvChunk($chunk1),
    new ImportCsvChunk($chunk2),
])
->then(fn (Batch $batch) => Notification::send($user, new ImportComplete))
->catch(fn (Batch $batch, Throwable $e) => Log::error('Batch failed'))
->dispatch();
```

### Use `WithoutOverlapping` for Concurrency Control

```php
public function middleware(): array
{
    return [new WithoutOverlapping($this->product->id)->untilProcessing()];
}
```

Without `untilProcessing()`, the lock extends through queue wait time. With it, the lock releases when processing starts, allowing new instances to queue.

<a name="more-info"></a>
## More Info

- [Laravel Queue Documentation](https://laravel.com/docs/queues)
- [Laravel Job Middleware Documentation](https://laravel.com/docs/queues#job-middleware)
- [Laravel Horizon Documentation](https://laravel.com/docs/horizon)
- [Keep Commands Small and Defer to Jobs](./keep-commands-small-defer-to-jobs.md)
- [Laravel Boost Best Practices PR](https://github.com/laravel/boost/pull/628)
