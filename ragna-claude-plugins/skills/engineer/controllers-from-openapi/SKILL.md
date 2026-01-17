---
name: controllers-from-openapi
description: Generate Spring Boot REST controllers, DTOs, and validation from OpenAPI 3.x YAML specifications
allowed-tools: ["Read", "Write", "Edit", "Bash", "Glob"]
---

# Controllers from OpenAPI Skill

This skill generates production-ready Spring Boot REST controllers, request/response DTOs, and validation logic from OpenAPI 3.x YAML specifications, following contract-first API development principles.

## Purpose

Generate from OpenAPI spec:
- **Spring Boot REST Controllers** with proper annotations
- **Request DTOs** (Java Records) with validation constraints
- **Response DTOs** (Java Records)
- **Pagination support** for list endpoints
- **Security annotations** from OpenAPI security schemes
- **OpenAPI annotations** for Swagger UI integration

## When to Use

- After generating OpenAPI YAML with `openapi-from-architecture` skill
- During principal-engineer scaffolding phase
- When implementing contract-first API development
- When standardizing REST controller structure across services

## Workflow Integration

```
backend-architect          principal-engineer                principal-engineer
(API draft in arc42) → (openapi-from-architecture) → (controllers-from-openapi)
                           ↓                             ↓
                       openapi.yaml                  Controllers + DTOs
```

## Instructions

### Phase 1: Read OpenAPI Specification

1. **Locate OpenAPI file:**
   - Primary location: `src/main/resources/openapi.yaml`
   - Microservices: `src/main/resources/openapi/{service-name}-openapi.yaml`
   - Alternative: `src/main/resources/api/openapi.yaml`
   - Legacy location: `./openapi.yaml` (project root - for backwards compatibility)
   - Ask user if not found in standard locations

2. **Parse OpenAPI structure:**
   - Extract API info (title, version, description)
   - Extract all paths and operations
   - Extract all schema definitions
   - Extract security schemes
   - Extract tags for controller organization

3. **Identify architecture pattern:**
   - Check `info.x-architecture-pattern` or infer from project structure
   - Determine package structure based on pattern

### Phase 2: Generate Request DTOs

For each request schema in `components/schemas` that ends with `Request`:

**File location based on pattern:**
- **Hexagonal:** `infrastructure/adapter/in/web/dto/{SchemaName}.java`
- **Vertical Slice:** `feature/{resource}/dto/{SchemaName}.java`
- **DDD:** `{boundedcontext}/api/dto/{SchemaName}.java`
- **Layered:** `dto/{SchemaName}.java`

**DTO Template (Java Record):**

```java
package {package-based-on-pattern}.dto;

import jakarta.validation.constraints.*;
import java.time.Instant;
import java.time.LocalDate;
import java.util.List;

/**
 * Request DTO for {operation description}
 *
 * Generated from OpenAPI specification:
 * - Operation: {operationId}
 * - Schema: {schemaName}
 *
 * OpenAPI source: {openapi-file-path}
 */
public record {SchemaName}(
    {for each property in schema:}
    {validation-annotations based on schema constraints}
    {type} {propertyName}{if not last: ,}
) {}
```

**Validation Mapping:**

| OpenAPI Constraint | Jakarta Validation |
|-------------------|-------------------|
| `required: true` | `@NotNull` or `@NotBlank` (for strings) |
| `minLength` | `@Size(min = {value})` |
| `maxLength` | `@Size(max = {value})` |
| `minimum` | `@Min({value})` |
| `maximum` | `@Max({value})` |
| `pattern` | `@Pattern(regexp = "{value}")` |
| `format: email` | `@Email` |
| `format: date` | Type: `LocalDate` |
| `format: date-time` | Type: `Instant` |
| `enum` | Custom enum type |

**Example Request DTO:**

```java
package com.example.userservice.infrastructure.adapter.in.web.dto;

import jakarta.validation.constraints.*;

/**
 * Request DTO for creating a new user
 *
 * Generated from OpenAPI specification:
 * - Operation: createUser
 * - Schema: CreateUserRequest
 *
 * OpenAPI source: src/main/resources/openapi.yaml
 */
public record CreateUserRequest(
    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    @Size(max = 255, message = "Email must not exceed 255 characters")
    String email,

    @NotBlank(message = "First name is required")
    @Size(min = 1, max = 100, message = "First name must be between 1 and 100 characters")
    String firstName,

    @NotBlank(message = "Last name is required")
    @Size(min = 1, max = 100, message = "Last name must be between 1 and 100 characters")
    String lastName,

    @NotNull(message = "Role is required")
    UserRole role
) {}
```

