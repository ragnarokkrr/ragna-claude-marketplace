---
name: spring-virtual-threads
description: Generate JDK 21+ virtual threads, structured concurrency, records, sealed types, and pattern matching code for Spring Boot
allowed-tools: ["Read", "Edit", "Write"]
---

# Spring Boot Virtual Threads & JDK 21+ Features Skill

This skill generates production-ready code leveraging JDK 21+ features including virtual threads, structured concurrency, scoped values, records, sealed types, and pattern matching in Spring Boot applications.

## Purpose

Enable modern Java features in Spring Boot:
- **Virtual Threads** (Project Loom) for scalable I/O
- **Structured Concurrency** for coordinated parallel operations
- **Scoped Values** for thread-local data
- **Records** for immutable DTOs
- **Sealed Types** for domain modeling
- **Pattern Matching** for type-safe conditionals
- **Sequenced Collections** for ordered data

## Instructions

When this skill is invoked:

1. **Determine feature requirements:**
   - Ask which JDK 21+ features are needed
   - Identify use case: REST API, async processing, domain modeling
   - Confirm Spring Boot version (3.2+ required)

2. **Generate appropriate code:**
   - Virtual threads configuration for Tomcat/Undertow
   - Structured concurrency examples for parallel operations
   - Record-based DTOs with validation
   - Sealed type hierarchies for domain models
   - Pattern matching for request handling

3. **Provide integration guidance:**
   - Configuration class placement
   - Bean definitions
   - Integration with Spring components

## Virtual Threads Configuration

### Basic Virtual Thread Setup

**Java Configuration Class:**
```java
package com.example.config;

import org.springframework.boot.web.embedded.tomcat.TomcatProtocolHandlerCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.web.client.RestClient;

import java.util.concurrent.Executor;
import java.util.concurrent.Executors;

@Configuration
@EnableAsync
public class VirtualThreadConfig {

    /**
     * Configure Tomcat to use virtual threads for request handling.
     * This allows Spring Boot to handle 10,000+ concurrent requests efficiently.
     */
    @Bean
    public TomcatProtocolHandlerCustomizer<?> protocolHandlerVirtualThreadExecutorCustomizer() {
        return protocolHandler -> {
            protocolHandler.setExecutor(Executors.newVirtualThreadPerTaskExecutor());
        };
    }

    /**
     * Task executor for @Async methods using virtual threads.
     * Replaces traditional thread pools with lightweight virtual threads.
     */
    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        return Executors.newVirtualThreadPerTaskExecutor();
    }

    /**
     * RestClient configured to use virtual threads.
     * Each HTTP call runs on a virtual thread, allowing massive concurrency.
     */
    @Bean
    public RestClient restClient() {
        return RestClient.builder()
            .requestFactory(new JdkClientHttpRequestFactory())
            .build();
    }
}
```

### YAML Configuration for Virtual Threads

```yaml
# application.yml
spring:
  threads:
    virtual:
      enabled: true  # Enable virtual threads for Spring MVC (Boot 3.2+)

server:
  tomcat:
    threads:
      max: 200  # Still useful as upper bound, but virtual threads scale beyond this
      min-spare: 10
```

## Structured Concurrency

### Parallel API Calls with Structured Concurrency

