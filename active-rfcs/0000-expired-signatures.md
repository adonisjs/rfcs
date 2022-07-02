- Start Date: 2022-07-02
- Reference Issues: N/A
- Implementation: N/A

## Summary

The RFC introduces a new method to `RequestContract` that will allow developers to check if a signed URL _used to_ be valid, but has expired.

## Motivation

A common use case for Signed URLs is to send a signed URL to a user's email inbox for features like "magic login emails" or "forgot password" workflows. It is a good security practice to expire those Signed URLs so that if the user's email archive is compromised in the far future, the attacker will not be able to authenticate with your system.

However, when checking Signed URLs today (using `RequestContract#hasValidSignature`), there is no way to tell whether the signature is invalid because it was tampered with or because it expired. The desired user experience for the two scenarios is different (e.g. showing a generic error page vs. allowing the user to generate a fresh Signed URL), but Adonis developers currently have no ability to differentiate between the two scenarios.


## Basic usage

The `Request` (fulfilling the `RequestContract`) available on the `HttpContextContract` can be updated to add another method, `hasExpiredSignature`:

```ts
import type { HttpContextContract } from '@ioc:Adonis/Core/HttpContext'

export default class SomethingController {
  public async show({ request }: HttpContextContract) {
    if (request.hasValidSignature()) {
      return this.showValid()
    }
    else if (request.hasExpiredSignature()) {
      return this.showTryAgain()
    }
    else {
    	return this.showError()
    }
  }
  // ...
}
```

## Detailed Design

Here is an example of how the typings would look if this change is adopted. Please note that `hasValidSignature` is unchanged.

```ts
declare module '@ioc:Adonis/Core/Request' {
  export interface RequestContract {
    hasExpiredSignature(purpose?: string): boolean;
    hasValidSignature(purpose?: string): boolean;
  }
}
```

## Known limitations

N/A

## Alternative Designs

Instead of adding a new method, `RequestContract#hasExpiredSignature` could be changed to return a different value for each scenario (valid and unexpired, invalid due to expiry, invalid due to tampering). However, this would likely lead to a worse developer experience since the return values for the invalid scenarios would either be truthy (thus eliminating the current ability to use `hasValidSignature` in the predicate of a conditional) or either `null` or `false` depending on why it was expired (which is harder to remember).

```ts
declare module '@ioc:Adonis/Core/Request' {
  export interface RequestContract {
    /*
     * Returns `true` if the signature is valid.
     * Returns `false` if the signature is invalid due to tampering.
     * Returns `null` if the signature would have been valid in the past, but has expired.
     */
    hasValidSignature(purpose?: string): boolean | null;
  }
}
```

## Breaking change adoption strategy

N/A - Not a breaking change.