# Apollo, gRPC, Hermes
- [Apollo](#apollo)
- [gRPC](#grpc)
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

### gRPC
- google Remote Procedure Call
  - In gRPC, a client application can directly call a method on a server application **on a different machine as if it were a local object**, making it easier for you to create distributed applications and services. 
<img src="https://grpc.io/img/landing-2.svg" height="300px">

- [Official Documentation](https://grpc.io/docs/what-is-grpc/introduction/)
- Open source
- HTTP/2 transport
- Protobuf (Protocol Buffers) schema
  - Serializing structured data
  - Alternative to JSON. More high-performance
  - Step 1. Define the structure ofr the data you want to serialize in a proto file
    - Protobuf data is structured as messages. A message is a small logical record of information containing a series of name-value pairs called fields.
  ```protobuf
  message PersonName {
    string first_name = 1;
    repeated string middle_name 2;
    string family_name = 3;
  }
  ```
  - All fields are optional
  - Need to pick a number that hasn't been used before. Don't need to be in sequential
  ```protobuf
   syntax = "proto3"
   
   service Greeter {   // Interface for a Greeter service
    // Sends a greeting
    rpc SayHello (HelloRequest) returns (HelloReply){}    // One rpc, with parameters and results
   }
   // Request message definition: containing a string field 'name'
   message HelloRequest {            
    string name = 1;
   }
   // Response message definition
   message HelloReply {
    string message = 1;
   }
   ```
  - Step 2. Use protoc (Protobuf compiler) to generate data access classes in your preferred languages from your proto definition
   - Provides simple accessors for each field like `name()` and `set_name()`
   - Maven plugin: `mvn generate-sources` will invoke protoc and compile all the schemas to java codes
   - For example, compiling the above protobuf will generate 
     - HelloRequest.java, HelloReply.java
     - GreeterGrpc.java
- gRPC and futures
  - Java - CompletableFuture
  - gRPC - ListenableFuture (Guaba)
- gRPC Service Implementation
  - gRPC is based around the idea of defining a service, specifying the methods that can be called remotely with their parameters and return types. 
  - On the server side, the server implements this interface and runs a gRPC server to handle client calls.
  - On the client side, the client has a stub (referred to as just a client in some languages) that provides the same methods as the server.
  ```java
  class GreeterImpl extends GreeterGrpc.GreeterImplBase
  ```
  - Implement a method for each RPC
  
    **.proto**
    ```protobuf
    rpc SayHello (HelloRequest) returns (HelloReply) {}
    ```
    **.java**
    ```java
    public void sayHello(
      HelloRequest request,
      StreamObserver<HelloReply> responseObserver) {...}
    ```
  - StreamObserver<T>: a way to send back a reply
- gRPC Service Concat in proto
  ```protobuf
  syntax = "proto3";
  
  message ConcatRequest {
    string string_one = 1;
    string string_two = 2;
  }
  message ConcatResponse {
    string response = 1;
  }
  service ConcatService {
    rpc Concat(ConcatRequest) returns (ConcatResponse) {}
  }
  ```
- gRPC Service Concat in Java
```java
class ConcatResource extends ConcatServiceImplBase {
  @Override
  void concat(final ConcatRequest request,
            final StreamObserver<ConcatResponse> response) {
    response.onNext(
      ConcatResponse.newBuilder()     // Build the response
        .setResponse(request.getStringOne() + request.getStringTwo())
        .build()
    );
    response.onCompleted();      // tell the caller all response have been sent, no need to wait for anything anymore
}
```
- gRPC Server in plain Java
```java
import io.grpc.Server;
import io.grps.ServerBuilder;

public class ConcatServer {
  private Server server;
  private void start() throws IOException {
    server = ServerBuilder.forPort(50051)
       .addService(new ConcatResource())
       .build()
       .start();
  ...
}
```
- gRPC Server in Apollo
```java
public static void main(final String...args) {
  final Service service = 
    ...
    .withModule(GrpcServerModule.create())
    .withModule(GrpcServerInterceptorsModule.create())
static void configure(final Environment environment) {
  final GrpcServer grpcServer = environment.resolve(GrpcServer.class);
  grpcServer.addService(new ConcatResource());  // add your API/Resource
  grpcServer.start();
```
- gRPC Client
  - Channel: connects to the remote server, using host:port or service discovery
  - Stub: client-side object representing the server, Call `stub.sayHello(...)` = make a remote call to the server
    - Blocking stub: mainly for testing purposes, don't wanna block threads if possible
    - Async stub
    - Future stubs
  - gRPC Client in Java
  ```java
  import io.grpc.ManagedChannel;
  import io.grpc.ManagedChannelBuilder;
  ManagedChannel channel = ManagedChannelBuilder.forAddress(host, port).build();
  GreeterGrpc.GreeterBlockingStub blockingStub = GreeterGrpc.newBlockingStub(channel);
  blockingStub.sayHello(...)
  ```
  - gRPC Client in Apollo
  ```java
  public static void main() {
    final Service service = ...
  }
  GrpcChannelFactory chFactory = env.resolve(GrpcChannelFactory.class);
  ManagedChannel channel = chFactory.forTarget("nls://greeter").usePlaintext().build();   // nls: service discovery protocol + service_name
  GreeterGrpc.GreeterServiceStub stub = GreeterGrpc.newStub(channel);
  ```
  - gRPC Client Call
  ```java
  HelloRequest request = HelloRequest.newBuilder()
    .setName("")
    .build();
  try {
    HelloReply response = blockingStub.sayHello(request);
    logger.info("Greeting: " + response.getMessage());
  } catch (StatusRuntimeException e) {
    logger.log(...);
    return;
  }
  ```
  - gRPC Client Call from a gRPC service
  ```java
  favoriteSongServiceStub.withDeadline(Deadline.after(1000, MILLISECONDS))
    .getFavorite(
      GetTrackRequest.newBuilder()
        .setUsername(request.getName())
        .build())
  ```
  - grpcurl
    ```
    grpcurl -max-time 2 -plaintext -d '{JSON_INPUT}' <host>:<port> ...
    ```
    will translate JSON_INPUT to protobuf
  - spgrpcurl: wrapper for grpcurl
  
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

  
  
  