```java
package com.example.service;

import org.springframework.stereotype.Service;
import java.util.concurrent.StructuredTaskScope;
import java.util.concurrent.StructuredTaskScope.Subtask;

@Service
public class OrderService {

    private final UserService userService;
    private final ProductService productService;
    private final PaymentService paymentService;

    public OrderService(UserService userService,
                       ProductService productService,
                       PaymentService paymentService) {
        this.userService = userService;
        this.productService = productService;
        this.paymentService = paymentService;
    }

    /**
     * Fetch order details by calling multiple services in parallel.
     * Uses structured concurrency to ensure all subtasks complete or all fail together.
     */
    public OrderDetails getOrderDetails(Long orderId) throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {

            // Launch parallel tasks - each runs on its own virtual thread
            Subtask<User> userTask = scope.fork(() ->
                userService.getUserById(orderId));

            Subtask<Product> productTask = scope.fork(() ->
                productService.getProductById(orderId));

            Subtask<Payment> paymentTask = scope.fork(() ->
                paymentService.getPaymentStatus(orderId));

            // Wait for all tasks to complete or fail
            scope.join()
                 .throwIfFailed();  // Propagate any exception

            // All tasks succeeded - collect results
            return new OrderDetails(
                userTask.get(),
                productTask.get(),
                paymentTask.get()
            );
        }
    }

    /**
     * Process order with timeout and cancellation.
     * If any task fails or timeout occurs, all tasks are cancelled.
     */
    public ProcessResult processOrder(OrderRequest request) throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {

            Subtask<ValidationResult> validation = scope.fork(() ->
                validateOrder(request));

            Subtask<ReservationResult> reservation = scope.fork(() ->
                reserveInventory(request));

            Subtask<PaymentResult> payment = scope.fork(() ->
                processPayment(request));

            // Wait up to 5 seconds
            scope.joinUntil(Instant.now().plusSeconds(5))
                 .throwIfFailed();

            return new ProcessResult(
                validation.get(),
                reservation.get(),
                payment.get()
            );
        }
    }
}
```

## Records for DTOs

### Immutable Request/Response DTOs

```java
package com.example.dto;

import jakarta.validation.constraints.*;
import java.time.Instant;
import java.util.Objects;

/**
 * Resource DTO using record for immutability and conciseness.
 * Includes compact constructor for validation and derived methods.
 */
public record ResourceDto(
    Long id,
    @NotBlank String name,
    @Size(max = 1000) String description,
    ResourceStatus status,
    Instant createdAt,
    Instant updatedAt
) {
    // Compact constructor for additional validation
    public ResourceDto {
        Objects.requireNonNull(name, "name cannot be null");
        if (name != null && name.isBlank()) {
            throw new IllegalArgumentException("name cannot be blank");
        }
    }

    // Derived properties as instance methods
    public boolean isActive() {
        return status == ResourceStatus.ACTIVE;
    }

    public long ageInSeconds() {
        return Instant.now().getEpochSecond() - createdAt.getEpochSecond();
    }
}

/**
 * Create request with builder-like with methods.
 */
public record CreateResourceRequest(
    @NotBlank String name,
    @Size(max = 1000) String description,
    ResourceStatus status
) {
    public CreateResourceRequest {
        // Defaults
        if (status == null) {
            status = ResourceStatus.ACTIVE;
        }
    }

    // Builder-like methods
    public CreateResourceRequest withName(String name) {
        return new CreateResourceRequest(name, this.description, this.status);
    }
}

/**
 * Update request - nullable fields indicate optional updates.
 */
public record UpdateResourceRequest(
    String name,  // nullable = optional update
    String description,
    ResourceStatus status
) {
    public boolean hasNameUpdate() {
        return name != null;
    }
}

/**
 * Paginated response using record.
 */
public record PagedResponse<T>(
    List<T> items,
    int page,
    int size,
    long totalElements,
    int totalPages
) {
    public boolean hasNext() {
        return page < totalPages - 1;
    }

    public boolean hasPrevious() {
        return page > 0;
    }
}
```

## Sealed Types for Domain Modeling

### Type-Safe Domain Status Hierarchy

