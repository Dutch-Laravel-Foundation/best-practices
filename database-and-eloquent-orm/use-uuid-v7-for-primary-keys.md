# Use UUID v7 for Primary Keys

<a name="introduction"></a>
## Introduction

UUID v7 (Universally Unique Identifier version 7) is a time-ordered identifier format standardized in RFC 9562 (May 2024). Unlike traditional auto-incrementing integers or fully random UUIDs (v4), UUID v7 combines a 48-bit Unix timestamp (in milliseconds) with random bits, resulting in identifiers that are both globally unique and naturally time-sorted.

Since Laravel 12, the `HasUuids` trait generates UUID v7 by default, making it straightforward to adopt this approach in your applications.

<a name="why"></a>
## Why

- **Better database performance**: UUID v7's sequential nature means new records append near the end of B-tree indexes, reducing page splits and improving write performance by up to 50% compared to random UUIDs
- **Security through obscurity**: IDs cannot be guessed or enumerated, preventing attackers from discovering valid resource identifiers
- **Pre-generation capability**: IDs can be generated before database insertion, useful for queuing jobs or preparing related records
- **Distributed system friendly**: No coordination needed between multiple application instances or database nodes—each can generate unique IDs independently
- **Database cluster compatibility**: Works reliably with clustered databases (like Galera/MariaDB) where auto-increment behavior can be unpredictable
- **Embedded timestamp**: Creation time can be extracted from the ID itself, providing audit trail capabilities without additional columns
- **Standardized format**: Unlike Laravel's legacy `orderedUuid()` (which used a custom v4 variant), UUID v7 follows RFC 9562 and is portable across frameworks and languages

<a name="suitable-for"></a>
## Suitable For

- Applications that expose IDs in URLs or APIs (prevents enumeration attacks)
- Distributed systems with multiple application servers or database replicas
- Microservices architectures where different services need to generate IDs independently
- Event sourcing or CQRS patterns where IDs must be known before persistence
- Applications using database clusters where auto-increment consistency cannot be guaranteed
- Projects that need to merge data from multiple sources without ID conflicts
- Medium to large scale applications where index performance matters

<a name="less-suitable"></a>
## Less Suitable

- Small, single-server applications where simplicity is paramount
- Systems with extreme storage constraints (UUIDs use 16 bytes vs 4 bytes for integers)
- Legacy systems tightly coupled to integer IDs that would require extensive refactoring
- Scenarios requiring sub-millisecond ordering precision (UUID v7 has millisecond granularity)
- Debug-heavy environments where human-readable sequential IDs aid troubleshooting

<a name="implementation"></a>
## Implementation

<a name="migration"></a>
### Migration

Replace the standard `id()` method with `uuid()` and mark it as primary:

```php
Schema::create('posts', function (Blueprint $table) {
    $table->uuid('id')->primary();
    $table->string('title');
    $table->text('content');
    $table->timestamps();
});
```

<a name="model"></a>
### Model

Add the `HasUuids` trait to your model. In Laravel 12+, this trait generates UUID v7 by default:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    use HasUuids;
}
```

If you need to specify which columns should receive UUIDs (for models with multiple UUID columns), override the `uniqueIds` method:

```php
public function uniqueIds(): array
{
    return ['id', 'external_id'];
}
```

<a name="foreign-keys"></a>
### Foreign Keys

Use `foreignUuid()` instead of `foreignId()` for relationships:

```php
Schema::create('comments', function (Blueprint $table) {
    $table->uuid('id')->primary();
    $table->foreignUuid('post_id')->constrained()->cascadeOnDelete();
    $table->text('body');
    $table->timestamps();
});
```

<a name="uuid-v7-vs-ulid"></a>
## UUID v7 vs ULID

Both UUID v7 and ULID provide time-ordered unique identifiers. Choose based on your specific needs:

| Aspect | UUID v7 | ULID |
|--------|---------|------|
| Format | 36 characters (with hyphens) | 26 characters |
| Standard | RFC 9562 | No formal RFC |
| Time precision | Milliseconds | Milliseconds |
| Human readability | Standard UUID format | Shorter, URL-friendly |
| Laravel trait | `HasUuids` | `HasUlids` |
| Cross-platform | Universal support | Less common |
| Storage | 16 bytes (binary) | 16 bytes (binary) |

For most Laravel applications, **UUID v7 is recommended** due to its RFC standardization and universal compatibility across frameworks and languages.

<a name="considerations"></a>
## Considerations

- **Storage overhead**: UUIDs require 16 bytes compared to 4 bytes for integers. For tables with millions of rows and multiple foreign keys, this can add up
- **Index size**: Larger keys mean larger indexes, which may affect memory usage and query performance on very large tables
- **Collision during seeding**: When rapidly generating records within the same millisecond (e.g., during database seeding), collisions are statistically possible at extreme scale. Use unique constraints to catch these rare cases
- **Debugging**: UUID values are harder to communicate verbally or remember during debugging compared to simple integers like `42`

<a name="more-info"></a>
## More Info

- [Laravel Documentation: UUIDs and ULIDs](https://laravel.com/docs/eloquent#uuid-and-ulid-keys)
- [RFC 9562: UUID v7 Specification](https://datatracker.ietf.org/doc/html/rfc9562)
- [Laravel 12 Upgrade Guide](https://laravel.com/docs/12.x/upgrade)
- [PostgreSQL UUID Performance: Benchmarking v4 and v7](https://dev.to/umangsinha12/postgresql-uuid-performance-benchmarking-random-v4-and-time-based-v7-uuids-n9b)
- [Goodbye to Sequential Integers, Hello UUIDv7 (Buildkite)](https://buildkite.com/resources/blog/goodbye-integers-hello-uuids/)
- [UUIDv7 Guide: Performance Benefits & Database Optimization](https://www.bindbee.dev/blog/why-bindbee-chose-uuidv7)
