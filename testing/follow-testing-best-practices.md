# Follow Testing Best Practices

<a name="introduction"></a>
## Introduction

Beyond choosing a testing framework (see [Use PHPUnit or Pest](./use-phpunit-or-pest-for-testing.md)), there are several Laravel-specific testing patterns that improve test speed, reliability, and expressiveness. These include using `LazilyRefreshDatabase` for performance, model assertions for clarity, factory states for readability, and correctly ordering fakes to avoid breaking model events.

<a name="why"></a>
## Why

- **Faster test suites**: `LazilyRefreshDatabase` only runs migrations when the schema changes, significantly speeding up large suites
- **Expressive assertions**: `assertModelExists()` is more type-safe and produces clearer failure messages than raw `assertDatabaseHas()`
- **Self-documenting tests**: Named factory states like `->unverified()` communicate intent better than manual attribute overrides
- **Correct behavior**: Calling `Event::fake()` after factory setup prevents silently breaking model events that factories depend on

<a name="suitable-for"></a>
## Suitable For

- All Laravel applications with test suites
- Teams looking to speed up CI/CD pipelines
- Projects with complex factory relationships
- Applications with event-driven or queued behavior that needs testing

<a name="less-suitable"></a>
## Less Suitable

- Very small test suites where speed optimizations aren't noticeable
- Projects not using Laravel's built-in testing features

<a name="examples"></a>
## Examples

### Use `LazilyRefreshDatabase`

```php
// Slower: runs all migrations every test run
use Illuminate\Foundation\Testing\RefreshDatabase;

// Faster: only migrates when schema changes
use Illuminate\Foundation\Testing\LazilyRefreshDatabase;
```

### Use Model Assertions

```php
// Bad: raw database assertion
$this->assertDatabaseHas('users', ['id' => $user->id]);

// Good: more expressive and type-safe
$this->assertModelExists($user);
```

### Use Factory States and Sequences

Named states make tests self-documenting:

```php
// Bad: manual attribute override
User::factory()->create(['email_verified_at' => null]);

// Good: named state communicates intent
User::factory()->unverified()->create();
```

### Call `Event::fake()` After Factory Setup

Model factories rely on model events (e.g., `creating` to generate UUIDs). Calling `Event::fake()` before factory calls silences those events, producing broken models:

```php
// Bad: Event::fake() prevents factory model events
Event::fake();
$user = User::factory()->create();

// Good: create models first, then fake events
$user = User::factory()->create();
Event::fake();
```

### Use `Exceptions::fake()` for Exception Assertions

Instead of `withoutExceptionHandling()`, use `Exceptions::fake()` to assert the correct exception was reported while the request completes normally.

### Use `recycle()` to Share Relationship Instances

Without `recycle()`, nested factories create separate instances of the same conceptual entity:

```php
Ticket::factory()
    ->recycle(Airline::factory()->create())
    ->create();
```

<a name="more-info"></a>
## More Info

- [Laravel Testing Documentation](https://laravel.com/docs/testing)
- [Laravel Database Testing Documentation](https://laravel.com/docs/database-testing)
- [Laravel Mocking Documentation](https://laravel.com/docs/mocking)
- [Use PHPUnit or Pest for Testing](./use-phpunit-or-pest-for-testing.md)
- [Stick to One Testing Framework](./stick-to-one-testing-framework.md)
- [Laravel Boost Best Practices PR](https://github.com/laravel/boost/pull/628)