```java
package com.example.domain;

/**
 * Sealed interface for resource status - compiler knows all implementations.
 * Enables exhaustive pattern matching and type safety.
 */
public sealed interface ResourceStatus
    permits ActiveStatus, InactiveStatus, ArchivedStatus, PendingStatus {

    String displayName();
    boolean canTransitionTo(ResourceStatus newStatus);
    boolean isTerminal();
}

public record ActiveStatus() implements ResourceStatus {
    @Override
    public String displayName() { return "Active"; }

    @Override
    public boolean canTransitionTo(ResourceStatus newStatus) {
        return newStatus instanceof InactiveStatus ||
               newStatus instanceof ArchivedStatus;
    }

    @Override
    public boolean isTerminal() { return false; }
}

public record InactiveStatus() implements ResourceStatus {
    @Override
    public String displayName() { return "Inactive"; }

    @Override
    public boolean canTransitionTo(ResourceStatus newStatus) {
        return newStatus instanceof ActiveStatus ||
               newStatus instanceof ArchivedStatus;
    }

    @Override
    public boolean isTerminal() { return false; }
}

public record PendingStatus(String reason) implements ResourceStatus {
    @Override
    public String displayName() { return "Pending: " + reason; }

    @Override
    public boolean canTransitionTo(ResourceStatus newStatus) {
        return newStatus instanceof ActiveStatus ||
               newStatus instanceof ArchivedStatus;
    }

    @Override
    public boolean isTerminal() { return false; }
}

public record ArchivedStatus(Instant archivedAt) implements ResourceStatus {
    @Override
    public String displayName() { return "Archived"; }

    @Override
    public boolean canTransitionTo(ResourceStatus newStatus) {
        return false; // Terminal state
    }

    @Override
    public boolean isTerminal() { return true; }
}
```

### Sealed Type for API Responses

```java
package com.example.api;

/**
 * Sealed API response type - forces explicit handling of success/error cases.
 */
public sealed interface ApiResponse<T>
    permits SuccessResponse, ErrorResponse, ValidationErrorResponse {

    int statusCode();
    Instant timestamp();
}

public record SuccessResponse<T>(
    T data,
    int statusCode,
    Instant timestamp
) implements ApiResponse<T> {
    public SuccessResponse(T data) {
        this(data, 200, Instant.now());
    }
}

public record ErrorResponse<T>(
    String message,
    String errorCode,
    int statusCode,
    Instant timestamp
) implements ApiResponse<T> {
    public ErrorResponse(String message, int statusCode) {
        this(message, "INTERNAL_ERROR", statusCode, Instant.now());
    }
}

public record ValidationErrorResponse<T>(
    Map<String, String> fieldErrors,
    int statusCode,
    Instant timestamp
) implements ApiResponse<T> {
    public ValidationErrorResponse(Map<String, String> fieldErrors) {
        this(fieldErrors, 400, Instant.now());
    }
}
```

## Pattern Matching

### Type-Safe Request Processing

```java
package com.example.controller;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/resources")
public class ResourceController {

    private final ResourceService resourceService;

    public ResourceController(ResourceService resourceService) {
        this.resourceService = resourceService;
    }

    /**
     * Pattern matching with records for request handling.
     */
    @PostMapping
    public ResponseEntity<ApiResponse<ResourceDto>> createResource(
            @RequestBody CreateResourceRequest request) {

        ApiResponse<ResourceDto> response = resourceService.createResource(request);

        // Pattern matching on sealed type - exhaustive
        return switch (response) {
            case SuccessResponse<ResourceDto>(var data, var code, var ts) ->
                ResponseEntity.status(code).body(response);

            case ValidationErrorResponse<ResourceDto>(var errors, var code, var ts) ->
                ResponseEntity.status(code).body(response);

            case ErrorResponse<ResourceDto>(var msg, var errCode, var code, var ts) ->
                ResponseEntity.status(code).body(response);
        };
    }

    /**
     * Pattern matching for status transitions.
     */
    @PatchMapping("/{id}/status")
    public ResponseEntity<String> updateStatus(
            @PathVariable Long id,
            @RequestBody ResourceStatus newStatus) {

        Resource resource = resourceService.findById(id);

        // Pattern matching with guards
        String message = switch (resource.status()) {
            case ActiveStatus() when newStatus instanceof InactiveStatus ->
                "Deactivating resource";

            case ActiveStatus() when newStatus instanceof ArchivedStatus ->
                "Archiving active resource";

            case InactiveStatus() when newStatus instanceof ActiveStatus ->
                "Reactivating resource";

            case ArchivedStatus() ->
                "Cannot change archived resource status";

            case PendingStatus(String reason) when newStatus instanceof ActiveStatus ->
                "Approving pending resource: " + reason;

            default ->
                "Invalid status transition";
        };

        if (!message.startsWith("Invalid") && !message.startsWith("Cannot")) {
            resourceService.updateStatus(id, newStatus);
        }

        return ResponseEntity.ok(message);
    }
}
```

