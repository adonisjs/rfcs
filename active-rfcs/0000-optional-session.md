- Start Date: 2020-09-07
- Target Major Version: 5
- Reference Issues: [adonisjs/session #58](https://github.com/adonisjs/session/issues/58)
- Implementation PR: (leave this empty)

# Summary

Introduce a way to disable session per route.

# Basic example

Here's an example if we decide to use the session's configuration to define the route that shouldn't have a session.

```ts
const sessionConfig: SessionConfig = {
  // ...

  disableForRoutes: [
  	'/',
  	'/about',
  ],

  // ...
}
```

# Motivation

At the moment, any routes create a session by default.
This can create a performance issue for high traffic application and may be undesired.

We should provide an easy and developer-friendly way to disable session on some routes.

# Detailed design

Here are the specs:

- If you already have a session, it shouldn't be destroyed when accessing a "no-session" route.
- If you already have a session, you shouldn't have access to it when accessing a "no-session" route.
- If you don't have a session, you should not create one when accessing a "no-session" route.

Here's an example:

```ts
Route.get('/', '...')       // Session is disabled
Route.get('/about', '...')  // Session is disabled

Route.get('/dashboard', '...')
```

- If we access `/dashboard`, a session will be created and kept during the whole navigation.
- If we access `/` or `/about`, no session will be created.
- If we access `/dashboard` and then go to `/` or `/about`, the session will be created in `/dashboard` but not available for the developer in `/` and `/about`.
