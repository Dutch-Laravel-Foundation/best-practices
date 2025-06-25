# Claude Instructions for Laravel Best Practices

This document provides instructions for using Claude (or other AI assistants) to create, verify, and maintain Laravel best practices in this repository.

## Table of Contents

- [Creating a New Best Practice](#creating-a-new-best-practice)
- [Verifying an Existing Best Practice](#verifying-an-existing-best-practice)
- [Quality Checklist](#quality-checklist)
- [Repository Structure](#repository-structure)
- [Formatting Guidelines](#formatting-guidelines)
- [Example Prompts](#example-prompts)

## Creating a New Best Practice

When asking Claude to create a new best practice, provide this context:

```
I want to create a new Laravel best practice. Please help me create a markdown file that follows the repository structure and formatting guidelines found in CLAUDE.md.

The best practice should cover: [DESCRIBE YOUR TOPIC]

Please check:
1. If this is a duplicate of existing practices
2. Which directory it should go in
3. Ensure it follows the correct format
4. Use proper linking conventions
```

## Verifying an Existing Best Practice

Use this prompt to have Claude verify a best practice:

```
Please review this Laravel best practice file and check it against the quality checklist in CLAUDE.md. 

[PASTE YOUR MARKDOWN CONTENT]

Specifically verify:
- Format compliance
- Duplicate detection
- Directory placement
- Link formatting
- Content quality
```

## Quality Checklist

### ✅ **Content Structure**
- [ ] Clear, descriptive title using sentence case
- [ ] Introduction section explaining what the practice is
- [ ] "Why" section with bullet points explaining benefits
- [ ] "Suitable For" section describing ideal use cases
- [ ] "Less Suitable" section describing when not to use
- [ ] "More Info" section with additional resources

### ✅ **Technical Format**
- [ ] Uses HTML anchor tags: `<a name="section-name"></a>`
- [ ] Consistent heading structure (H1 for title, H2 for main sections)
- [ ] Proper markdown formatting
- [ ] Code examples use appropriate language tags (```php, ```bash, etc.)

### ✅ **Links and References**
- [ ] **Internal links** use relative paths (e.g., `./other-practice.md` or `../security/auth-practice.md`)
- [ ] **External links** have descriptive titles, not plain URLs
  - ✅ Good: `[Laravel Authorization Documentation](https://laravel.com/docs/authorization)`
  - ❌ Bad: `https://laravel.com/docs/authorization`
- [ ] All links are functional and point to correct locations
- [ ] No broken internal references

### ✅ **Directory Placement**
Check the practice is in the correct directory:

- `project-structure-and-code-architecture/` - App organization, folder structure, architecture patterns
- `code-standards/` - Coding style, formatting, conventions
- `security-and-authentication/` - Security practices, auth patterns
- `database-and-eloquent-orm/` - Database design, ORM usage, migrations
- `routing/` - Route organization, middleware, API routes
- `apis/` - API design, REST/GraphQL practices
- `testing/` - Testing strategies, test organization
- `version-control/` - Git workflows, branching strategies
- `application-performance/` - Optimization, caching, profiling
- `application-rollout/` - Deployment, CI/CD, environments
- `maintenance/` - Monitoring, logging, updates
- `documentation/` - Code documentation, API docs
- `recommended-stacks/` - Technology recommendations
- `packages-and-services/` - Third-party integrations, package recommendations

### ✅ **Duplicate Detection**
- [ ] Check existing practices in the target directory
- [ ] Verify the topic isn't already covered in other sections
- [ ] If similar practices exist, ensure this adds unique value

### ✅ **Content Quality**
- [ ] Clear, concise writing
- [ ] Practical, actionable advice
- [ ] Real-world examples when applicable
- [ ] Balanced view (includes limitations)
- [ ] Laravel-specific (not generic PHP advice)

## Repository Structure

```
best-practices/
├── README.md
├── CLAUDE.md (this file)
├── example.md (reference for styling)
└── [section-directories]/
    ├── README.md (section overview)
    └── [practice-name].md (individual practices)
```

Each section directory should have:
- A `README.md` with section introduction and links to all practices
- Individual practice files with descriptive, kebab-case names

## Formatting Guidelines

### File Naming
- Use kebab-case: `use-policies-for-authorization.md`
- Be descriptive but concise
- Avoid redundant words like "laravel" or "best-practice"

### Content Template
```markdown
# Practice Title

<a name="introduction"></a>
## Introduction

Brief explanation of what this practice involves.

<a name="why"></a>
## Why

- Benefit 1
- Benefit 2
- Benefit 3

<a name="suitable-for"></a>
## Suitable For

- Use case 1
- Use case 2

<a name="less-suitable"></a>
## Less Suitable

- When not to use this practice

<a name="more-info"></a>
## More Info

- [Descriptive Link Title](https://example.com)
- [Another Resource](https://example.com)
```

### Link Examples

**✅ Correct Internal Links:**
```markdown
See also [Code Standards](../code-standards/) for related practices.
Check our [Authentication guide](./auth-best-practices.md) for more details.
```

**✅ Correct External Links:**
```markdown
- [Laravel Documentation](https://laravel.com/docs)
- [PHP Standards Recommendations](https://www.php-fig.org/psr/)
```

**❌ Incorrect Links:**
```markdown
- https://laravel.com/docs (no title)
- [guide](/absolute/path/guide.md) (absolute path)
```

## Example Prompts

### For Creating a Practice
```
Create a Laravel best practice about [TOPIC]. 

Context: I'm contributing to a Laravel best practices repository. Please:

1. Check if this topic already exists by reviewing the repository structure in CLAUDE.md
2. Determine the appropriate directory
3. Create a properly formatted markdown file following the template in CLAUDE.md
4. Ensure all links are formatted correctly (relative paths for internal, descriptive titles for external)
5. Include practical examples and clear explanations

The practice should cover: [SPECIFIC DETAILS]
```

### For Verification
```
Please verify this Laravel best practice against the CLAUDE.md checklist:

[PASTE CONTENT]

Check for:
- Proper formatting and structure
- Correct directory placement
- Link formatting compliance
- Duplicate detection
- Content quality and clarity
```

### For Repository Updates
```
I've added a new best practice file. Please help me:

1. Update the relevant section README.md to include a link to the new practice
2. Verify the main README.md sections list is complete
3. Check that all internal links in the repository are still valid

New file: [PATH/FILENAME]
```

## Notes for Contributors

When using Claude to work with this repository:
- Always reference this CLAUDE.md file in your prompts
- Ask for verification against the quality checklist
- Request help with link formatting and directory structure
- Use Claude to check for duplicates before creating new practices

This ensures consistency and quality across all contributions to the Laravel Best Practices repository.
