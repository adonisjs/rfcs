- Start Date: 2022-01-02
- Reference Issues: N/A
- Implementation PR: N/A

# Summary

I am currently developing a Laravel Telescope clone for AdonisJS, called Adoscope. https://github.com/Julien-R44/adoscope

In order to make this project happen, we need to apply some changes to the Adonis core packages. 
This RFC will mainly outline the needs of Adoscope and then we will probably discuss the best solutions to implement in the Adonis core in order to solve this.

# Motivation

The goal of this RFC is to expose as many events as possible for the most common tasks of an application. This way, package creators will be able to create more things. 
In my case this is essential for Adoscope to be able to happen, and will also be very useful for [adonis5-prometheus](https://github.com/julien-r44/adonis5-prometheus) to offer more metric builtins and nice dasbhoard grafana. 
In the case of Adonis Prometheus, thanks to these modifications, we could imagine to have a beautiful grafana dashboard ready to use which would propose : 
- The number of mails per minute
- The evolution of the average time of the redis orders
- Evolution of the average time of db queries
- The longest DB queries
- And many other things etc...

# Detailed design
> Nomenclature :
> - Entry: An entry represents an event that occurs in the Adonis application and that will be registered in Adoscope. 
>   
>   Currently, the different entries types are: Commands, Events, Exceptions, Logs, Mails, Models ( hooks ), Queries, Redis, > Requests.

First, I will detail the data requirements needed in the case of Adoscope, then we will see the possible technical ways to retrieve them.

## Authenticated user
### Data requirements
For each of the entries types, we need to be able to tell if a user caused it. If so, we must be able to find the user in question.

### Possible solutions
For me the only solution is Local Storage Async. Example :

```ts
Event.on('db:query', (query) => {
  const ctx = HttpContext.get()
  const user = retrieveUserFromAuthTokenOrAnything(ctx)

  AdoscopeEntry.create({
     type: 'QUERY',
     data: { query, user_id }
   })
}) 
```

## Related entries
### Data requirements
During an HTTP request, many things can happen. A request can trigger a database query, then a caching with redis, followed by an email.

For this reason, we need to be able to tell, for each event, which HTTP request caused it.

This way, in Adoscope we can visualize all this. We can know that this HTTP request triggered this or that event.

However, there is another type of entry that can produce other entries: commands. In this case I don't think local storage async is necessary, but is there an API in Adonis that allows you to know which command is executing ?

### Possible solutions
For HTTPs Requests : Storing the corresponding HTTP request entry ID in Async Local Storage, and then retrieving it when an event is triggered.

Since I don't really know how async local storage works in practice for now, I'm going to use pseudocode :
```ts
// Store the entry id of the initiating request in Async Local Storage
Event.on('http:request' (request) => {
  const entry = await AdoscopeEntry.create({
    type: 'REQUEST',
    data: { query }
  })

  asyncLocalStorage.store('current_request_entry_id', entry.id)
})

// Retrieve the entry id of the initiating request from Async Local Storage so we can link related entries
Event.on('db:query', (query) => {
  let idOfInitiatingEntry

  if (app.environnement === 'web') {
    idOfInitiatingEntry = getIdOfInitiatingEntryFromAsyncLocalStorage()
  } else if (app.environnement === 'command') {
    idOfInitiatingEntry = getIdOfInitiatingEntry(/* ?? */)
  }

  AdoscopeEntry.create({
     type: 'QUERY',
     data: { query, idOfInitiatingEntry }
   })
}) 
```

## Requests
### Data requirements
As soon as a request is processed by the Adonis HTTP server, it should be possible to retrieve this data :
- IP Address
- URI
- Method ( HTTP / POST / PUT etc. )
- Triggered Controller and Method ( ex: `UserController.index` )
- Triggered middlewares
- Headers, Payload, Session
- Used View ( if any ). In case the server return is just a JSON, no issues, we store it in Adoscope. However, if we return a View, we shouldn't store the HTML in Adoscope, it wouldn't make sense. So we need to be able to access the View used, and ideally also retrieve the data bindings. This way we only record the name of the template used and display it to the user in Adoscope
- Server response and response status
- Duration of the request

### Possible solutions
- Trigger an event when a request is received by the Adonis HTTP server.
- Using middleware

## Commands
### Data requirements
As soon as a command was executed by Adonis, it should be possible to retrieve this data :
- Command name
- Exit code
- Arguments and flags

### Possible solutions
- Trigger an event when command has been processed before killing the process
- In case command was executed programmatically within the same process, also trigger an event without the exit code as payload

## DB Queries
### Data requirements
As soon as a query was processed by Adonis, it should be possible to retrieve this data :
- Connection name as defined in configuration
- SQL Queries
- Bindings
- Duration

### Possible solutions
- Trigger an event when the query has been executed

## Events
### Data requirements
As soon as an event was triggered, it should be possible to retrieve this data : 
- Name of the event
- Payload
- Listeners count ( how many listeners are listening to this event )

### Possible solutions
```ts
import Event from '@ioc:Adonis/Core/Event'

Event.onAny((event, data) => {
  await AdoscopeEntry.create({
    type: 'EVENT',
    data: { event, data }
  })
})
```

## Redis 
### Data requirements
As soon as a redis command was executed, it should be possible to retrieve this data :
- Command
- Duration

### Possible solutions
- Trigger an event when the redis command has been executed

## Logs
### Data requirements
As soon as a log was written ( with Adonis/Core/Logger ), it should be possible to retrieve this data :
- Log Level 
- Message 
- Additional values

### Possible solutions
- Trigger an event when the log has been written

## Exceptions
### Data requirements
As soon as an exception was thrown, it should be possible to retrieve this data :
- Message
- Code
- Stack trace
- Location and line ( if possible ? )

## Models
### Data requirements
As soon as a model hook was triggered ( created, updated, deleted, retrieved ), it should be possible to retrieve this data :
- Hook name
- Model name
- Model id
- Model changes ( if any )

### Possible solutions
- Trigger an event when the model hook was triggered

## Events disabled by default
In cases where performance is likely to be affected because a lot of events are emitted, we could disable by default the emission. I'm thinking for example of Redis, which will probably be responsible for many events. In this case, the user must explicitly activate the emission: 

```ts
import Redis from '@ioc:Adonis/Addons/Redis'

Redis.enableEvents()
Redis.disableEvents()
```

I was able to develop the Watcher for Mails, and Events. Thanks to this `mail:send` event, the code seems much cleaner :
- https://github.com/Julien-R44/adoscope/blob/main/app/Watchers/MailWatcher.ts
- https://github.com/Julien-R44/adoscope/blob/main/app/Watchers/EventWatcher.ts

I also tried to develop the watcher for the requests, but via a Middleware, especially to be able to test the rest of Adoscope. This solution really seems to be a hackish solution, and some data seems to me finally impossible to recover this way.
- https://github.com/Julien-R44/adoscope/blob/main/app/Middleware/AdoscopeMiddleware.ts

# Drawbacks

It is possible that adding Events everywhere like this could affect the performance of Adonis. 
We will certainly have to do some benchmarks to evaluate the cost of this feature. In particular for events that are often raised, I think in particular events for Requests or Redis.

But as suggested below, offering an API to enable and disable the emission of events could be a solution.

# Alternatives

I don't really see any other solution than Events to achieve this.
