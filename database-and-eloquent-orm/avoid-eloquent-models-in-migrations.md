# Avoid Using Eloquent Models in Migrations

<a name="introduction"></a>
## Introduction

Database migrations should use raw database queries or Laravel's Query Builder (DB facade) instead of Eloquent models. While it may seem convenient to use models for data manipulation during migrations, this practice can lead to broken migrations and unpredictable behavior over time.

<a name="why"></a>
## Why

- **Models evolve, migrations don't** - When you change an Eloquent model (add fields, change casting, modify relationships), the model only reflects the current state, not its historical state. A migration using that model may fail when run on a fresh database.
- **Schema mismatch during execution** - Eloquent expects the database schema to match the model definition. During migrations, the schema is in flux, which can cause failures when a model references fields or relationships that don't exist yet.
- **Unpredictable migration order** - If Migration 1 uses a model that reflects changes from Migration 2, Migration 1 will break because Migration 2 hasn't run yet.
- **Separation of concerns** - Migrations are specifically designed for database structure, not object-driven data manipulation. Using the DB facade maintains this separation.
- **Long-term stability** - Migrations need to be reproducible indefinitely. Using models breaks this guarantee as soon as the model changes.

<a name="suitable-for"></a>
## Suitable For

- **All migrations** - Use Query Builder or raw SQL for any data manipulation in migrations
- **Data seeding during migrations** - Use DB facade instead of models to populate initial data
- **Schema modifications** - Always use Schema Builder for structural changes
- **Fresh installations** - Ensures migrations run correctly on new environments regardless of model changes

<a name="less-suitable"></a>
## Less Suitable

- **Database seeders** - Seeders are meant for test data and can safely use Eloquent models since they're run separately and aren't part of the migration history
- **One-time scripts** - For scripts that won't be run again, using models may be acceptable (though still not recommended)

<a name="examples"></a>
## Examples

**❌ Bad - Using Eloquent Model:**
```php
use App\Models\Post;

public function up()
{
    Schema::table('posts', function (Blueprint $table) {
        $table->boolean('is_published')->default(false);
    });

    // This will break if the Post model changes
    Post::query()->update(['is_published' => true]);
}
```

**✅ Good - Using Query Builder:**
```php
use Illuminate\Support\Facades\DB;

public function up()
{
    Schema::table('posts', function (Blueprint $table) {
        $table->boolean('is_published')->default(false);
    });

    // This will always work
    DB::table('posts')->update(['is_published' => true]);
}
```

**✅ Good - Using Raw SQL:**
```php
use Illuminate\Support\Facades\DB;

public function up()
{
    Schema::table('posts', function (Blueprint $table) {
        $table->boolean('is_published')->default(false);
    });

    // Raw SQL is also reliable
    DB::statement('UPDATE posts SET is_published = true');
}
```

<a name="more-info"></a>
## More Info

- [Laravel Migrations Documentation](https://laravel.com/docs/migrations)
- [Laravel Query Builder Documentation](https://laravel.com/docs/queries)
- [Laravel Database Seeding Documentation](https://laravel.com/docs/seeding)
