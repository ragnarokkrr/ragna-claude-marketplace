---
name: spring-testing-setup
description: Generate Spring Boot testing framework with JUnit 5, Mockito, Testcontainers, and coverage setup
allowed-tools: ["Read", "Edit", "Write"]
---

# Spring Boot Testing Setup Skill

This skill generates comprehensive testing infrastructure for Spring Boot applications including unit tests, integration tests, slice tests, Testcontainers, contract testing, and code coverage configuration.

## Purpose

Establish production-grade testing framework:
- **Test pyramid structure** (unit, integration, e2e)
- **JUnit 5** with modern assertions (AssertJ)
- **Mockito** for mocking dependencies
- **Testcontainers** for real database/message broker tests
- **Slice tests** (@WebMvcTest, @DataJpaTest)
- **Spring Cloud Contract** for API contract testing
- **JaCoCo** for code coverage (>85% target)
- **Arch Unit** for architecture testing

## Testing Templates

### Base Integration Test Class

```java
package com.example.integration;

import org.junit.jupiter.api.BeforeAll;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
@Testcontainers
public abstract class IntegrationTestBase {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test")
        .withReuse(true);  // Reuse container across tests

    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
        .withExposedPorts(6379)
        .withReuse(true);

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
        registry.add("spring.data.redis.host", redis::getHost);
        registry.add("spring.data.redis.port", () -> redis.getMappedPort(6379));
    }

    @BeforeAll
    static void beforeAll() {
        System.setProperty("spring.flyway.enabled", "true");
    }
}
```

### Controller Slice Test

```java
@WebMvcTest(ResourceController.class)
@Import(SecurityConfig.class)
class ResourceControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ResourceService resourceService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @WithMockUser(roles = "USER")
    void shouldGetResourceById() throws Exception {
        ResourceDto resource = new ResourceDto(1L, "Test", "Description", ResourceStatus.ACTIVE);
        when(resourceService.findById(1L)).thenReturn(Optional.of(resource));

        mockMvc.perform(get("/api/v1/resources/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.name").value("Test"));

        verify(resourceService).findById(1L);
    }

    @Test
    @WithMockUser(roles = "ADMIN")
    void shouldCreateResource() throws Exception {
        CreateResourceRequest request = new CreateResourceRequest("New", "Desc");
        ResourceDto created = new ResourceDto(1L, "New", "Desc", ResourceStatus.ACTIVE);

        when(resourceService.create(any())).thenReturn(created);

        mockMvc.perform(post("/api/v1/resources")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"));
    }
}
```

### Repository Test

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class ResourceRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine");

    @DynamicPropertySource
    static void properties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private ResourceRepository repository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void shouldSaveAndFindResource() {
        Resource resource = Resource.builder()
            .name("Test")
            .description("Description")
            .status(ResourceStatus.ACTIVE)
            .build();

        Resource saved = repository.save(resource);
        entityManager.flush();
        entityManager.clear();

        Optional<Resource> found = repository.findById(saved.getId());

        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("Test");
    }
}
```

### Service Unit Test

```java
@ExtendWith(MockitoExtension.class)
class ResourceServiceTest {

    @Mock
    private ResourceRepository repository;

    @Mock
    private EventPublisher eventPublisher;

    @InjectMocks
    private ResourceService service;

    @Test
    void shouldCreateResource() {
        CreateResourceRequest request = new CreateResourceRequest("Name", "Desc");
        Resource entity = Resource.builder()
            .id(1L)
            .name("Name")
            .description("Desc")
            .status(ResourceStatus.ACTIVE)
            .build();

        when(repository.save(any(Resource.class))).thenReturn(entity);

        ResourceDto result = service.create(request);

        assertThat(result.id()).isEqualTo(1L);
        assertThat(result.name()).isEqualTo("Name");

        verify(repository).save(any(Resource.class));
        verify(eventPublisher).publish(any(ResourceCreatedEvent.class));
    }
}
```

## Maven Configuration

```xml
<dependencies>
    <!-- Testing -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <!-- Testcontainers -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>testcontainers</artifactId>
        <version>1.19.3</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>postgresql</artifactId>
        <version>1.19.3</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>1.19.3</version>
        <scope>test</scope>
    </dependency>

    <!-- AssertJ -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.24.2</version>
        <scope>test</scope>
    </dependency>

    <!-- Spring Cloud Contract -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-contract-verifier</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- ArchUnit -->
    <dependency>
        <groupId>com.tngtech.archunit</groupId>
        <artifactId>archunit-junit5</artifactId>
        <version>1.2.1</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <!-- JaCoCo for code coverage -->
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.11</version>
            <executions>
                <execution>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
                <execution>
                    <id>report</id>
                    <phase>test</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                </execution>
                <execution>
                    <id>check</id>
                    <goals>
                        <goal>check</goal>
                    </goals>
                    <configuration>
                        <rules>
                            <rule>
                                <element>BUNDLE</element>
                                <limits>
                                    <limit>
                                        <counter>INSTRUCTION</counter>
                                        <value>COVEREDRATIO</value>
                                        <minimum>0.85</minimum>
                                    </limit>
                                </limits>
                            </rule>
                        </rules>
                    </configuration>
                </execution>
            </executions>
        </plugin>

        <!-- Spring Cloud Contract Plugin -->
        <plugin>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-contract-maven-plugin</artifactId>
            <extensions>true</extensions>
            <configuration>
                <testFramework>JUNIT5</testFramework>
                <baseClassForTests>com.example.integration.ContractTestBase</baseClassForTests>
            </configuration>
        </plugin>

        <!-- Surefire for unit tests -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.3</version>
            <configuration>
                <includes>
                    <include>**/*Test.java</include>
                    <include>**/*Tests.java</include>
                </includes>
            </configuration>
        </plugin>

        <!-- Failsafe for integration tests -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-failsafe-plugin</artifactId>
            <version>3.2.3</version>
            <executions>
                <execution>
                    <goals>
                        <goal>integration-test</goal>
                        <goal>verify</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## Test Application Properties

```yaml
# src/test/resources/application-test.yml
spring:
  datasource:
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
  flyway:
    enabled: false
  cache:
    type: simple

logging:
  level:
    root: WARN
    com.example: DEBUG
```

## Best Practices

- ✓ Use `@SpringBootTest` sparingly (slow), prefer slice tests
- ✓ Test repositories with real database (Testcontainers)
- ✓ Mock external dependencies in unit tests
- ✓ Use AssertJ for fluent assertions
- ✓ Aim for >85% code coverage
- ✓ Name tests clearly: `should[Expected]When[Condition]`
- ✓ Use `@Nested` for grouping related tests
- ✓ Leverage parameterized tests with `@ParameterizedTest`
