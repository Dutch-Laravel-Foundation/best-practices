# Use PHPUnit or Pest for Testing

<a name="introduction"></a>
## Introduction

Laravel provides first-class support for both PHPUnit and Pest as testing frameworks. PHPUnit is the traditional, class-based testing framework that has been the standard in PHP for many years, while Pest offers a modern, function-based approach with a simpler syntax. Both frameworks are officially supported and integrate seamlessly with Laravel's testing features.

<a name="why"></a>
## Why

- **Official support**: Both frameworks are officially maintained and documented by Laravel
- **Consistent tooling**: Laravel's testing helpers work identically with both frameworks
- **Community standards**: Using these frameworks ensures compatibility with community packages and best practices
- **Built-in assertions**: Both provide comprehensive assertion libraries specifically designed for Laravel applications
- **Easy setup**: Laravel comes pre-configured with PHPUnit, and Pest can be added with minimal configuration
- **Continuous improvement**: Both frameworks receive regular updates and improvements alongside Laravel releases

<a name="suitable-for"></a>
## Suitable For

- All Laravel applications that require automated testing
- Projects where team members prefer traditional xUnit-style testing (PHPUnit)
- Projects where team members prefer modern, function-based testing syntax (Pest)
- Applications that need to integrate with Laravel's testing helpers (database factories, HTTP testing, etc.)
- Teams transitioning from other PHP frameworks that use PHPUnit
- New projects looking for a clean, expressive testing syntax (Pest)

<a name="less-suitable"></a>
## Less Suitable

- Applications with existing test suites in other PHP testing frameworks (though migration is possible)
- Projects with specific requirements for alternative testing tools not compatible with Laravel's testing architecture
- Teams with strong preferences for testing frameworks outside the PHP ecosystem (though this would typically indicate a different language choice altogether)

<a name="examples"></a>
## Examples

### PHPUnit Example

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

class UserRegistrationTest extends TestCase
{
    use RefreshDatabase;

    public function test_users_can_register(): void
    {
        $response = $this->post('/register', [
            'name' => 'John Doe',
            'email' => 'john@example.com',
            'password' => 'password',
            'password_confirmation' => 'password',
        ]);

        $response->assertRedirect('/dashboard');
        $this->assertDatabaseHas('users', [
            'email' => 'john@example.com',
        ]);
    }
}
```

### Pest Example

```php
<?php

use App\Models\User;

test('users can register', function () {
    $response = $this->post('/register', [
        'name' => 'John Doe',
        'email' => 'john@example.com',
        'password' => 'password',
        'password_confirmation' => 'password',
    ]);

    $response->assertRedirect('/dashboard');
    expect(User::where('email', 'john@example.com')->exists())->toBeTrue();
});
```

<a name="choosing-between-phpunit-and-pest"></a>
## Choosing Between PHPUnit and Pest

**Choose PHPUnit if:**
- Your team is already familiar with traditional xUnit-style testing
- You're maintaining an existing Laravel project that uses PHPUnit
- You prefer class-based, object-oriented testing structure
- You need compatibility with older Laravel versions (pre-Laravel 8)

**Choose Pest if:**
- You're starting a new project and want a modern testing syntax
- Your team prefers functional programming styles
- You want less boilerplate code in your tests
- You're using Laravel 8 or newer

**Note:** Both frameworks can coexist in the same project, allowing for gradual migration if desired.

<a name="more-info"></a>
## More Info

- [Laravel Testing Documentation](https://laravel.com/docs/testing)
- [PHPUnit Documentation](https://phpunit.de/documentation.html)
- [Pest PHP Official Website](https://pestphp.com/)
- [Laravel Testing with Pest](https://laravel.com/docs/testing#introduction-to-pest)
- [PHPUnit Assertions](https://docs.phpunit.de/en/11.0/assertions.html)
- [Pest Expectations](https://pestphp.com/docs/expectations)
