# Use Policies and Gates for Authorization

<a name="introduction"></a>
## Introduction

Policies and gates are standard components within Laravel that can be used to determine whether an action may be performed.

<a name="why"></a>
## Why

- It allows you to put authorization logic in one place (in a Policy class or Service Provider) instead of separate if statements. This prevents duplicated code  
- The authorization code is more reusable  
- It is decoupled authorization code from business logic (separation of concerns)  
- First class citizen within Laravel, which means it is well maintained, and policies gates can also be used in unit tests.

<a name="suitable-for"></a>
## Suitable For

- Almost every Laravel application

<a name="less-suitable"></a>
## Less Suitable

- For smaller applications it can cause some unnecessary government effort to apply very strict policies and gates. It may be more useful to provide authorization with separate if statements

<a name="more-info"></a>
## More Info

- [https://laravel.com/docs/12.x/authorization#gates](https://laravel.com/docs/12.x/authorization#gates)  
- Save roles and permissions to the database - [https://spatie.be/docs/laravel-permission/v6/introduction](https://spatie.be/docs/laravel-permission/v6/introduction)
