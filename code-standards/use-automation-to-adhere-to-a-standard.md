# Use Automation to Adhere to a Standard

<a name="introduction"></a>
## Introduction

Adhering to a given standard can be done manually or via automated tooling. This tooling can come in the form of IDE plugins, git hooks, CI/CD steps. The fact that the developer doesn't have to 'think' about the *exact* rules defined in the standard leaves room for the actual problem at hand and not the formatting of the code itself.

<a name="why"></a>
## Why

- By utilizing automated tooling for adhering to the code (git hook, IDE features, CI/CD automated actions) less (human) time and effort is spent on formatting than needed. It can and should become a non-issue.  
- Automation ensures consistent application of the chosen standard across the entire codebase. When rules are adjusted and/or updated automation can again ensure consistent application of the updates.

<a name="suitable-for"></a>
## Suitable For

- Medium to large projects

<a name="less-suitable"></a>
## Less Suitable

- Smaller projects might suffice with just the IDE-integration or manually running of a tool once in a while to ensure consistency.

<a name="more-info"></a>
## More Info

- [https://laravel.com/docs/12.x/pint](https://laravel.com/docs/12.x/pint)  
- [https://github.com/marketplace/actions/laravel-pint](https://github.com/marketplace/actions/laravel-pint)
