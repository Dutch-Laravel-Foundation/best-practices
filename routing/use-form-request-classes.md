# Use Form Request Classes

<a name="introduction"></a>
## Introduction

Laravel Form Request classes extract validation and authorization logic from controllers into dedicated classes. Type-hinting a Form Request in a controller method triggers automatic validation and authorization before the method executes. This keeps controllers thin and validation logic reusable.

<a name="why"></a>
## Why

- **Separation of concerns**: Validation logic lives in its own class, not cluttering controller methods
- **Reusability**: The same Form Request can be used across multiple controllers or actions
- **Automatic execution**: Type-hinting the Form Request triggers validation and authorization automatically — no manual `validate()` call needed
- **Safety**: Using `$request->validated()` ensures only validated data is passed to mass operations, preventing unvalidated fields from leaking through

<a name="suitable-for"></a>
## Suitable For

- Any controller method that accepts user input
- Forms with multiple validation rules
- Endpoints where authorization and validation are closely related
- APIs where consistent validation error responses matter

<a name="less-suitable"></a>
## Less Suitable

- Extremely simple endpoints with one or two trivial validation rules
- Closure-based routes in prototyping or testing scenarios

<a name="examples"></a>
## Examples

### Extract Validation into Form Requests

```php
// Bad: inline validation in controllers
public function store(Request $request)
{
    $request->validate([
        'title' => 'required|max:255',
        'body' => 'required',
    ]);
}

// Good: dedicated Form Request class
public function store(StorePostRequest $request)
{
    Post::create($request->validated());
}
```

### Always Use `validated()`

Never use `$request->all()` for mass operations:

```php
// Bad: includes unvalidated fields
Post::create($request->all());

// Good: only validated data
Post::create($request->validated());
```

### Prefer Array Notation for Rules

Array syntax is more readable and composes cleanly with `Rule::` objects. Prefer it in new code, but match existing convention:

```php
// Preferred for new code
'email' => ['required', 'email', Rule::unique('users')],

// Follow existing convention if the project uses string notation
'email' => 'required|email|unique:users',
```

### Use `Rule::when()` for Conditional Validation

```php
'company_name' => [
    Rule::when($this->account_type === 'business', ['required', 'string', 'max:255']),
],
```

### Use the `after()` Method for Custom Validation

Use `after()` instead of `withValidator()` for custom validation logic that depends on multiple fields:

```php
public function after(): array
{
    return [
        function (Validator $validator) {
            if ($this->quantity > Product::find($this->product_id)?->stock) {
                $validator->errors()->add('quantity', 'Not enough stock.');
            }
        },
    ];
}
```

<a name="more-info"></a>
## More Info

- [Laravel Form Request Validation Documentation](https://laravel.com/docs/validation#form-request-validation)
- [Laravel Validation Rules Documentation](https://laravel.com/docs/validation#available-validation-rules)
- [Use Route Model Binding](./use-route-model-binding.md) — for automatic model resolution in controllers
- [Use Action Classes for Business Logic](../project-structure-and-code-architecture/use-action-classes-for-business-logic.md) — for extracting business logic from controllers
- [Laravel Boost Best Practices PR](https://github.com/laravel/boost/pull/628)
