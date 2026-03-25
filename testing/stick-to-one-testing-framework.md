# Stick to One Testing Framework

<a name="introduction"></a>
## Introduction

While Pest is built on top of PHPUnit and the two frameworks are technically compatible, mixing both testing styles in the same Laravel project introduces unnecessary complexity and confusion. A consistent testing approach using either PHPUnit or Pest throughout your entire test suite is strongly recommended.

<a name="why"></a>
## Why

- **Reduced cognitive load**: Team members only need to learn and remember one testing syntax and approach
- **Easier onboarding**: New developers joining the project face a simpler learning curve with a single, consistent testing style
- **Command-line consistency**: Avoid confusion about which test runner to use (`vendor/bin/phpunit` vs `vendor/bin/pest`)
- **Better maintainability**: Consistent patterns make it easier to update and refactor tests across the entire codebase
- **Avoid compatibility issues**: Some PHPUnit annotations (like `@runTestsInSeparateProcesses`) are not compatible with Pest
- **Clearer project standards**: A single testing framework establishes clear expectations for all contributors
- **Simplified CI/CD pipelines**: No need to handle multiple test runners or special configurations

<a name="suitable-for"></a>
## Suitable For

- All Laravel projects, regardless of size
- Teams with multiple developers
- Projects with long-term maintenance requirements
- Codebases where consistency and readability are priorities
- Open-source projects where external contributors need clear guidelines

<a name="less-suitable"></a>
## Less Suitable

- Projects in active migration from PHPUnit to Pest (temporary mixed state is acceptable during transition)
- Monorepos where different packages have legitimate reasons for using different frameworks (though even here, consistency is generally better)

<a name="handling-existing-mixed-test-suites"></a>
## Handling Existing Mixed Test Suites

If you inherit or currently have a project with mixed PHPUnit and Pest tests, consider migrating to a single framework:

### Migration to Pest

Pest provides several tools to help convert PHPUnit tests:

1. **Drift Plugin** (Automatic):
```bash
composer require pestphp/pest-plugin-drift --dev
./vendor/bin/pest --drift
```

2. **Laravel Shift** (Semi-Automatic):
   - Use the [Pest Converter](https://laravelshift.com/phpunit-to-pest-converter) service
   - Estimated time savings: ~3 hours for typical projects
   - Handles most conversions automatically

3. **Manual Migration**:
   - Convert tests gradually as you work on related features
   - Use Pest's compatibility layer during the transition period
   - Ensure all new tests use the target framework

### Migration to PHPUnit

If your team prefers PHPUnit:
- Pest tests can be manually rewritten as PHPUnit test classes
- This is typically more labor-intensive than migrating to Pest
- Consider this route if the team strongly prefers class-based testing

<a name="what-gets-converted"></a>
## What Gets Converted (PHPUnit to Pest)

When using automated conversion tools:
- ✅ Lifecycle methods (`setUp`, `tearDown`) → Pest hooks (`beforeEach`, `afterEach`)
- ✅ Test methods → Pest `test()` or `it()` functions
- ✅ Data providers → Pest datasets
- ✅ Test groups → Pest `group()` chaining
- ✅ PHPUnit assertions → Pest expectations (where available)

**Important**: Private helper methods in test classes become functions, which may require manual fixes since they lose access to `$this`.

<a name="exceptions"></a>
## Exceptions

The only scenario where mixing frameworks is acceptable:

**During Active Migration**: A temporary period where you're converting tests from one framework to another is acceptable, but should be:
- Time-boxed (set a deadline for completion)
- Clearly communicated to the team
- Documented in the project README or contributing guidelines
- Prioritized to minimize the mixed state duration

<a name="examples"></a>
## Examples

### ❌ Bad: Mixed Test Suite

```
tests/
├── Feature/
│   ├── UserRegistrationTest.php    # PHPUnit class
│   ├── checkout_test.php            # Pest functional test
│   └── ProfileTest.php              # PHPUnit class
└── Unit/
    ├── calculate_discount_test.php  # Pest functional test
    └── OrderTest.php                # PHPUnit class
```

### ✅ Good: Consistent PHPUnit Test Suite

```
tests/
├── Feature/
│   ├── UserRegistrationTest.php
│   ├── CheckoutTest.php
│   └── ProfileTest.php
└── Unit/
    ├── DiscountCalculatorTest.php
    └── OrderTest.php
```

### ✅ Good: Consistent Pest Test Suite

```
tests/
├── Feature/
│   ├── UserRegistrationTest.php
│   ├── CheckoutTest.php
│   └── ProfileTest.php
└── Unit/
    ├── DiscountCalculatorTest.php
    └── OrderTest.php
```

<a name="project-documentation"></a>
## Project Documentation

Document your chosen testing framework in your project:

**In README.md or CONTRIBUTING.md:**

```markdown
## Testing

This project uses Pest for all tests. When writing new tests:

- Use `test()` or `it()` functions, not PHPUnit classes
- Run tests with `./vendor/bin/pest` or `php artisan test`
- Follow existing test patterns in the `tests/` directory

See [Pest documentation](https://pestphp.com/docs) for syntax reference.
```

Or for PHPUnit:

```markdown
## Testing

This project uses PHPUnit for all tests. When writing new tests:

- Extend `Tests\TestCase` for feature tests
- Extend `PHPUnit\Framework\TestCase` for unit tests
- Run tests with `./vendor/bin/phpunit` or `php artisan test`
- Follow PSR-4 naming conventions for test classes

See [PHPUnit documentation](https://docs.phpunit.de) for syntax reference.
```

<a name="more-info"></a>
## More Info

- [Migrating from PHPUnit to Pest](https://pestphp.com/docs/migrating-from-phpunit-guide)
- [Pest Converter by Laravel Shift](https://laravelshift.com/phpunit-to-pest-converter)
- [Converting a PHPUnit Testsuite to Pest - Spatie](https://spatie.be/courses/testing-laravel-with-pest/converting-a-phpunit-testsuite-to-pest)
- [Pest Drift Plugin Documentation](https://pestphp.com/docs/plugins#drift)
- [Laravel Testing Documentation](https://laravel.com/docs/testing)
