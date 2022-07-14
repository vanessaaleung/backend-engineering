# Backend Engineering
- [Monolith vs Microservices Architecture](#monolith-vs-microservices-architecture)
- [Service to Service communication](#service-to-service-communication)
- [Scaling](#scaling)
- [Monitoring](#monitoring)
- [Functional Programming](#functional-programming)
- [Asynchronous Programming](#asynchronous-programming)
- [Thread](#thread)
- [CompletableFuture](#completablefuture)
- [Apollo, gRPC, Hermes](https://github.com/vanessaaleung/backend-engineering/blob/main/Apollo-gRPC-Hermes.md)
- [Testing](https://github.com/vanessaaleung/backend-engineering-learning/blob/main/testing.md)

## Monolith vs Microservices Architecture
- Monolith: User Interface + Business Logic + Data Access Layer in a single code base
- Microservices: Split up Monolith, have a dedicated system and databse for each business logic
  - Benefits: independently scale a system which is heavily used
<img src="https://d1.awsstatic.com/Developer%20Marketing/containers/monolith_1-monolith-microservices.70b547e30e30b013051d58a93a6e35e77408a2a8.png" height="300px">
  
## Service to Service communication
- HTTP
- Hermes: more high performance than HTTP
- gRPC: remote procedure call

## Scaling
- Vertical scaling (bigger machines)
- Horizontal scaling (more machine)
- Virtual Machines: virtualizes the hardware
- Kubenetes: virtualizes the operating system

## Monitoring
- Logging: e.g. Google Cloud Logging
- Metrics: time-series data showing how things changed, e.g. Grafana (dashboard), pager duty (alerting)
- Distributed request tracing: shows how a request goes through different systems/services, how long does it take, e.g. Lightstep
<img src="https://images.saasworthy.com/lightstep_4904_screenshot_1574232008_0sov2.png" height="300px">

## Functional Programming
- Everything is immutable, avoids mutability
- [Functional Interfaces](#functional-interfaces)
- [Lambda Expressions](#lambda-expressions)
- [Method Reference](#method-reference)
- [Optionals](#optionals)
- [Streams](#streams)

### Functional Interfaces
- An interface with exactly one abstract method
- Can be implemented by Lamdba Expressions or Method Reference
- Example: Predicate
```java
public interface predicate<T> {
  boolean test(T t);
  
  default Predicate<T> negate() {
    return (t) -> !test(t);
  }
  
  ... other non-abstract methods
}
```

### Lambda Expressions
- Syntactic sugar
- Instances of Functional Interfaces
- Implements the only method in the Functional Interface
- Implemented inline, as a variable or as in put to a method like a stream. The compiler infers the underlying Functional Interface using the type of the variable or parameter.
- Benefits: more readable, type safety
```java
// Syntax
// 1st style
(p1, p2) -> doSomething(p1, p2);

// 2nd style: curly braces for multi statement method body or return statement
(parameters) -> {
  return doSomething(parameters);
}
```
```java
// Example 1
@FunctionalInterface
public interface Runnable {
  public abstract void run();
}

// 1st style
Runnable runnable = new Runnable() {
  @Override
  public void run() {
    System.out.println("Hello");
  }
};
new Thread(runnable).start();

// 2nd style
Runnable runnable = () -> System.out.println("Hello");
new Thread(runnable).start();
```

```java
// Example 2
// 1st style
Comparator<Integer> c = 
  new Comparator<Integer>() {
    @Override
    public int compare(Integer a, Integer b) {
      return a - b;
    }
  };

// 2nd style
Comparator<Integer> c = (a, b) -> a - b;
```

#### Function Shapes
- built-in in `java.util.function`
- Consumer: takes one param of specified type, returns nothing
```java
Consumer<Integer> printer = i -> System.out.println(i);
```
- Function: takes one param of specified type, returns the specified type
```java
Function<Integer, Integer> twice = i -> i * 2;
```
- Supplier: takes no parameters, returns the specified type
```java
Supplier<Long> currentTime = () -> Instant.now().toEpochMilli();
```
- Predicate: takes one param of specified type, returns boolean
```java
Predicate<Integer> greaterThanTen = num -> num > 10;
```

### Method Reference
- shorthand for special type of Lambda expressions
- Opt/Alt + Enter in IntelliJ can convert a lambda expression to a method reference
```java
// Lambda
n -> Adder.addOneTo(n);
// Method Reference
Adder:addOneTo;

// Lambda
Adder adder = new Adder(10);
n -> adder.add(n);
// Method Reference
adder::add;

// Lambda
n -> new Adder(n);
// Method Reference
Adder::new;
```

### Optionals
- immutable container used to contain a non-null value of any Object type
- benefits: avoids NPEs and `if (x == null)`
```java
Optional<Integer> v1 = Optional.of(5); // Optional[5]
Optional<Integer> v2 = Optional.empty(); // empty Optional
Optional<Integer> v3 = Optional.of(null); // throws NullPointerException
Optional<Integer> v4 = Optional.ofNullable(null); // empty Optional
Optional<Integer> v5 = Optional.ofNullable(x); // nullable -> Optional
```
- Example
```java
class UserData {
  public String name;
  public String product;
  public Optional<String> email;
}

interface UserStore {
  // Return Optional with user if userId is known, otherwise Optional.empty()
  Optional<UserData> lookup(final String userId);
}

class InMemoryUserStore implements UserStore {
  public InMemoryUserStore(Map<String, UserData> users) {
    this.users = users;
  }
  public Optional<UserData> lookup(String userId) {
    return Optional.ofNullable(users.get(userId));
  }
}

users.lookup(userId)
  .map(user -> user.name)
  .orElse("");

users.lookup(userId)
  .flatMap(user -> user.email)
  .orElse(null);
```
- Unwrapping an Optional
  - `get()`: returns the contents if present, otherwise throws an exception should be avoided, anti-pattern
  - `orElse(fallbackValue)`
  - `orElseGet(() -> fallbackComputation())`: requires a Supplier
  - `orElseThrow(() -> generateException())`
  - `isPresent()`: using optional imperatively, better option - `ifPresent(v -> useValue(v))`

### Stream
- a sequence of objects which supports various methods taht can be pipelined together to get the desired result
- not a data structure, but can take input from different data structures
- immutable
- benefit: avoids explicit for/while-loops
<img src="https://1.bp.blogspot.com/-XEU2WqWiI4g/XZc3e0v8djI/AAAAAAAAAhg/WTdc1dqVwiUAmizN-abuvSNRWuYSy_UrQCEwYBhgL/s1600/Ska%25CC%2588rmavbild%2B2019-10-03%2Bkl.%2B09.42.17.png" height="300px">

```java
list.stream()                     // generator
  .map(e -> e.name())             // operation: not applied straight awai
  .filter(n -> !n.isEmpty())      // operation
  .findFirst()                    // terminator, returns Optional<T>
```
- Generators: used to feed data into the streams
  - `Stream.empty()`: Empty stream
  - `Stream.of(1,2,3)`: Stream of three elements
  - `list.stream()`: Stream from a List, can be used on any Java Collection
  - `map.entrySet().stream()`: Stream from the entries in a map
- A stream only be consumed once
```java
Stream<Integer> nStream = Stream.of(1,2,3);
nStream.map(x -> x * 2).forEach(System.out::println);
nStream.map(x -> x * 3).forEach(System.out::println); // throws IllegalSstateException (nStream is closed)
```
- Collectors: `toList()`, `toSet()`, `toCollection()`
```java
numbers.collect(Collectors.toCollection(LinkedList::new));
```
- `Collectors.toMap(keyMapper, valueMapper, mergeFunction)`: `mergeFunction`: is used if there are conflicting keys in the map.
```java
numbers
  .collect(Collectors.toMap(
    Function.identity(),
    i -> i * 100,
    (oldV, newV) -> newV       // use new value in case of conflict
  ));
```
- `groupingBy(keyExtractor)`: groups a stream into a Map based on a given function
```java
Map<String, List<Employee>> byDept
  = employees.stream()
      .collect(Collectors.groupingBy(Employee::getDepartment));
```

## Asynchronous Programming
- Pros: Allows multiple things to happen at the same time. It doesn't block the execution flow
<img src="https://www.baeldung.com/wp-content/uploads/sites/4/2020/07/sync.png" height="300px">

- Service-Oriented Architecture (SOA): each service will have to talk to other systems to reach its objective
- Each request is handled on a separate "thread"
- Cons: Codes get convoluted. Shared state and mutable objects should be avoided/used in a "thread-safe" manner

## Thread
- A unit of execution within a process
- Each thread runs a path of execution through the program
- When starting an application, execution begins on the Main Trhead
- Additional trheads may be spun off to execute code in parallel to the main thread
  - e.g. The main thread gets a list of files on the file system, then spins off 10 threads to each process 1/10th of the files
- The program will exit when the Main Thread commpletes execution
  - Must explicitly tell the Main Thread to wait for the completion of the "child" threads
<img src="https://miro.medium.com/max/1838/0*FOD_35FfbwsXnnx6.png" height="300px">

```java
class MyCustomThread extends Thread {
  @Override
  public void run() {
    for (int i = 0; i < 10; i++) {
      System.out.println("From Thread: " + i);
     }
    }
  }
}

MyCustomThread thread = new MyCustomThread();

thread.start();
```

```java
class Printer extends Thread {
  private int id;
  public Printer(int id) { this.id = id; }
  
  @Override
  public void run() {
    for (int i = 1; i <= 3; i++) {
      System.out.println("[Thread " + id + "]: " + i);
    }
  }
}

Printer printer1 = new Printer(1);
Printer printer2 = new Printer(2);
printer1.start();
printer2.start();

printer1.join();
printer2.join();
```
- Thread vvs Runnable
  - Implementing Runnable interface is generally considered to be better
  - Runnable favors composition over inheritance
  - Runnable allows execution to be handled by an ExecutorService
  - Java doesn't support multiple inheritance. Runnable alllos to extend fro ma superclass besides Thread

### Thread Pool
- a way to reuse previously created threads in order to execute tasks
- certain predefined limits are established (e.g. number of threads) prior to the creation of the pool
```java
// Threadpool with 1 Thread
Executor e1 = Executors.newSingleThreadExecutor()
Executor e2 = Executors.newFixedThreadPool(10)
```

### ExecutorService
- An interface designed to allow you to easily submit tasks which need to be executed
- Contains methods for **controlling the progress** of the tasks which are submitted to it
- Allows a Runnable or a Callable to be submitted and executed asyncchronously on a Threadpool
- Maintains a blocking queue which can take requests and execute them as resources become available
<img src="https://www.baeldung.com/wp-content/uploads/2016/08/2016-08-10_10-16-52-1024x572.png" height="300px">

```java
ExecutorService ex = Executors.newSingleThreadExecutor();
ex.submit(() -> {
  for (int i = 0; i < 10; i++) {
    System.out.println("From Runnable: " + i);
  }
});
```

## Callable & Future
- Runnable interface cannot produce a result
- Callable interface allows you to execute async code, but also provides utilities to return a result
- The result returned by Callable is a Future
- It can also throw exceptions fro the block of computation
<img src="https://static.javatpoint.com/core/images/callable-and-future-in-java.png" height="300px">

### Future
- Represents the result of an async computation
- Provides methods to retrieve result from an async computation and also to check its status
- Methods offered by the Future Interface: `get()`, `get(timeout, timeUnit)`, `isDone()`, `cancel()`, `isCancelled()`
  - `get()`: try to avoid using it. Waits indefinitely for the result of the async computation
  - `get(timeout, timeUnit)`: similar to `get()`, but only waits for a specific time
```java
Callable<Integer> c = 
  () -> IntStream
          .range(1, 1000)
          .reduce()
          .getAsInt();
ExecutorService ex = Executors.newSingleThreadExecutor();
Future<Integer> f = ex.submit(c);

System.out.println("Is it done: " + f.isDone());  // could be True or False
System.out.println("Result: " + f.get());         // will wait for the result of the async computation
System.out.println("Is it done: " + f.isDone());  // will always be True
```
- Is old and busted
- CompletableFuture is new
- Won't deal with Future or Executors directly. Will instead interact with libraries returning them

### CompletableFuture
- Implements the Future and CompletionStage interfaces
- Allows you to create complex chains of async events
- Functions should return a CompletionStage (the interface)
- CompletableFuture class has utilities for easily returning objects through CompletionStage interface
- Chained CompletionStages allow us to implement series of async tasks which depend on the results of one another
- **thenApply**: takes the result of a CompletionStage, then returns its own result
```java
public CompletionStage<List<String>> getArtistGenres(final String artistId) {
  CompletionStage<Artist> artistFuture = metadataClient.getArtist(artistId);
  
  return artistFuture.thenApply(artistMetadata -> {
    reutrn artistMetadata.genres();
  });
}
```
- **thenComposeAsync**: takes the result of a CompletionStage, then returns another CompletionStage whcih gets unnestest into the parent
```java
public CompletionStage<Map<String, String>> getArtistGenresWithDetails(final String artistId) {
  CompletionStage<Artist> artistGenresFuture = getArtistGenres(artistId);
  
  return artistGenresFuture.thenComposeAsync(artistGenreList -> {
    // getGenreDetails returns CompletionStage<Map<String, String>>
    reutrn genreClient.getGenreDetails(artistGenreList);
  });
}
```
- **thenCompose**: chain CompletionStage within thenCompose
```java
public CompletionStage<Map<String, String>> getArtistGenresWithDetails(final String artistId) {
  CompletionStage<Artist> artistGenresFuture = getArtistGenres(artistId);
  
  return artistGenresFuture.thenCompose(artistGenreList -> {
    // getGenreDetails returns CompletionStage<Map<String, String>>
    return genreClient.getGenreDetails(artistGenreList)
            .thenApply(genreDetails -> {
                return genreDetails.entrySet().stream()
                          .map()
                          .collect();
            });
  });
}
```
- **thenCombine**: execute two CompletionStages in parallel and take action on the result of both futures when they complete
- **thenAccept**: consumes the result of a CompletionStage, but doesn't return anything
- **supplyAsync/completedFuture**: create a CompletableFuture from a synchronous object/task

  });
}
- **.exceptionally()**: Exception Handling
```java
  return artistFuture.thenApply()
          .exceptionally(e -> {
            LOG.error(e);
            return List.of();  // returns an empty list
          });
```
