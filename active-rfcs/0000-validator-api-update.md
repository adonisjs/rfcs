- Start Date: 2021-06-17
- Reference Issues: N/A
- Implementation: N/A

## Summary

The RFC proposes the update to Validator API.

This proposal suggests to clean up the api by splitting the roles of methods, removing options from schemes and grouping them into a corresponding groups: rules or a new group - modifiers, that takes the role of value mutation.

The goal is to semantically divide parts of the library, allowing the final user to control on when to modify the value or validate by rule.

## Motivation

Currently it is pretty hard to expand the validator API due to the mix of roles within the library, as some schemes have options that semantically refer to rules and some mutate the value, dividing logic into groups will make validator API even more structured and controllable.

## Basic usage

Currently the validator API is used like this:
```ts
schema.create({
  title: schema.string({ trim: true }, [rules.unique({ table: 'posts', column: 'title' })]),
  slug: schema.string({ trim: true }, [rules.unique({ table: 'posts', column: 'slug' })]),
  excerpt: schema.string.optional(),
  content: schema.string.optional(),
  description: schema.string.optional(),
})
```

The proposed API may look like this:
```ts
schema.create({
  title: schema.string([ 
    modifiers.trim(),
    modifiers.escape(),
    modifiers.casefy('titleCase'),
    rules.minLength(4),
    rules.unique({ table: 'posts', column: 'title' })
  ]),
  slug: schema.string([ 
    modifiers.trim(), 
    modifiers.slugify({ table: 'posts', column: 'slug' }), 
    rules.unique({ table: 'posts', column: 'slug' })
  ]),
  excerpt: schema.string.optional(),
  content: schema.string.optional(),
  description: schema.string.optional(),
})
```

Or may allow the user to do something like this:
```ts
schema.create({
  title: schema.string([ 
    modifiers.trim(),
    rules.minLength(4),
    rules.unique({ table: 'posts', column: 'title' }),
    modifiers.escape(), // <---- Escape after validation
    modifiers.casefy('titleCase'), // <---- Change case after validation
  ]),
  slug: schema.string([ 
    modifiers.trim(), 
    modifiers.slugify({ table: 'posts', column: 'slug' }), 
    rules.unique({ table: 'posts', column: 'slug' })
  ]),
  excerpt: schema.string.optional(),
  content: schema.string.optional(),
  description: schema.string.optional(),
})
```

Currently there are just 3 schema types with options that can be migrated to different groups:

1. **String**, with trim and escape options that can be migrated to modifiers group
2. **Date**, with a format option, that can be moved to modifiers group
3. **File**, with size and extnames, that can be moved to rules group

## Detailed Design

There may be three main logical groups in the library:
1. **Schema** - should validate and check primitive type of incoming data
2. **Rule** - should validate the value against a set of rules
3. **Modifier** - can modify the value, changing it to an expected form

## Known limitations
NONE

## Breaking change adoption strategy

The update can be pushed in two steps:

1. Extract options from schemes to corresponding rules and modifiers and mark old options with `@deprecated` notice, this will allow not to break the library and expand it with new **modifiers**
2. Remove deprecated options some time later in a major update.