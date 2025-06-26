# Keep Commands Small and Defer Business Logic to Jobs

<a name="introduction"></a>
## Introduction

Laravel console commands should be kept lightweight and focused on handling command-line input/output, while complex business logic should be deferred to queued jobs. This architectural pattern separates concerns between the command layer (responsible for CLI interaction) and the business logic layer (handled by jobs), resulting in more maintainable and scalable applications.

<a name="why"></a>
## Why

- **Better Performance**: Commands that execute heavy business logic synchronously can cause timeouts or block other processes. Jobs allow asynchronous processing, drastically speeding up command execution.

- **Improved Reusability**: When business logic is encapsulated in jobs rather than commands, it becomes easy for controllers, other jobs, and commands to use the same logic.

- **Scalability**: Jobs can be distributed across multiple workers and queues, allowing better resource utilization and the ability to scale processing based on demand.

- **Enhanced Error Handling**: Jobs provide robust error handling mechanisms including automatic retry capabilities with configurable backoff strategies, failed job handling, and comprehensive monitoring.

- **Scheduler Efficiency**: When multiple scheduled commands run simultaneously, lightweight commands that quickly dispatch jobs prevent blocking the scheduler, ensuring all scheduled tasks run on time.

- **Better Testing**: Business logic in jobs can be tested independently from command-line concerns, leading to more focused and maintainable tests.

<a name="suitable-for"></a>
## Suitable For

- **Scheduled Commands**: Tasks that run on a schedule and perform data processing, imports, exports, or other time-consuming operations
- **Long-Running Operations**: Commands that process large datasets, make multiple API calls, or perform complex calculations
- **Operations Requiring Retry Logic**: Tasks that might fail due to external dependencies (APIs, third-party services)
- **Batch Processing**: Commands that need to process multiple items where each item could be processed independently
- **Resource-Intensive Tasks**: Operations that consume significant memory or CPU resources

<a name="less-suitable"></a>
## Less Suitable

- **Interactive Commands**: Commands that require user input or real-time feedback during execution
- **Simple Database Queries**: Quick operations like cache clearing or simple database cleanups that complete in seconds
- **Development/Debugging Commands**: One-off commands used for debugging or development purposes
- **Commands Requiring Immediate Results**: Operations where the command must wait for and display the result immediately

<a name="implementation"></a>
## Implementation

### Basic Pattern

**Command (Small and Focused):**
```php
namespace App\Console\Commands;

use App\Jobs\ProcessDataJob;
use Illuminate\Console\Command;

class ProcessDataCommand extends Command
{
    protected $signature = 'data:process {type}';
    protected $description = 'Queue data processing job';

    public function handle()
    {
        $type = $this->argument('type');
        
        // Minimal logic - just dispatch the job
        ProcessDataJob::dispatch($type);
        
        $this->info("Data processing job queued for type: {$type}");
        
        return 0;
    }
}
```

**Job (Contains Business Logic):**
```php
namespace App\Jobs;

use App\Services\DataProcessor;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessDataJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public $tries = 3;
    public $timeout = 120;

    public function __construct(
        private string $type
    ) {}

    public function handle(DataProcessor $processor)
    {
        // All the heavy business logic goes here
        $processor->processType($this->type);
    }

    public function failed(\Throwable $exception)
    {
        // Handle job failure
        \Log::error('Data processing failed', [
            'type' => $this->type,
            'error' => $exception->getMessage()
        ]);
    }
}
```

### Scheduled Commands with Jobs

```php
// In app/Console/Kernel.php
protected function schedule(Schedule $schedule)
{
    // Good: Command quickly dispatches job
    $schedule->command('data:process daily')
        ->daily()
        ->withoutOverlapping();
    
    // Better: Direct job scheduling for simple cases
    $schedule->job(new ProcessDataJob('hourly'))
        ->hourly();
}
```

### Preventing Duplicate Jobs

For scheduled commands, use unique jobs to prevent overlap:

```php
use Illuminate\Contracts\Queue\ShouldBeUnique;

class ScheduledImportJob implements ShouldQueue, ShouldBeUnique
{
    public function uniqueId(): string
    {
        return 'scheduled-import-' . $this->importType;
    }
    
    public function uniqueFor(): int
    {
        return 3600; // 1 hour
    }
}
```

<a name="common-pitfalls"></a>
## Common Pitfalls

### Using Sync Queue Driver
The most important pitfall is using the default `sync` driver, which executes jobs synchronously and defeats the entire purpose of this pattern:

```php
// BAD: Job blocks the command with sync driver
QUEUE_CONNECTION=sync

// GOOD: Job runs asynchronously
QUEUE_CONNECTION=redis  # or database, sqs, beanstalkd
```

When using the `sync` driver:
- Jobs execute immediately in the same process as the command
- Commands will block until the job completes
- No retry capabilities are available
- Failed jobs aren't tracked
- You lose all benefits of deferring to jobs

Always ensure your queue connection is configured for asynchronous processing when implementing this pattern.

<a name="more-info"></a>
## More Info

- [Laravel Queue Documentation](https://laravel.com/docs/queues)
- [Laravel Task Scheduling Documentation](https://laravel.com/docs/scheduling)
- [Laravel Console Commands Documentation](https://laravel.com/docs/artisan)
- [Supervisor Configuration for Laravel](https://laravel.com/docs/queues#supervisor-configuration)
- [Laravel Horizon for Queue Monitoring](https://laravel.com/docs/horizon)
- [Laravel Queue Workers Documentation](https://laravel.com/docs/queues#running-the-queue-worker)