### Phase 3: Generate Response DTOs

For each response schema in `components/schemas` that ends with `Dto` or is referenced in responses:

**Response DTO Template:**

```java
package {package-based-on-pattern}.dto;

import java.time.Instant;

/**
 * Response DTO for {resource}
 *
 * Generated from OpenAPI specification:
 * - Schema: {schemaName}
 *
 * OpenAPI source: {openapi-file-path}
 */
public record {SchemaName}(
    {for each property in schema:}
    {type} {propertyName}{if not last: ,}
) {}
```

**Example Response DTO:**

```java
package com.example.userservice.infrastructure.adapter.in.web.dto;

import java.time.Instant;

/**
 * Response DTO for User
 *
 * Generated from OpenAPI specification:
 * - Schema: UserDto
 *
 * OpenAPI source: src/main/resources/openapi.yaml
 */
public record UserDto(
    Long id,
    String email,
    String firstName,
    String lastName,
    UserRole role,
    Instant createdAt,
    Instant updatedAt
) {}
```

### Phase 4: Generate Enums

For each enum defined in OpenAPI schemas:

```java
package {package-based-on-pattern}.dto;

/**
 * {Enum description from OpenAPI}
 *
 * Generated from OpenAPI specification
 */
public enum {EnumName} {
    {VALUE1},
    {VALUE2},
    {VALUE3}
}
```

**Example:**

```java
package com.example.userservice.infrastructure.adapter.in.web.dto;

/**
 * User role enumeration
 *
 * Generated from OpenAPI specification
 */
public enum UserRole {
    USER,
    ADMIN,
    MODERATOR
}
```

### Phase 5: Generate Pagination DTOs

If OpenAPI spec contains paginated responses (schemas ending with `Page`):

```java
package {package-based-on-pattern}.dto;

import java.util.List;

/**
 * Paginated response for {Resource}
 *
 * Generated from OpenAPI specification:
 * - Schema: {ResourcePage}
 *
 * Follows Spring Data Page structure
 */
public record {Resource}Page(
    List<{Resource}Dto> content,
    int page,
    int size,
    long totalElements,
    int totalPages,
    boolean last
) {}
```

### Phase 6: Generate REST Controllers

For each tag in OpenAPI (or group by resource if no tags):

**Controller location based on pattern:**
- **Hexagonal:** `infrastructure/adapter/in/web/{Resource}Controller.java`
- **Vertical Slice:** `feature/{resource}/{Resource}Controller.java`
- **DDD:** `{boundedcontext}/api/{Resource}Controller.java`
- **Layered:** `controller/{Resource}Controller.java`

**Controller Template:**

```java
package {package-based-on-pattern};

import {dto-package}.*;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
import io.swagger.v3.oas.annotations.security.SecurityRequirement;
import jakarta.validation.Valid;
import jakarta.validation.constraints.Min;

/**
 * REST Controller for {Resource}
 *
 * Generated from OpenAPI specification:
 * - Tag: {tagName}
 * - Base path: {basePath}
 *
 * OpenAPI source: {openapi-file-path}
 *
 * Architecture Pattern: {pattern}
 *
 * TODO: Inject service layer for business logic
 * TODO: Implement exception handling (use @ControllerAdvice)
 * TODO: Add integration tests
 */
@RestController
@RequestMapping("{basePath}")
@RequiredArgsConstructor
@Slf4j
@Validated
@Tag(name = "{tagName}", description = "{tagDescription}")
public class {Resource}Controller {

    // TODO: Inject service via constructor
    // private final {Resource}Service {resource}Service;

    {for each operation in paths for this tag:}
    {generate-method-for-operation}
}
```

**Method Generation for Each Operation:**