### Pattern Matching in Service Layer

```java
package com.example.service;

import org.springframework.stereotype.Service;

@Service
public class ResourceProcessor {

    /**
     * Process resources based on type with pattern matching.
     */
    public String processResource(Resource resource) {
        // Record pattern matching with deconstruction
        return switch (resource) {
            case Resource(var id, var name, ActiveStatus _, _, _) when id > 1000 ->
                processVipResource(name);

            case Resource(_, var name, ActiveStatus _, _, _) ->
                processStandardResource(name);

            case Resource(_, var name, InactiveStatus _, _, _) ->
                "Skipping inactive: " + name;

            case Resource(_, var name, ArchivedStatus(var archivedAt), _, _) ->
                "Archived on " + archivedAt + ": " + name;

            case Resource(_, var name, PendingStatus(var reason), _, _) ->
                "Pending (" + reason + "): " + name;

            case null -> throw new IllegalArgumentException("Null resource");
        };
    }

    /**
     * Event handling with exhaustive pattern matching.
     */
    public void handleEvent(DomainEvent event) {
        switch (event) {
            case ResourceCreatedEvent(Resource resource) -> {
                log.info("Resource created: {}", resource.name());
                notificationService.notifyCreation(resource);
            }

            case ResourceUpdatedEvent(Resource oldRes, Resource newRes) -> {
                log.info("Resource updated: {} -> {}", oldRes.name(), newRes.name());
                auditService.logUpdate(oldRes, newRes);
            }

            case ResourceDeletedEvent(Long id, String deletedBy) -> {
                log.info("Resource {} deleted by {}", id, deletedBy);
                cleanupService.cleanupResource(id);
            }

            case ResourceArchivedEvent(Resource resource, Instant archivedAt) -> {
                log.info("Resource archived: {}", resource.name());
                archiveService.moveToArchive(resource);
            }
        }
    }
}
```

## Scoped Values (Alternative to ThreadLocal)

```java
package com.example.context;

import java.util.concurrent.StructuredTaskScope;

/**
 * Scoped values provide structured, immutable thread-local data.
 * Better than ThreadLocal - automatically cleaned up, can't leak.
 */
public class RequestContext {

    // Define scoped values
    public static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();
    public static final ScopedValue<String> USER_ID = ScopedValue.newInstance();
    public static final ScopedValue<String> TENANT_ID = ScopedValue.newInstance();

    /**
     * Execute operation with scoped context.
     */
    public static <T> T withContext(String requestId, String userId,
                                    String tenantId, Supplier<T> operation) {
        return ScopedValue.where(REQUEST_ID, requestId)
                         .where(USER_ID, userId)
                         .where(TENANT_ID, tenantId)
                         .call(operation);
    }

    /**
     * Access scoped values in any method.
     */
    public static String getRequestId() {
        return REQUEST_ID.orElseThrow(() ->
            new IllegalStateException("No request context"));
    }

    public static String getUserId() {
        return USER_ID.orElseThrow(() ->
            new IllegalStateException("No user context"));
    }
}

/**
 * Filter to establish request context.
 */
@Component
public class RequestContextFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                        FilterChain chain) throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;

        String requestId = UUID.randomUUID().toString();
        String userId = extractUserId(httpRequest);
        String tenantId = extractTenantId(httpRequest);

        // Establish scoped context for this request
        ScopedValue.where(RequestContext.REQUEST_ID, requestId)
                  .where(RequestContext.USER_ID, userId)
                  .where(RequestContext.TENANT_ID, tenantId)
                  .run(() -> {
                      try {
                          chain.doFilter(request, response);
                      } catch (Exception e) {
                          throw new RuntimeException(e);
                      }
                  });
    }
}
```

