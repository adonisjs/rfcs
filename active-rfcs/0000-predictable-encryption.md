- Start Date: 2022-07-02
- Reference Issues:
	- https://discord.com/channels/423256550564691970/423263766570991617/991841509026582528
	- https://discord.com/channels/423256550564691970/423263695804563456/992536521682993253
- Implementation: N/A

## Summary

The RFC introduces an option to `Encryption.encrypt` that will allow developers to generate predictable encryptions. When using this option, each time that you encrypt the same value with the same options, it will generate the same encrypted string.

## Motivation

While unpredictable encryption is good for many things, the lack of predictable encryption is a barrier to one specific motivation: encrypting a value to be stored in the database that you want to be able to index and query against. Without predictable encryption, the alternative isn't to use unpredictable encryption--it's to store the value in plain text.

By way of example, one can imagine wanting to use email address as the uid for Auth, but to store them encrypted instead of as plain text. With predictable encryption, you can do this. (Hashing won't work for this use case, because the value needs to be decryptable so we can, for example, send an email to the user.) 

## Basic usage

The `Encryption.encrypt` method imported from `@ioc:Adonis/Core/Encryption` can be updated to add one more argument, `predictable?: boolean`.
:

```ts
import Encryption from "@ioc:Adonis/Core/Encryption";

Encryption.encrypt('user@example.com', null, 'login', true);
// => f78924i987f2p9i8f2p89if298pfi

Encryption.encrypt('user@example.com', null, 'login', true);
// => f78924i987f2p9i8f2p89if298pfi

Encryption.encrypt('user@example.com', null, null, true);
// => ao9xu7foa9xfu9a7ufx9aufxa97uf

Encryption.encrypt('user@example.com', null, null, true);
// => ao9xu7foa9xfu9a7ufx9aufxa97uf
```

Decryption would continue to work for predictably-encrypted values with no changes:

```ts
import Encryption from "@ioc:Adonis/Core/Encryption";

Encryption.decrypt('f78924i987f2p9i8f2p89if298pfi', 'login');
// => user@example.com

Encryption.decrypt('ao9xu7foa9xfu9a7ufx9aufxa97uf');
// => user@example.com
```

## Detailed Design

Here is an example of how the typings would look if this change is adopted. Please note that `decrypt` is unchanged.

```ts
declare module '@ioc:Adonis/Core/Encryption' {
  export interface EncryptionContract {
    encrypt(payload: any, expiresIn?: string | number, purpose?: string, predictable?: boolean): string;
    decrypt<T extends any>(payload: string, purpose?: string): T | null;
  }
}
```

If the new argument is not passed in, or is passed in as `false`, the Encryption.encrypt method behavees as it does today.

```ts
import Encryption from "@ioc:Adonis/Core/Encryption";

Encryption.encrypt('user@example.com', null, 'login', false);
// => g7afoe7kofae97kfoa9euk78foa9e

Encryption.encrypt('user@example.com', null, 'login', false);
// => 9dk3gd9gpdk91gpdk9pdk9dpi9gpd

Encryption.encrypt('user@example.com');
// => i86i28if1ikfigkdpg3dp0kd3pkg30

Encryption.decrypt('g7afoe7kofae97kfoa9euk78foa9e', 'login');
// => user@example.com

Encryption.decrypt('9dk3gd9gpdk91gpdk9pdk9dpi9gpd', 'login');
// => user@example.com

Encryption.decrypt('i86i28if1ikfigkdpg3dp0kd3pkg30');
// => user@example.com
```

## Known limitations

Encrypting a value predictably does so at the cost of security, so predictable encryption should only be used when the alternative is to _not_ encrypt a value. The documentation should teach developers about that pitfall / best practice.

## Alternative Designs

Instead of adding yet another argument, `Encryption.encrypt` could be changed to receive an `EncryptionOptions` object:

```ts
type EncryptionOptions = { expiresIn?: string | number, purpose?: string, predictable?: boolean}
```
However, this would be a breaking change unless the method was changed to support both the old and new signature, which would likely mean deprecating the old signature and removing it in the next major version of Adonis:

```ts
declare module '@ioc:Adonis/Core/Encryption' {
  export interface EncryptionContract {
    // Warns about deprecation
    encrypt(payload: any, expiresIn?: string | number, purpose?: string): string;
    // Preferred and documented usage
    encrypt(payload: any, EncryptionOptions): string;
    
    decrypt<T extends any>(payload: string, purpose?: string): T | null;
  }
}
```

## Breaking change adoption strategy

N/A - Not a breaking change.