```java
@{HttpMethod}Mapping{if path-params: ("/{id}")}
@Operation(
    summary = "{operation.summary}",
    description = "{operation.description}",
    operationId = "{operation.operationId}"
)
@ApiResponses({
    @ApiResponse(responseCode = "200", description = "{response.description}"),
    @ApiResponse(responseCode = "400", description = "Bad request"),
    @ApiResponse(responseCode = "401", description = "Unauthorized"),
    @ApiResponse(responseCode = "404", description = "Not found")
})
{if security required:}
@SecurityRequirement(name = "{securitySchemeName}")
public ResponseEntity<{ResponseType}> {operationId}(
    {for each parameter:}
    {generate-parameter-annotation} {paramType} {paramName}{if not last: ,}
) {
    log.info("{Log message with parameters}");

    // TODO: Implement business logic
    // {example service call}
    throw new UnsupportedOperationException("Not implemented yet - {operationId}");
}
```

**Parameter Generation:**

| Parameter Location | Spring Annotation | Example |
|-------------------|------------------|---------|
| `in: path` | `@PathVariable` | `@PathVariable Long id` |
| `in: query` | `@RequestParam` | `@RequestParam(defaultValue = "0") int page` |
| `in: header` | `@RequestHeader` | `@RequestHeader("X-API-Key") String apiKey` |
| `in: body` (requestBody) | `@Valid @RequestBody` | `@Valid @RequestBody CreateUserRequest request` |

**Complete Controller Example:**

```java
package com.example.userservice.infrastructure.adapter.in.web;

import com.example.userservice.infrastructure.adapter.in.web.dto.*;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
import io.swagger.v3.oas.annotations.security.SecurityRequirement;
import jakarta.validation.Valid;
import jakarta.validation.constraints.Min;

/**
 * REST Controller for User Management
 *
 * Generated from OpenAPI specification:
 * - Tag: Users
 * - Base path: /api/v1/users
 *
 * OpenAPI source: src/main/resources/openapi.yaml
 *
 * Architecture Pattern: Hexagonal
 *
 * TODO: Inject UserService for business logic
 * TODO: Implement exception handling (use @ControllerAdvice)
 * TODO: Add integration tests
 */
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
@Slf4j
@Validated
@Tag(name = "Users", description = "User management operations")
public class UserController {

    // TODO: Inject service via constructor
    // private final UserService userService;

    @GetMapping
    @Operation(
        summary = "List all users",
        description = "Returns paginated list of users with filtering and sorting support",
        operationId = "listUsers"
    )
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Successful operation"),
        @ApiResponse(responseCode = "400", description = "Invalid parameters"),
        @ApiResponse(responseCode = "401", description = "Unauthorized")
    })
    @SecurityRequirement(name = "bearerAuth")
    public ResponseEntity<UserPage> listUsers(
            @RequestParam(defaultValue = "0") @Min(0) int page,
            @RequestParam(defaultValue = "20") @Min(1) int size,
            @RequestParam(required = false) String sort) {
        log.info("Listing users - page: {}, size: {}, sort: {}", page, size, sort);

        // TODO: Implement business logic
        // UserPage result = userService.listUsers(page, size, sort);
        // return ResponseEntity.ok(result);
        throw new UnsupportedOperationException("Not implemented yet - listUsers");
    }

    @GetMapping("/{id}")
    @Operation(
        summary = "Get user by ID",
        description = "Returns a single user by its unique identifier",
        operationId = "getUserById"
    )
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Successful operation"),
        @ApiResponse(responseCode = "401", description = "Unauthorized"),
        @ApiResponse(responseCode = "404", description = "User not found")
    })
    @SecurityRequirement(name = "bearerAuth")
    public ResponseEntity<UserDto> getUserById(@PathVariable Long id) {
        log.info("Getting user with ID: {}", id);

        // TODO: Implement business logic
        // UserDto user = userService.getUserById(id);
        // return ResponseEntity.ok(user);
        throw new UnsupportedOperationException("Not implemented yet - getUserById");
    }

    @PostMapping
    @Operation(
        summary = "Create new user",
        description = "Creates a new user with the provided data",
        operationId = "createUser"
    )
    @ApiResponses({
        @ApiResponse(responseCode = "201", description = "User created successfully"),
        @ApiResponse(responseCode = "400", description = "Invalid request data"),
        @ApiResponse(responseCode = "401", description = "Unauthorized"),
        @ApiResponse(responseCode = "409", description = "User already exists")
    })
    @SecurityRequirement(name = "bearerAuth")
    public ResponseEntity<UserDto> createUser(@Valid @RequestBody CreateUserRequest request) {
        log.info("Creating user with email: {}", request.email());

        // TODO: Implement business logic
        // UserDto created = userService.createUser(request);
        // return ResponseEntity.status(HttpStatus.CREATED).body(created);
        throw new UnsupportedOperationException("Not implemented yet - createUser");
    }

    @PutMapping("/{id}")
    @Operation(
        summary = "Update user",
        description = "Updates an existing user with the provided data",
        operationId = "updateUser"
    )
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "User updated successfully"),
        @ApiResponse(responseCode = "400", description = "Invalid request data"),
        @ApiResponse(responseCode = "401", description = "Unauthorized"),
        @ApiResponse(responseCode = "404", description = "User not found")
    })
    @SecurityRequirement(name = "bearerAuth")
    public ResponseEntity<UserDto> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UpdateUserRequest request) {
        log.info("Updating user with ID: {}", id);

        // TODO: Implement business logic
        // UserDto updated = userService.updateUser(id, request);
        // return ResponseEntity.ok(updated);
        throw new UnsupportedOperationException("Not implemented yet - updateUser");
    }

    @DeleteMapping("/{id}")
    @Operation(
        summary = "Delete user",
        description = "Deletes a user by its unique identifier",
        operationId = "deleteUser"
    )
    @ApiResponses({
        @ApiResponse(responseCode = "204", description = "User deleted successfully"),
        @ApiResponse(responseCode = "401", description = "Unauthorized"),
        @ApiResponse(responseCode = "404", description = "User not found")
    })
    @SecurityRequirement(name = "bearerAuth")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        log.info("Deleting user with ID: {}", id);

        // TODO: Implement business logic
        // userService.deleteUser(id);
        // return ResponseEntity.noContent().build();
        throw new UnsupportedOperationException("Not implemented yet - deleteUser");
    }
}
```

