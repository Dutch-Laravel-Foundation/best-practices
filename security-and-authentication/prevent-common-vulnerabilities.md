# Prevent Common Vulnerabilities

<a name="introduction"></a>
## Introduction

Laravel provides built-in protections against the most common web application vulnerabilities, but they need to be used correctly. This covers essential security practices including mass assignment protection, SQL injection prevention, XSS escaping, CSRF protection, file upload validation, rate limiting, and encrypting sensitive database fields. For authorization patterns, see [Use Policies and Gates for Authorization](../project-structure-and-code-architecture/use-policies-and-gates-for-authorization.md).

<a name="why"></a>
## Why

- **Defense in depth**: Each practice addresses a different attack vector — together they cover the OWASP Top 10 risks relevant to Laravel applications
- **Framework support**: Laravel already provides all the tools; you just need to use them consistently
- **Data protection**: Encrypting sensitive fields and keeping secrets out of code protects against data breaches
- **Availability**: Rate limiting prevents brute-force attacks and abuse of authentication and API endpoints

<a name="suitable-for"></a>
## Suitable For

- All Laravel applications, regardless of size
- Applications handling user input, authentication, or file uploads
- Applications storing sensitive data (API keys, tokens, personal information)

<a name="less-suitable"></a>
## Less Suitable

- N/A — these practices apply to every Laravel application

<a name="examples"></a>
## Examples

### Mass Assignment Protection

Every model must define `$fillable` (whitelist) or `$guarded` (blacklist):

```php
// Bad: all fields are mass assignable
class User extends Model
{
    protected $guarded = [];
}

// Good: explicit whitelist
class User extends Model
{
    protected $fillable = [
        'name',
        'email',
        'password',
    ];
}
```

Never use `$guarded = []` on models that accept user input.

### Prevent SQL Injection

Always use parameter binding. Never interpolate user input into queries:

```php
// Bad: SQL injection vulnerability
DB::select("SELECT * FROM users WHERE name = '{$request->name}'");

// Good: parameter binding
User::where('name', $request->name)->get();

// Good: raw expressions with bindings
User::whereRaw('LOWER(name) = ?', [strtolower($request->name)])->get();
```

### Escape Output to Prevent XSS

Use `{{ }}` for HTML escaping. Only use `{!! !!}` for trusted, pre-sanitized content:

```blade
{{-- Bad: unescaped user content --}}
{!! $user->bio !!}

{{-- Good: auto-escaped --}}
{{ $user->bio }}
```

### CSRF Protection

Include `@csrf` in all POST/PUT/DELETE Blade forms:

```blade
<form method="POST" action="/posts">
    @csrf
    <input type="text" name="title">
</form>
```

### Rate Limit Auth and API Routes

```php
RateLimiter::for('login', function (Request $request) {
    return Limit::perMinute(5)->by($request->ip());
});

Route::post('/login', LoginController::class)->middleware('throttle:login');
```

### Validate File Uploads

Validate MIME type, extension, and size. Never trust client-provided filenames:

```php
public function rules(): array
{
    return [
        'avatar' => ['required', 'image', 'mimes:jpg,jpeg,png,webp', 'max:2048'],
    ];
}
```

Store with generated filenames:

```php
$path = $request->file('avatar')->store('avatars', 'public');
```

### Encrypt Sensitive Database Fields

Use `encrypted` cast for API keys and tokens, and mark the attribute as `hidden`:

```php
class Integration extends Model
{
    protected $hidden = ['api_key', 'api_secret'];

    protected function casts(): array
    {
        return [
            'api_key' => 'encrypted',
            'api_secret' => 'encrypted',
        ];
    }
}
```

### Audit Dependencies

Run `composer audit` periodically and automate it in CI:

```bash
composer audit
```

<a name="more-info"></a>
## More Info

- [Laravel Security Documentation](https://laravel.com/docs/security)
- [Laravel CSRF Protection](https://laravel.com/docs/csrf)
- [Laravel Rate Limiting](https://laravel.com/docs/rate-limiting)
- [Laravel Encryption Documentation](https://laravel.com/docs/encryption)
- [Use Policies and Gates for Authorization](../project-structure-and-code-architecture/use-policies-and-gates-for-authorization.md) — for authorization patterns
- [Implement Content Security Policy (CSP)](./implement-content-security-policy.md) — for additional XSS protection via security headers
- [Use Configuration Properly](../project-structure-and-code-architecture/use-configuration-properly.md) — for keeping secrets out of code
- [Laravel Boost Best Practices PR](https://github.com/laravel/boost/pull/628)
