# Use Configuration Properly

<a name="introduction"></a>
## Introduction

Laravel's configuration system is designed with a clear rule: environment variables should only be accessed in config files, and application code should always use `config()`. This pattern ensures that configuration caching works correctly and that environment checks are reliable. Additionally, using constants and language files instead of hardcoded strings improves maintainability.

<a name="why"></a>
## Why

- **Caching compatibility**: Direct `env()` calls return `null` when config is cached — using `config()` always works correctly
- **Centralized settings**: All configuration lives in config files, making it easy to find and audit what the application uses
- **Reliable environment checks**: `App::environment()` and `app()->isProduction()` work regardless of config caching, unlike `env('APP_ENV')`
- **Refactoring safety**: Using class constants instead of magic strings for model states and types makes IDE refactoring possible and eliminates typo-related bugs

<a name="suitable-for"></a>
## Suitable For

- All Laravel applications
- Applications deployed with `config:cache` (i.e., most production environments)
- Projects with multiple environments (local, staging, production)

<a name="less-suitable"></a>
## Less Suitable

- N/A — these practices apply to every Laravel application

<a name="examples"></a>
## Examples

### `env()` Only in Config Files

```php
// Bad: returns null when config is cached
$key = env('API_KEY');

// Good: define in config, use via config()
// config/services.php
'key' => env('API_KEY'),

// Application code
$key = config('services.key');
```

### Use `App::environment()` for Environment Checks

```php
// Bad: breaks with config caching
if (env('APP_ENV') === 'production') {

// Good: always reliable
if (app()->isProduction()) {
// or
if (App::environment('production')) {
```

### Use Constants Instead of Magic Strings

```php
// Bad: typo-prone magic string
return $this->type === 'normal';

// Good: refactorable constant
return $this->type === self::TYPE_NORMAL;
```

### Use Language Files When Already Present

If the application already uses language files for localization, use `__()` for user-facing strings too. Do not introduce language files purely for English-only apps — simple string literals are fine there:

```php
// Only when lang files already exist in the project
return back()->with('message', __('app.article_added'));
```

### Use Encrypted Env for Production Secrets

Never store production secrets in plain `.env` files in version control:

```bash
php artisan env:encrypt --env=production --readable
php artisan env:decrypt --env=production
```

For cloud deployments, prefer the platform's native secret store (AWS Secrets Manager, Vault, etc.) and inject at runtime.

<a name="more-info"></a>
## More Info

- [Laravel Configuration Documentation](https://laravel.com/docs/configuration)
- [Laravel Environment Configuration](https://laravel.com/docs/configuration#environment-configuration)
- [Laravel Encryption Documentation](https://laravel.com/docs/encryption)
- [Prevent Common Vulnerabilities](../security-and-authentication/prevent-common-vulnerabilities.md) — for keeping secrets secure and encrypting sensitive fields
- [Laravel Boost Best Practices PR](https://github.com/laravel/boost/pull/628)
