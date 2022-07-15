# Testing
- [Flavors of Testing](#flavors-of-testing)
- [Test Driven Development (TDD)](#test-driven-development-tdd)
- [Testing Private Methods (Java)](#testing-private-methods-java)
- [JUnit5 Framework](#junit5-framework)
- [Assertions](#assertions)
- [Mocking](#mocking)
- [Integration testing](#integration-testing)

## Flavors of Testing
- Unit Testing: small, tightly-focussed tests of small things
- Integration testing: checking multiple modules working together
- End-to-End Testing: does the whole thing work?
- Regression Testing: something was working before still works? ensures an application still functions as expected after any code changes, updates, or improvements
- Black-Box Testing: testing a module/assembly without reference to its internal structure

## Test Driven Development (TDD)
_Write unit tests first before writting production code_
- Write a unit test that fails
- Write production code that makes the test pass

## Testing Private Methods (Java)
- test code can't call private methods
- `@VisibleForTesting`: to make the method public

## JUnit5 Framework
- A class
- Some instance methods marked @Test

```java
public class Calculator {
  public int add(int a, int b) {
    return a + b;
  }
}

public class CalculatorTest {
  private final Calculator calculator = new Calculator();
  
  @Test
  public void testSimpleAddition() {
    assertEquals(2, calculator.add(1,1));
  }
}
```
- Annotations
  - **@Test**: saying this is a test method
  - **@Timeout**: fail the test after too long. For example: **@Timeout(1)** == timed out after 1 second.
  - **@Tag**: run tests with a given tag: business test, model test, etc.
  - **@RepeatedTest**: run the test N times (in case of flakiness)
  - **@ParameterizedTest**: run the test for a list of several strings/numbers etc.
    - **@ValueSource**: supply strings/classes, etc.
    ```java
    @ParameterizedTest
    @ValueSource(strings = {
      "",
      "tatatat",
      "Madam"
    })
    void yesPalindrome(String s) {
      assertTrue(isPalindrome(s));
    }
    ```
- Lifecycle Methods
  <img src="https://www.lambdatest.com/blog/wp-content/uploads/2021/08/Per-Class-Lifecycle.png" height="300px">
  
  - @BeforeAll: run once, before the first test is executed
  - @BeforeEach: run once per test, before each test
  - @AfterEach
  - @AfterAll

## Assertions
- `assertTrue`, `assertFalse`
- `assertEquals`
- `assertNotNull`
- `assertArraysEquals`
- `assertTimeout`


## Mocking
- Primarily used in unit testing
- To isolate the behariov of the object, replace the other objects by mocks to simulate the behavior of real objects

```java
import static org.mockito.Mockito.mock;

public class TranslationGuardianTest {
  private Supportedlanguages supportedLanguages;
  private TranslationGuardian translationGuardian;

  public void setup() {
    supportedLanguages = Mockito.mock(SupportedLanguages.class);
    translationGuardian = new TranslationGuardian(supportedLanguages);
  }
}
```

### Recording
```java
import static org.mockito.Mockito.when;
when(supportedLanguages.isSupported("pl")).thenReturn(true);
when(supportedLanguages.getLanguages()).thenReturn(List.of("sv", "en", "de"));
```

### Verification
```java
// checks that the fallbackToDefaultTranslation was called once
verify(serviceMetrics).fallbackToDefaultTranslation();
// checks that the bakePizza was called three times
verify(serviceMetrics, times(3)).bakePizza();
// checks that there are no more interactions
verifyNoMoreInteractions(serviceMetrics);
```

### Example
```java
import static org.mockito.Mockito.mock;
public class MissingTranslationGuardianTest {
  private Supportedlanguages supportedLanguages = mock(SupportedLanguages.class);
  private ServiceMetrics serviceMetrics = mock(ServiceMetrics.class);
  private LocaleHelper localeHelper = mock(LocaleHelper.class);
  
  @Test
  public void shouldMarkKeyAsMissingForSupportedLanguages() {
    MissingTranslationGuardian missingTranslationGuardian = 
      new MissingTranslationGuardian(supportedLanguages, serviceMetrics, localeHelper);
      
    when(supportedLanguages.isSupported("pl")).thenReturn(true);
    
    missingTranslationGuardian.markIfTranslationIsMissing("pl", "dumplings");
    
    verify(serviceMetrics).markFallbackToDefaultTranslation();
    verifyZeroInteractions(localeHelper);
  }
}
```

### Spy
- A wrapper around the real object and allows to verify interactions with object instance or to mock specific behaviors
```java
import static org.mockito.Mockito.spy;
private StubClient stubClient = spy(new StubClient());
when(stubClient.sentRequests()).thenCallRealMethod();
```

### ArgumentCaptor
```java
private ArgumentCaptor<Request> requestCaptor = ArgumentCaptor.forClass(Request.class);

@Test
public void testRemoteCall() {
  Client client = new Client();
  when(client.request(requestCaptor.capture()))
    .thenReturn(completedFuture(future))
```

## Integration Testing
- Collect modules together and test them as a subsystem in order to verify they collaborate as intended to achieve some larger piece of behavior
- In contrast to unit tests

## Integration test of gRPC Application
### Step 2. Set up the apolloContainer
which will be used to make the requests against the service

```java
private static final ApolloContainer SERVICE =
  ApolloContainer.imageFromBuild().withExposedPorts(5990);
```

### Step 3. Start the apolloContainer
```java
@BeforeAll   // starting a container is expensive and will take a long time. Just wanna start once
static void setup() {
  SERVICE.start();
}
```

### Step 4. Create the InProcess channel
```java
final ManagedChannel channel = 
  ManagedChannelBuilder.forTarget(
    SERVICE.getExternalEndpoint(5990).toString())
  .userPlaintext()
  .build();
```
- is created based on the string passed to .forTarget
- important for that string to be unique per test case


