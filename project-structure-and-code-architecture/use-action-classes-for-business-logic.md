# Use Action Classes for Business Logic

<a name="introduction"></a>
## Introduction

Action classes are single-purpose, invokable classes that encapsulate one discrete business operation. Combined with constructor dependency injection and coding to interfaces at system boundaries, they keep controllers thin, business logic testable, and external dependencies swappable.

<a name="why"></a>
## Why

- **Single responsibility**: Each action class does one thing well, making it easy to find, test, and modify business logic
- **Reusability**: The same action can be called from controllers, commands, jobs, and other actions
- **Testability**: Constructor injection makes dependencies explicit and easy to mock
- **Swappability**: Coding to interfaces at system boundaries (payment gateways, notification channels, external APIs) allows swapping implementations without changing business logic

<a name="suitable-for"></a>
## Suitable For

- Business operations that are called from multiple places (controllers, commands, jobs)
- Operations with external dependencies that should be mockable in tests
- Complex operations that would make controllers or jobs too large
- Applications with multiple integration points (payment providers, shipping services, etc.)

<a name="less-suitable"></a>
## Less Suitable

- Simple CRUD operations that are only used in one place
- Operations where a single Eloquent call suffices
- Prototyping or throwaway code where the overhead isn't justified

<a name="examples"></a>
## Examples

### Single-Purpose Action Class

```php
class CreateOrderAction
{
    public function __construct(private InventoryService $inventory) {}

    public function execute(array $data): Order
    {
        $order = Order::create($data);
        $this->inventory->reserve($order);

        return $order;
    }
}
```

### Use Dependency Injection

Always use constructor injection. Avoid `app()` or `resolve()` inside classes:

```php
// Bad: service locator pattern
class OrderController extends Controller
{
    public function store(StoreOrderRequest $request)
    {
        $service = app(OrderService::class);

        return $service->create($request->validated());
    }
}

// Good: constructor injection
class OrderController extends Controller
{
    public function __construct(private OrderService $service) {}

    public function store(StoreOrderRequest $request)
    {
        return $this->service->create($request->validated());
    }
}
```

### Code to Interfaces at System Boundaries

Depend on contracts for external integrations to enable testability and swappability:

```php
// Bad: concrete dependency
class OrderService
{
    public function __construct(private StripeGateway $gateway) {}
}

// Good: interface dependency
interface PaymentGateway
{
    public function charge(int $amount, string $customerId): PaymentResult;
}

class OrderService
{
    public function __construct(private PaymentGateway $gateway) {}
}
```

Bind in a service provider:

```php
$this->app->bind(PaymentGateway::class, StripeGateway::class);
```

<a name="more-info"></a>
## More Info

- [Laravel Service Container Documentation](https://laravel.com/docs/container)
- [Laravel Service Providers Documentation](https://laravel.com/docs/providers)
- [Laravel Boost Best Practices PR](https://github.com/laravel/boost/pull/628)