### Phase 7: Generate Error Response DTO

If OpenAPI defines ErrorResponse schema:

```java
package {package-based-on-pattern}.dto;

import java.time.Instant;
import java.util.List;

/**
 * Standard error response structure
 *
 * Generated from OpenAPI specification:
 * - Schema: ErrorResponse
 *
 * Used by @ControllerAdvice for consistent error handling
 */
public record ErrorResponse(
    Instant timestamp,
    int status,
    String error,
    String message,
    String path,
    List<ValidationError> errors
) {}

/**
 * Validation error details for 400 Bad Request responses
 */
public record ValidationError(
    String field,
    String rejectedValue,
    String message
) {}
```

### Phase 8: Add Maven/Gradle Dependencies

Ensure `pom.xml` or `build.gradle` includes:

```xml
<!-- Spring Boot Web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Spring Boot Validation -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

<!-- OpenAPI/Swagger -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>

<!-- Lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

### Phase 9: Generate Controller Tests

For each controller, generate test skeleton:

**Test location:** `src/test/java/{package-path}/{Resource}ControllerTest.java`

```java
package {package-based-on-pattern};

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

/**
 * Test for {Resource}Controller
 *
 * Generated from OpenAPI specification
 *
 * TODO: Implement test cases for all endpoints
 * TODO: Add security context for authenticated tests
 * TODO: Test validation rules
 */
