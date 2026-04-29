# Write Effective Migrations

<a name="introduction"></a>
## Introduction

Migrations are the version control for your database schema. Well-written migrations are focused, reversible, and include proper indexing from the start. Since migrations are frozen snapshots in time, they require special discipline — once deployed to production, they should never be modified.

<a name="why"></a>
## Why

- **Consistency**: Using `constrained()` for foreign keys ensures automatic naming and referential integrity
- **Safety**: Never modifying deployed migrations prevents inconsistent database states across environments
- **Performance**: Adding indexes in the migration rather than as an afterthought avoids forgotten performance optimizations
- **Reversibility**: Writing `down()` methods allows safe rollbacks during failed deployments and in CI pipelines
- **Clarity**: Keeping one concern per migration makes it easy to identify what changed and when

<a name="suitable-for"></a>
## Suitable For

- All Laravel applications using migrations
- Teams with multiple developers working on the same database schema
- Projects with CI/CD pipelines that run migrations

<a name="less-suitable"></a>
## Less Suitable

- N/A — these practices apply to any project using Laravel migrations

<a name="examples"></a>
## Examples

### Use `constrained()` for Foreign Keys

```php
$table->foreignId('user_id')->constrained()->cascadeOnDelete();

// Non-standard names
$table->foreignId('author_id')->constrained('users');
```

### Never Modify Deployed Migrations

```php
// Bad: editing a migration that already ran in production
// 2024_01_01_create_posts_table.php
$table->string('slug')->unique(); // added after deployment

// Good: new migration to alter the table
// 2024_03_15_add_slug_to_posts_table.php
Schema::table('posts', function (Blueprint $table) {
    $table->string('slug')->unique()->after('title');
});
```

### Add Indexes in the Migration

```php
// Bad: no indexes on frequently queried columns
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained();
    $table->string('status');
    $table->timestamps();
});

// Good: indexes added from the start
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->index();
    $table->string('status')->index();
    $table->timestamp('shipped_at')->nullable()->index();
    $table->timestamps();
});
```

### Mirror Column Defaults in Model `$attributes`

When a column has a database default, mirror it in the model so new instances have correct values before saving:

```php
// Migration
$table->string('status')->default('pending');

// Model
protected $attributes = [
    'status' => 'pending',
];
```

### Write Reversible `down()` Methods

```php
public function down(): void
{
    Schema::table('posts', function (Blueprint $table) {
        $table->dropColumn('slug');
    });
}
```

For intentionally irreversible migrations, leave a clear comment and require a forward-fix migration instead.

### Keep Migrations Focused

Never mix DDL (schema changes) and DML (data manipulation) in a single migration:

```php
// Bad: partial failure creates unrecoverable state
public function up(): void
{
    Schema::create('settings', function (Blueprint $table) { /* ... */ });
    DB::table('settings')->insert(['key' => 'version', 'value' => '1.0']);
}

// Good: separate migrations
// Migration 1: create_settings_table
Schema::create('settings', function (Blueprint $table) { /* ... */ });

// Migration 2: seed_default_settings
DB::table('settings')->insert(['key' => 'version', 'value' => '1.0']);
```

<a name="more-info"></a>
## More Info

- [Laravel Migrations Documentation](https://laravel.com/docs/migrations)
- [Avoid Eloquent Models in Migrations](./avoid-eloquent-models-in-migrations.md) — use Query Builder instead of Eloquent in migrations
- [Laravel Boost Best Practices PR](https://github.com/laravel/boost/pull/628)
