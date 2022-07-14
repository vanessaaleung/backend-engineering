# gRPC
- google Remote Procedure Call
  - In gRPC, a client application can directly call a method on a server application **on a different machine as if it were a local object**, making it easier for you to create distributed applications and services. 
<img src="https://developer.token.io/tailrd_rest_api_doc/content/resources/images/grpc-proto_concept.png" height="300px">

- [Official Documentation](https://grpc.io/docs/what-is-grpc/introduction/)
- HTTP/2 transport

- [Protobuf](#protobuf)
- [Service Implementation](#service-implementation)
- [RPC life cycle](#rpc-life-cycle)

## Protobuf (Protocol Buffers)
- Serializing structured data
- Alternative to JSON. More high-performance

<img src="https://www.ionos.com/digitalguide/fileadmin/DigitalGuide/Screenshots_2020/diagram-of-grpc-workflow.png" height="300px">

- Step 1. Define the structure ofr the data you want to serialize in a proto file
  - Protobuf data is structured as messages.
  - A message is a small logical record of information containing a series of name-value pairs called fields.
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

## Service Implementation
- gRPC is based around the idea of defining a service, specifying the methods that can be called remotely with their parameters and return types. 
- On the server side, the server implements this interface and runs a gRPC server to handle client calls.
- On the client side, the client has a stub (referred to as just a client in some languages) that provides the same methods as the server.
- gRPC lets you define four kinds of service method:

  <img src="https://www.t2.sa/sites/default/files/inline-images/grpc-2.png" height="300px">
  
  - Unary RPCs: the client sends a single request to the server and gets a single response back.
  <br></br>
  ```rpcgen
  rpc SayHello(HelloRequest) returns (HelloResponse);
  ```
  - Server streaming RPCs: the server returns a stream of messages. The client completes once it has all the server's messages.
  <br></br>
  ```rpcgen
  rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);
  ```
  - Client streaming RPCs: the client sends a stream of messages to the server. The server responds with a single message typically after it has received all the client's messages.
  <br></br>
  ```rpcgen
  rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);
  ```
  - Bidirectional streaming RPCs: both sides send a sequence of messages using a read-write stream. The two streams operate independently. The server could wait to receive all the client messages before writing its responses, or it could alternately read a message then write a message, or some other combination of reads and writes. The order of messages in each stream is preserved.
  <br></br>
  ```rpcgen
  rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
  ```

## RPC life cycle
  1. Once the **client calls a stub method**, the server is notified that the RPC has been invoked with the client’s metadata for this call, the method name, and the specified deadline if applicable.
  2. The **server** can then either **send back its own initial metadata** (which must be sent before any response) straight away, **or wait for the client’s request message**. Which happens first, is application-specific.
  3. Once the **server** has the client’s request message, it does whatever work is necessary to **create and populate a response**. The response is then **returned (if successful) to the client** together with status details (status code and optional status message) and optional trailing metadata.
  4. If the response status is OK, then the client gets the response, which **completes the call on the client side**.

### Step 1. Defining the service
_Defining the gRPC service and the method request and response types using protocol buffers_
```protobuf
// Specify a java_package file
option java_package = "io.grpc.examples.routeguide";

// Define a service
service RouteGuide {
  // Define rpc methods inside the service
  
  // Unary streaming
  rpc GetFeature(Point) returns (Feature) {}
  
  // Server-side streaming
  rpc ListFeatures(Rectangle) returns (stream Feature) {}
  
  // Client-side streaming
  rpc RecordRoute(stream Point) returns (routeSummary) {]
}

// Define message types
message Point {
  int32 latitude = 1;
  int32 longitude = 2;
}
```

### Step 2. Generating client and server code
_Generate gRPC client and server interfaces using protoc_

Following classes will be generated from the example above
- `Feature.java`, `Point.java`, `Rectangle.java` and others which contain all the protocol buffer code to populate, serialize, and retrieve our reques tand response message types.
- `RouteGuideGrpc.java`: contains
  - a base class for `RouteGuide` servers to implement `RouteGuideGrpc.RouteGuideImplBase` with all the methods defined in the `RouteGuide` service.
  - stub classes that clients can use to talk to a `RouteGuide` server

### Step 3. Creating the server

### Step 4. Implementing RouteGuide

### Step 5. Starting the server

### Step 6. Creating the client

#### Step 6.1. Instantiating the stub
- **Stub**: client-side object representing the server. For example, calling `stub.sayHello(...)` = making a remote call to the server
  - Blocking stub: the RPC calls waits for the server to respond. Mainly for testing purposes, as we don't wanna block threads if possible.
  - Non-blocking/Asynchronous stub: response is returned asynchronously
  - Future stubs
- **Channel**: connects to the remote server, specifying the server address and port we want to connect to
<br></br>
- Step 6.1.1 - Create a gRPC channel for the stub
  ```java
  public RouteGuideClient(String host, int port) {
    this(ManagedChannelBuilder.forAddress(host, port).usePlaintext());
  }
  
  /** Construct client for accessing RouteGuide server using the existing channel. */
  public RouteGuideClient(ManagedChannelBuilder<?> channelBuilder) {
    channel = channelBuilder.build();
    blockingStub = RouteGuideGrpc.newBlockingStub(channel);
    asyncStub = RouteGuideGrpc.newStub(channel);
  }
  ```
#### Step 6.1.2 - Use the channel to create our stubs
  ```java
  public RouteGuideClient(...) {
    ...
    blockingStub = RouteGuideGrpc.newBlockingStub(channel);
    asyncStub = RouteGuideGrpc.newStub(channel);
  }
  ```

### Step 6.2. Calling service methods

### Other Notes
- Implement a method for each RPC

  **.proto**
  ```rpcgen
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
  