@WebMvcTest({Resource}Controller.class)
class {Resource}ControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    // TODO: Mock service dependencies
    // @MockBean
    // private {Resource}Service {resource}Service;

    @Test
    void list{Resources}_ShouldReturnPagedResults() throws Exception {
        // TODO: Implement test
        mockMvc.perform(get("{basePath}")
                .param("page", "0")
                .param("size", "20"))
                .andExpect(status().isOk());
    }

    @Test
    void get{Resource}ById_ShouldReturnResource_WhenExists() throws Exception {
        // TODO: Implement test
        mockMvc.perform(get("{basePath}/1"))
                .andExpect(status().isOk());
    }

    @Test
    void create{Resource}_ShouldReturnCreated_WhenValidRequest() throws Exception {
        // TODO: Implement test
        mockMvc.perform(post("{basePath}")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{}"))
                .andExpect(status().isCreated());
    }

    // TODO: Add more test methods for PUT, DELETE, error cases
}
```

## Type Mapping

Map OpenAPI data types to Java types:

| OpenAPI Type | OpenAPI Format | Java Type |
|--------------|----------------|-----------|
| `string` | - | `String` |
| `string` | `date` | `java.time.LocalDate` |
| `string` | `date-time` | `java.time.Instant` |
| `string` | `email` | `String` (@Email) |
| `string` | `uri` | `String` or `java.net.URI` |
| `string` | `uuid` | `String` or `java.util.UUID` |
| `integer` | `int32` | `Integer` |
| `integer` | `int64` | `Long` |
| `number` | `float` | `Float` |
| `number` | `double` | `Double` |
| `boolean` | - | `Boolean` |
| `array` | - | `List<T>` |
| `object` | - | Custom DTO |
| enum | - | Java `enum` |

## Naming Conventions

Follow Spring Boot and Java naming conventions:

| Component | Convention | Example |
|-----------|-----------|---------|
| **Controller Class** | `{Resource}Controller` | `UserController` |
| **Request DTO** | `{Action}{Resource}Request` | `CreateUserRequest` |
| **Response DTO** | `{Resource}Dto` | `UserDto` |
| **Page DTO** | `{Resource}Page` | `UserPage` |
| **Enum** | `{Name}` (PascalCase) | `UserRole` |
| **Method** | `{operationId}` from OpenAPI | `createUser`, `getUserById` |
| **Package (Hexagonal)** | `.infrastructure.adapter.in.web` | |
| **Package (Layered)** | `.controller`, `.dto` | |

## Security Integration

Based on OpenAPI security schemes, add Spring Security:

**JWT Bearer Token:**
```java
@SecurityRequirement(name = "bearerAuth")
// Add to controller or method level

// In application.yml or SecurityConfig
@PreAuthorize("hasRole('USER')")
```

**OAuth2:**
```java
@SecurityRequirement(name = "oauth2")

// In SecurityConfig
@EnableWebSecurity
@Configuration
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) {
        return http
            .oauth2ResourceServer(oauth2 -> oauth2.jwt())
            .build();
    }
}
```

## Best Practices

1. **Use Java Records for DTOs** (immutable, concise)
2. **Apply @Validated at controller level** for method-level validation
3. **Use @Valid for request bodies** to trigger validation
4. **Return ResponseEntity<T>** for flexible status codes
5. **Log all endpoint invocations** with key parameters
6. **Add OpenAPI annotations** for Swagger UI documentation
7. **Generate TODO comments** for implementation guidance
8. **Include example service calls** in comments
9. **Follow RESTful conventions** (proper HTTP methods and status codes)
10. **Keep controllers thin** - delegate to service layer

## Output Structure

```
src/
├── main/
│   └── java/
│       └── com/example/service/
│           ├── infrastructure/adapter/in/web/  (Hexagonal)
│           │   ├── UserController.java
│           │   ├── OrderController.java
│           │   └── dto/
│           │       ├── CreateUserRequest.java
│           │       ├── UpdateUserRequest.java
│           │       ├── UserDto.java
│           │       ├── UserPage.java
│           │       ├── UserRole.java
│           │       └── ErrorResponse.java
│           └── (or different structure for other patterns)
└── test/
    └── java/
        └── com/example/service/
            └── infrastructure/adapter/in/web/
                ├── UserControllerTest.java
                └── OrderControllerTest.java
```

## Checklist

Before completing this skill, verify:

- [ ] All DTOs are generated as Java Records
- [ ] All validation constraints from OpenAPI are applied
- [ ] All controllers have proper @RestController and @RequestMapping
- [ ] All methods have @Operation and @ApiResponses annotations
- [ ] All request bodies have @Valid annotation
- [ ] All path parameters use @PathVariable
- [ ] All query parameters use @RequestParam with defaults
- [ ] Security annotations match OpenAPI security requirements
- [ ] Logging statements are added to all methods
- [ ] TODO comments guide implementation
- [ ] Test skeletons are generated
- [ ] Package structure follows architecture pattern
- [ ] Response types match OpenAPI specification
- [ ] HTTP status codes match OpenAPI responses
- [ ] Enum types are generated for OpenAPI enums

## Integration with Spring Boot

Add to `application.yml`:

```yaml
springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
    operationsSorter: method
  show-actuator: false
```

Access Swagger UI at: `http://localhost:8080/swagger-ui.html`

---

Generated controllers follow Spring Boot best practices and are ready for service layer implementation.
