WebMvcTest annotation with class name will required only classes added as dependency of
ReviewController. 
```java
@WebMvcTest(ReviewController.class) // read documentation of WebMvcTest
 @MockBean
 private ReviewService reviewService; //Used to mock the bean
```

Jackson object mapper
```java
 ArrayNode result = objectMapper.createArrayNode();

    ObjectNode statistic = objectMapper.createObjectNode();
    statistic.put("bookId", 1);
    statistic.put("isbn", "42");
    statistic.put("avg", 89.3);
    statistic.put("ratings", 2);

    result.add(statistic);

```

```java

@Autowired
private MockMvc mockMvc; // WebMvcTest will automatically autowire this

this.mockMvc
      .perform(get("/api/books/reviews"))
      .andExpect(status().isOk())
      .andExpect(jsonPath("$.size()", Matchers.is(1)));

//With unauthorised status
this.mockMvc
      .perform(get("/api/books/reviews/statistics"))
      .andExpect(status().isUnauthorized());
```
Use of JWT
```java
 @Test
    // @WithMockUser(username = "duke") // mocking the user with annotation
  void shouldReturnReviewStatisticsWhenUserIsAuthenticated() throws Exception {
    this.mockMvc
      .perform(get("/api/books/reviews/statistics")
        //.with(user("duke")))  
        //.with(httpBasic("duke", "password"))) // basic bearer token
        .with(jwt()))
      .andExpect(status().isOk());

    //happy path with jwt value
    String requestBody = """
            {
              "reviewTitle": "Great Java Book!",
              "reviewContent": "I really like this book!",
              "rating": 4
            }
          """;

    when(reviewService.createBookReview(eq("42"), any(BookReviewRequest.class),
      eq("duke"), endsWith("spring.io")))
      .thenReturn(84L);

    this
      .mockMvc
      .perform(post("/api/books/{isbn}/reviews", 42)
        .contentType(MediaType.APPLICATION_JSON)
        .content(requestBody)
        .with(jwt().jwt(builder -> builder
          .claim("email", "duke@spring.io")
          .claim("preferred_username", "duke"))))
      .andExpect(status().isCreated())
      .andExpect(header().exists("Location"))
      .andExpect(header().string("Location", Matchers.containsString("/books/42/reviews/84")));

// Bean validation
// most of the bean validation can be done at unit test level. we can test here custom validation
 String requestBody = """
        {
          "reviewContent": "I really like this book!",
          "rating": -1
        }
      """;

    this
      .mockMvc
      .perform(post("/api/books/{isbn}/reviews", 42)
        .contentType(MediaType.APPLICATION_JSON)
        .content(requestBody)
        .with(jwt().jwt(builder -> builder
          .claim("email", "duke@spring.io")
          .claim("preferred_username", "duke"))))
      .andExpect(status().isBadRequest())
      .andDo(MockMvcResultHandlers.print());

  //No interaction and forbidden
  this.mockMvc
      .perform(delete("/api/books/{isbn}/reviews/{reviewId}", 42, 3)
        .with(jwt()))
      .andExpect(status().isForbidden());

    verifyNoInteractions(reviewService);

    //Role based security
    @WithMockUser(roles = "moderator")
    void shouldAllowDeletingReviewsWhenUserIsAuthenticatedAndHasModeratorRole() throws Exception {
    this.mockMvc
      .perform(delete("/api/books/{isbn}/reviews/{reviewId}", 42, 3)
        // .with(jwt().authorities(new SimpleGrantedAuthority("ROLE_moderator")))
      )
      .andExpect(status().isOk());
    }
```