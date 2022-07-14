# Apollo, gRPC, Hermes
- [Apollo](#apollo)
- [gRPC](https://github.com/vanessaaleung/backend-engineering/blob/main/gRPC.md)
- [Hermes](#hermes)
- [Apollo and Hermes](#apollo-and-hermes)

## Apollo
- Apollo: manages modules lifecycle. A module can be a server or anything
```java
final Service service = 
  Services.usingName(SERVICE_NAME)
    .withModule(...)
    .withEnvVarPrefix("")
    ...
    build();
StandaloneService.boot(service, args);
```
- Apollo API - Environment
  - `Environment.resolve()`: resolves an instance of a class out of the underlying module system
  - `.config()`: get all the config values
  - `.closer()`: shutting down the application gracefully
  - `.routingEngine()`: register Hermes endpoints
- Config file:
  - `<service-name>.conf`
  - run locally, may not want to use production secret/database, use `<service-name>-user.conf` will override the production config
  ```protobuf
  grpc.server {
    port: 5990
    healthService.enabled: true
  }
  hermes.server {
    port: 5700
  }
  ```
- Service Auth
  - Authentication: proven identity of the calling service
  - Authorization: control what different calling services have access to
  ```protobuf
  serviceauth {
    enabled: true
    enforcing: true
    defaultAction: "deny"
    accessPolicies: [
      { matchType: "prefix", path: "/_meta/0/", method: "GET", principals: [] },
      ...
    ]
  ```
- Apollo as an Application Server: need to add one or more of the following modules
  - GrpcServerModule
  - HttpServerModule
  - HermesServerModule
- Apollo as a clinet: enable Apollo making outgoing calls, need to add one or more of the following modules
  - GrpcChannelFactoryModule
  - HermesClientModule
  - Currently has five different Google data centers, set with `apollo.domain`. It's set automatically in production.


## Hermes
- Network protocol
- Methods such as GET, POST, PUT
- Requests are uniquely identifiable, associated with unique identifier
- Hermes servers may return multiple payloads/responses, can do batch requests which HTTP can't do
- gRPC: Context from which it retrieves data is a thread local
  - In Java each action runs on a thread, each thread has one or more local context, which can be attached with piecies of info
  ```java
  RequestMetadataUtils.fromCurrentContext()
    .getUserInfo().get().getUsername()
  ```
- Hermes
  ```java
  HermesMessageUtil.userInfo(context.request()).getUsername()
  requestContext.requestScopedClient()
  ```
- curl cannot be used with Hermes
- jhurl: java-hermes-based curl
- hmcat: dump data from hermes sockets
- hmgrep: filter hermes message streams produced by hmcat
- hmtop: top

```bash
echo '...' | jhurl - X POST -z tcp://localhost:5700 "hm://concat/concat"
```

## Apollo and Hermes
- `Environment.routingEngine()`: register routes, a route is an association from an HTTP like path/verb to a java method
- RequestContext
  - each request has an associated context
  - it contains information about the current request
  - `.request()`: the current request. `Request.parameters()`: query params of the request (?param=value&x=y). `Request.payload()`: the body of the request.
  - `.pathArgs()`: path parameters of the request (/v1/user/<userId>)
  - `.requestScopedClient()`: a client to make upstream calls. Client that forwards important headers.
- Responding to a Hermes Request
```java
Route.async("POST", "/concat", this::concat)
    .withMiddleware(ConcatHermesResources::serialize)

// Return a future that has a response class that has a string in it.
CompletionStage<Response<String>> concat(final RequestContext context) {
    LOG.info("Received concat request over Hermes.")
    // Get request payload from request context.
    final String payload = context.request().payload().get().utf8();
    // parse request to an internal representation for the service.
    final ConcatRequest request = objectMapper.readValue(payload, ConcatRequest.class);
    // validate if request is valid or not
    return completedFuture(
      Response.forPayload(
        ConcatUtils.concatenate(request.stringOne(), request.stringTwo());
      )
    )
```
- Sending a Hermes Request
```java
// Return a future that has a response class that has a string in it.
CompletionSstage<Response<String>> concat(final RequestContext context) {
    LOG.info();
  // Get a client from the request itself
  final Client client = context.requestScopedClient();
  // Construct a request for the upstream service
  final Request request = Request.forUri("");
  // Call the upstream service, map response and reply back.
  return client.send(request).thenApply(response -> {
    final String name = parse(response.payload().get()).getName();
    return Response.forPayload(
      ConcatUtils.concatenate("My awesome track ", name)
    );
  });
}
```
- gRPC or Hermes

|Use Case                |gRPC                                      |Hermes                           |
|------------------------|------------------------------------------|---------------------------------|
|Backend to backend calls|Preferred if supported by upstream service|Preferred if gRPC isn't supported|
|Client (mobile or web) to backend calls|N/A                        |Supported                        |

  
  
  
