# Testing Spring Boot Applications

<details><summary>

## What is the difference between unit tests, slice tests and integration tests in Spring Boot?
</summary>

- **Unit tests:** plain JUnit 5 + Mockito, no Spring context. Fastest; test business logic.
- **Slice tests:** load only one layer: `@WebMvcTest` (controllers), `@DataJpaTest` (repositories), `@JsonTest`, `@RestClientTest`.
- **Integration tests:** `@SpringBootTest` loads the whole context; combine with **Testcontainers** for real MySQL/Kafka/Redis.

Aim for many unit tests, fewer slice tests, and a small number of end-to-end integration tests.
</details>

<details><summary>

## Write a unit test with Mockito.
</summary>

```java
@ExtendWith(MockitoExtension.class)
class CompanyServiceTest {

    @Mock CompanyRepository repo;
    @InjectMocks CompanyService service;

    @Test
    void returnsCompanyWhenFound() {
        when(repo.findById(1L)).thenReturn(Optional.of(new Company(1L, "Tracxn")));

        CompanyDto dto = service.findById(1L);

        assertThat(dto.name()).isEqualTo("Tracxn");
        verify(repo).findById(1L);
    }

    @Test
    void throwsWhenMissing() {
        when(repo.findById(99L)).thenReturn(Optional.empty());
        assertThatThrownBy(() -> service.findById(99L))
            .isInstanceOf(ResourceNotFoundException.class);
    }
}
```
</details>

<details><summary>

## How do you test a REST controller with @WebMvcTest?
</summary>

```java
@WebMvcTest(CompanyController.class)
class CompanyControllerTest {

    @Autowired MockMvc mvc;
    @MockitoBean CompanyService service;     // Boot 3.4+; earlier versions use @MockBean

    @Test
    void getCompany() throws Exception {
        when(service.findById(1L)).thenReturn(new CompanyDto(1L, "Tracxn"));

        mvc.perform(get("/api/v1/companies/1"))
           .andExpect(status().isOk())
           .andExpect(jsonPath("$.name").value("Tracxn"));
    }

    @Test
    void rejectsInvalidBody() throws Exception {
        mvc.perform(post("/api/v1/companies")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"name\":\"\"}"))
           .andExpect(status().isBadRequest());
    }
}
```
</details>

<details><summary>

## How do you test repositories and use Testcontainers?
</summary>

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class CompanyRepositoryTest {

    @Container
    @ServiceConnection                          // Boot 3.1+: wires the datasource automatically
    static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.0");

    @Autowired CompanyRepository repo;

    @Test
    void findsBySector() {
        repo.save(new Company(null, "Acme", "fintech"));
        assertThat(repo.findBySector("fintech")).hasSize(1);
    }
}
```

Testing against the real database catches dialect-specific SQL, index and constraint issues that an H2 in-memory database hides.
</details>

<details><summary>

## @Mock vs @MockBean (@MockitoBean) vs @Spy?
</summary>

- `@Mock` (Mockito): a mock in a plain unit test; no Spring involved.
- `@MockBean` / `@MockitoBean` (Spring): replaces a bean **inside the Spring context** with a mock. Each different set of mock beans creates a new context, which slows test suites.
- `@Spy`: wraps a real object; real methods run unless stubbed.
</details>

<details><summary>

## How do you test Kafka consumers?
</summary>

Use `@EmbeddedKafka` (spring-kafka-test) or a Kafka Testcontainer; publish a message and assert the side effect with Awaitility, since consumption is asynchronous.

```java
@SpringBootTest
@EmbeddedKafka(partitions = 1, topics = "call-events")
class TranscriptionConsumerIT {
    @Autowired KafkaTemplate<String, CallEvent> kafka;
    @Autowired TranscriptRepository transcripts;

    @Test
    void processesEvent() {
        kafka.send("call-events", "call-1", new CallEvent("call-1"));
        await().atMost(Duration.ofSeconds(10))
               .untilAsserted(() -> assertThat(transcripts.findByCallId("call-1")).isPresent());
    }
}
```
</details>
