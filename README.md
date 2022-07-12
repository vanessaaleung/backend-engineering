# Backend Engineering
- [Monolith vs Microservices Architecture](#monolith-vs-microservices-architecture)
- [Service to Service communication](#service-to-service-communication)
- [Scaling](#scaling)
- [Monitoring](#monitoring)
- [Functional Programming](#functional-programming)

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

