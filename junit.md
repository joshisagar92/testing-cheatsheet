# testing-cheatsheet

###JUnit
DisplayName annotation is used for writing a test definition without method constraints
```java
@DisplayName("Should fail when review contains 'lorem ipsum'")
```
Following test will run for each line in csv and each line is passed as review argument to 
test case
```java
  @ParameterizedTest
  @CsvFileSource(resources = "/badReview.csv")
  void test(String review) {
```

We can create an extentions which will provides data to the test case based on the configuration
in extension

```java
@ExtendWith(RandomReviewParameterResolverExtension.class)
class ReviewVerifierTest {
    @RepeatedTest(5)
    void shouldFailWhenRandomReviewQualityIsBad(@RandomReview String review) {
    
    }
}
public class RandomReviewParameterResolverExtension implements ParameterResolver {

  private static final List<String> badReviews = List.of("This book was shit I don't like it",
    "I was reading the book and I think the book is okay. I have read better books and I think I know what's good",
    "Good book with good agenda and good example. I can recommend for everyone");

  @Retention(RetentionPolicy.RUNTIME)
  @Target(PARAMETER)
  public @interface RandomReview {
  }

  @Override
  public boolean supportsParameter(ParameterContext parameterContext, ExtensionContext extensionContext) throws ParameterResolutionException {
    return parameterContext.isAnnotated(RandomReview.class);
  }

  @Override
  public Object resolveParameter(ParameterContext parameterContext, ExtensionContext extensionContext) throws ParameterResolutionException {
    return badReviews.get(ThreadLocalRandom.current().nextInt(0, badReviews.size()));
  }
}
```
Some Hamcrest matchers
```java
MatcherAssert.assertThat("ReviewVerifier did not pass a good review", result, Matchers.equalTo(true));
MatcherAssert.assertThat("Lorem ipsum", Matchers.endsWith("ipsum"));
MatcherAssert.assertThat(List.of(1, 2, 3, 4, 5), Matchers.hasSize(5));
MatcherAssert.assertThat(List.of(1, 2, 3, 4, 5), Matchers.anyOf(Matchers.hasSize(5), Matchers.emptyIterable()));
```
Some Junit matchers
```java
 Assertions.assertThat(result)
      .withFailMessage("ReviewVerifier did not pass a good review")
      .isEqualTo(true)
      .isTrue();

    Assertions.assertThat(List.of(1, 2, 3, 4, 5)).hasSizeBetween(1, 10);
    Assertions.assertThat(List.of(1, 2, 3, 4, 5)).contains(3).isNotEmpty();
```
JSON assertion
```java
  String result = "{\"name\": \"duke\", \"age\":\"42\", \"hobbies\": [\"soccer\", \"java\"]}";
  JSONAssert.assertEquals("{\"name\": \"duke\"}", result, false);
  JSONAssert.assertEquals("{\"hobbies\": [\"java\", \"soccer\"]}", result, false);
```
JSON assertion with JSONPath
```java
    String result = "{\"age\":\"42\", \"name\": \"duke\", \"tags\":[\"java\", \"jdk\"], \"orders\": [42, 42, 16]}";

    Assertions.assertEquals(2, JsonPath.parse(result).read("$.tags.length()", Long.class));
    Assertions.assertEquals("duke", JsonPath.parse(result).read("$.name", String.class));
    Assertions.assertEquals(100, JsonPath.parse(result).read("$.orders.sum()", Long.class));
```