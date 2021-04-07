RestClientTest annotation used for this.
```java
@RestClientTest(OpenLibraryRestTemplateApiClient.class)

//mock server will be created as a part of RestClientTest
@Autowired
  private MockRestServiceServer mockRestServiceServer;

 this.mockRestServiceServer
      .expect(requestTo(Matchers.containsString(ISBN)))
      // .expect(MockRestRequestMatchers.requestTo("/api/books?jscmd=data&format=json&bibkeys=ISBN:" + ISBN))
      .andRespond(withSuccess(new ClassPathResource("/stubs/openlibrary/success-" + ISBN + ".json")
        //classpath resource is a json file
        , MediaType.APPLICATION_JSON));

    Book result = cut.fetchMetadataForBook(ISBN);


 String response = """
       {
        "ISBN:9780596004651": {
          "publishers": [
            {
              "name": "O'Reilly"
            }
          ],
          "title": "Head second Java",
          "authors": [
            {
              "url": "https://openlibrary.org/authors/OL1400543A/Kathy_Sierra",
              "name": "Kathy Sierra"
            }
          ],
          "number_of_pages": 42,
          "cover": {
            "small": "https://covers.openlibrary.org/b/id/388761-S.jpg",
            "large": "https://covers.openlibrary.org/b/id/388761-L.jpg",
            "medium": "https://covers.openlibrary.org/b/id/388761-M.jpg"
          }
         }
       }
      """;

    this.mockRestServiceServer
      .expect(requestTo(Matchers.containsString("/api/books")))
      .andRespond(withSuccess(response, MediaType.APPLICATION_JSON));

    this.mockRestServiceServer
      .expect(requestTo(Matchers.containsString("/duke/42")))
      .andRespond(withSuccess(response, MediaType.APPLICATION_JSON));

    assertThrows(HttpServerErrorException.class, () -> {
          this.mockRestServiceServer
            .expect(requestTo("/api/books?jscmd=data&format=json&bibkeys=ISBN:" + ISBN))
            .andRespond(MockRestResponseCreators.withServerError());
    
          cut.fetchMetadataForBook(ISBN);
        });

    this.mockRestServiceServer
          .expect(requestTo("/api/books?jscmd=data&format=json&bibkeys=ISBN:" + ISBN))
          .andExpect(MockRestRequestMatchers.header("X-Custom-Auth", "Duke42"))
          .andExpect(MockRestRequestMatchers.header("X-Customer-Id", "42"))
          .andRespond(withSuccess(new ClassPathResource("/stubs/openlibrary/success-" + ISBN + ".json")
            , MediaType.APPLICATION_JSON));
```


```java
@ExtendWith(MockitoExtension.class) //Junit extension
class ReviewServiceTest {

@Mock
  private ReviewVerifier mockedReviewVerifier;

  @Mock
  private UserService userService;

  @Mock
  private BookRepository bookRepository;

  @Mock
  private ReviewRepository reviewRepository;

  @InjectMocks
  private ReviewService cut; // Inject mock will inject all required mocks
}

 assertThrows(IllegalArgumentException.class,
      () -> cut.createBookReview(ISBN, null, USERNAME, EMAIL));
```