## Complete Example: Modern Spring Boot Service

```java
package com.example.service;

import org.springframework.stereotype.Service;
import java.util.concurrent.StructuredTaskScope;
import java.util.concurrent.StructuredTaskScope.Subtask;

@Service
public class ModernOrderService {

    // Using records for DTOs
    public record OrderRequest(
        @NotNull Long userId,
        @NotEmpty List<OrderItem> items,
        PaymentMethod paymentMethod
    ) {}

    public record OrderItem(Long productId, int quantity, BigDecimal price) {}

    // Using sealed types for payment methods
    public sealed interface PaymentMethod
        permits CreditCard, DebitCard, PayPal {}

    public record CreditCard(String number, String cvv) implements PaymentMethod {}
    public record DebitCard(String number, String pin) implements PaymentMethod {}
    public record PayPal(String email) implements PaymentMethod {}

    /**
     * Process order using virtual threads + structured concurrency.
     */
    public ApiResponse<OrderDto> processOrder(OrderRequest request) {
        try {
            // Validate using pattern matching
            ValidationResult validation = validateOrder(request);
            if (!validation.isValid()) {
                return new ValidationErrorResponse<>(validation.errors());
            }

            // Execute parallel operations with structured concurrency
            try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {

                Subtask<UserInfo> userTask = scope.fork(() ->
                    userService.getUser(request.userId()));

                Subtask<List<ProductInfo>> productsTask = scope.fork(() ->
                    productService.getProducts(request.items()));

                Subtask<PaymentResult> paymentTask = scope.fork(() ->
                    processPayment(request.paymentMethod(), calculateTotal(request)));

                scope.join().throwIfFailed();

                // All succeeded - create order
                OrderDto order = createOrder(
                    userTask.get(),
                    productsTask.get(),
                    paymentTask.get()
                );

                return new SuccessResponse<>(order);
            }

        } catch (Exception e) {
            return new ErrorResponse<>(e.getMessage(), 500);
        }
    }

    /**
     * Payment processing with pattern matching on sealed types.
     */
    private PaymentResult processPayment(PaymentMethod method, BigDecimal amount) {
        return switch (method) {
            case CreditCard(String number, String cvv) ->
                creditCardProcessor.charge(number, cvv, amount);

            case DebitCard(String number, String pin) ->
                debitCardProcessor.charge(number, pin, amount);

            case PayPal(String email) ->
                paypalProcessor.charge(email, amount);
        };
    }
}
```

## Best Practices

### When to Use Virtual Threads
- ✓ I/O-bound operations (database calls, HTTP requests, file I/O)
- ✓ Blocking APIs that can't be made async
- ✓ High concurrency requirements (1000+ concurrent operations)
- ✗ CPU-bound operations (use traditional thread pools)
- ✗ Very short-lived operations (overhead not worth it)

### When to Use Structured Concurrency
- ✓ Multiple parallel operations that must all succeed
- ✓ Operations with timeouts or cancellation requirements
- ✓ Parent-child task relationships
- ✗ Fire-and-forget background tasks
- ✗ Independent operations that don't need coordination

### When to Use Records
- ✓ Immutable DTOs, request/response objects
- ✓ Value objects in domain model
- ✓ Configuration data
- ✗ Entities with identity (use @Entity classes)
- ✗ Objects requiring inheritance (use classes)

### When to Use Sealed Types
- ✓ Fixed set of subtypes (order status, payment method)
- ✓ Type-safe discriminated unions
- ✓ Domain modeling with exhaustive pattern matching
- ✗ Open extension points
- ✗ Plugin architectures

## Output Format

When invoked, provide:
1. Configuration class for virtual threads (if applicable)
2. Example service methods using structured concurrency
3. Record-based DTOs with validation
4. Sealed type hierarchies for domain modeling
5. Pattern matching examples for type-safe handling
6. Integration instructions and best practices
7. Placement guidance (package structure, file locations)
