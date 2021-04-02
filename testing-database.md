Setup to connect database with @DataJpaTest
Disable flyway and add P6Spy proxy
```java
@DataJpaTest(properties = {
  "spring.flyway.enabled=false",
  "spring.jpa.hibernate.ddl-auto=create-drop",
  "spring.datasource.driver-class-name=com.p6spy.engine.spy.P6SpyDriver", // P6Spy
  "spring.datasource.url=jdbc:p6spy:h2:mem:testing;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=false" // P6Spy
})
//For DataJpaTest by default h2 is configured
//Disable the Inmemory database configuration
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)  
class ReviewRepositoryTest {
```

Test container setup for postgress

```java
@DataJpaTest
@Testcontainers(disabledWithoutDocker = true)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class ReviewRepositoryNoInMemoryTest {

  @Container // will manage the lifecycle, if removed we have to start and stop containers by ourselves
  static PostgreSQLContainer<?> container = new PostgreSQLContainer<>("postgres:12.3")
    .withDatabaseName("test")
    .withUsername("duke")
    .withPassword("s3cret");
    //.isReuse(true) - will not kill the container

  @DynamicPropertySource
  static void properties(DynamicPropertyRegistry registry) {
    registry.add("spring.datasource.url", container::getJdbcUrl);
    registry.add("spring.datasource.password", container::getPassword);
    registry.add("spring.datasource.username", container::getUsername);
  }

//to populate the data in database. Data will be removed after test executions
@Test
  @Sql(scripts = "/scripts/INIT_REVIEW_EACH_BOOK.sql")
```