- Start Date: 2021-05-17
- Reference Issues: N/A
- Implementation: N/A

## Summary

The RFC introduces the support for WebSockets to the framework as a first-class citizen.

Unlike v4, we **should NOT** roll out our implementation of WebSockets, as it is practically impossible to create and maintain the clients for different platforms.

Instead, we should make use of `socket.io`. Socket.io has a [broad range of clients](https://socket.io/docs/v4)  and v4 also allows creating [`websocket` only](https://socket.io/docs/v4/client-initialization/#transports) connections.

## Motivation

Node.js is well known for its real-time capabilities, and being the mainstream framework, we should offer a delightful experience when creating real-time services on top of WebSockets.

## Basic usage

The code will be published as a separate official package (`@adonisjs/ws`), and one can import the initialized WebSocket server from the container as follows:

```ts
import Ws from '@ioc:Adonis/Addons/Ws'

Ws.on('message', () => {
})

Ws.on('typing', () => {
})
```

Define events inside a namespace

```ts
import Ws from '@ioc:Adonis/Addons/Ws'

Ws
  .namespace('chat')
  .on('message', () => {
  })
```

Use middleware

```ts
import Ws from '@ioc:Adonis/Addons/Ws'

Ws
  .on('message', () => {
  })
  .middleware('auth')
```

## Detailed Design

The WebSocket support should follow/use the existing patterns used by the framework and hide most of the complexities and boilerplate. 

Since we are using `socket.io`, it will do the heavy lifting for managing rooms/channels and multiplexing. We just need to take care of the following:

- Allow using Controllers to handle events. Just like routes and HTTP controllers.
- Allow using Middleware just like the HTTP requests.
- Support for all the socket.io features.

### Socket context

Just like the [HTTP context](https://docs.adonisjs.com/guides/context), we will create a context instance for the WebSocket connections as well. It will have more or less the same properties alongside the [socket instance](https://socket.io/docs/v4/server-socket-instance/).

### Event listeners

Socket.io make you to register the event listeners on every socket instance. For example:

```ts
io.on('connection', (socket) => {
  socket.on('message', () => {})
})
```

I want to move the event handlers at the top level and internally use [socket.onAny](https://socket.io/docs/v4/listening-to-events/#socket-onAny-listener) to invoke event handlers defined on the AdonisJS specific `Ws` object.

```ts
Ws.on('message', () => {})
```

Also, one will receive the Websocket context as the first argument and the event data as the 2nd argument.

```ts
Ws.on('message', (ctx, data) => {
  ctx.socket
  ctx.auth.user
  ctx.request
})
```

### Controllers
Just like the HTTP routes, one should be able to bind a controller method as the event handler. For example:

```ts
import Ws from '@ioc:Adonis/Addons/Ws'

Ws.on('message', 'ChatController.handleMessage')
```

And define the controllers as follows:

```ts
export default class ChatController {
  public async handleMessage (ctx, data) {
  }
}
```

### Working with namespaces
Socket.io makes use of the `of` method to create a [namespace](https://socket.io/docs/v4/namespaces/). I personally try to keep naming more descriptive and will opt for the `namespace` method.

```ts
Ws.namespace('chat')
```

Optionally one can register a connection handler as follows. The connection runs for every new socket connection after all the middleware.

```ts
Ws
  .namespace('chat')
  .connected(() => {})
```

Similarly, a `disconnected` handler is invoked when a socket disconnects. This is similar to the `socket.on('disconnect')` event of socket.io.

```ts
Ws
  .namespace('chat')
  .disconnected(() => {})
```  

#### Dynamic namespaces

Namespaces in socketio can be [dynamic](https://socket.io/docs/v4/namespaces/#Dynamic-namespaces) using a regular expression or a function. I believe we can make it work more like HTTP routes.

**I have not explored the technical possibilities here, but it should be doable to have routes like params in ws namespaces.**

```ts
Ws.namespace('/chat/:channel')
```

### Middleware
Just like HTTP middleware, one should be able to define middleware for all or a specific namespace.

Middleware registered globally will be applied to all the namespaces.

```ts
Ws.middleware.register([
  () => import('App/Middleware/Name')
])
```

Named middleware can be referenced on a specific namespace.

```ts
Ws.middleware.registerNamed({
  auth: () => import('App/Middleware/Auth')
})

Ws
  .namespace('chat')
  .middleware('auth')
```

### Rooms
Just like socket.io, only the server can make a socket join a given room. For example:

```ts
Ws.on('connection', ({ socket }) => {
  socket.join('room name')
})
``` 

Socketio doesn't have an API for clients to request for joining a room. It is always the server that adds a socket to a room.

### Broadcasting API
The broadcasting API will be same as [socket.io](https://socket.io/docs/v4/emit-cheatsheet/).

### CORS
Socket.io has its configuration for CORS. In our case, we can inherit from the CORS config used by the HTTP server but still allowing a separate CORS config for the WebSocket server.

### Clients
One can continue using the existing Socket.io clients. Nothing needs to change for them to work with AdonisJS.

## Known limitations
NONE

## Breaking change adoption strategy
NONE
