- Start Date: 2020-09-07
- Target Major Version: 5.0
- Implementation PR: (leave this empty)

# Summary

Current validation for any type of numbers is quite limited.
The unique rule we have is `unsigned` while we may need to validate a few other things:

- Is the number positive/negative?
- Is the number an integer?
- Has this number more than X decimal?

# Detailed design

4 rules need to be added.

| Name | Description |
| ------------- | ------------- |
| `integer` | Verify that the number is an integer (https://github.com/adonisjs/validator/pull/90) |
| `decimal` | Verify that the number has a maximum of X decimal |
| `positive` | Verify that the number is positive |
| `negative` | Verify that the number is negative |

The `negative` rule will do exactly the same as the current `unsigned` rule.
Since the name is easier, I propose to deprecate the use of `unsigned` in favour of `negative`.

Concerning the `decimal` rule, it's quite difficult in the JavaScript land to verify the number of decimal that a number has.
One way (and the best way in my opinion) is to use a regular expression.

```ts
// Testing for maximum 2 decimals
const regex = /^-?[0-9]+(\.\d{0,2})?$/;

regex.test('22') // true
regex.test('22.2') // true
regex.test('22.22') // true
regex.test('22.222') // false
```

We also need to sanitize the input first before converting it to a string.

```ts
Number(1e7).toString()
```
