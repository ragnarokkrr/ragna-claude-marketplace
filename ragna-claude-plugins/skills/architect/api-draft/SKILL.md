---
name: api-draft
description: Document REST, GraphQL, gRPC, and event-driven APIs with endpoints, channels, and message contracts
allowed-tools: ["Read", "Edit", "Write"]
---

# API Draft Documentation Skill

This skill generates comprehensive API documentation for both synchronous (REST, GraphQL, gRPC) and asynchronous (event-driven) APIs with clear contracts, examples, and integration guidance.

## Purpose

Create actionable API documentation covering:
- **REST APIs:** HTTP endpoints with methods, paths, parameters, responses
- **GraphQL APIs:** Queries, mutations, subscriptions with schema types
- **gRPC APIs:** Service definitions with RPC methods and message types
- **Async APIs:** Event channels with producers, consumers, and message schemas

## Instructions

When this skill is invoked:

1. **Determine API type:**
   - Synchronous: REST, GraphQL, or gRPC
   - Asynchronous: Event channels (Kafka, RabbitMQ, SQS, Pub/Sub)
   - Or both (common in modern architectures)

2. **Gather API information:**
   - **REST:** Resources, operations, parameters, status codes
   - **GraphQL:** Schema types, queries, mutations, subscriptions
   - **gRPC:** Services, RPCs, request/response messages
   - **Async:** Channels, events, producers, consumers, payloads

3. **Generate appropriate documentation:**
   - Use tables for structured information
   - Include examples and sample payloads
   - Document error responses
   - Show versioning and compatibility strategy

4. **Provide placement guidance:**
   - REST/GraphQL/gRPC → arc42 Section 5.2 or 5.3
   - Async APIs → arc42 Section 5.2 (Async API Draft subsection)

## Templates

### REST API Draft Table

Standard table format for HTTP endpoints:

| HTTP Method | Endpoint | Description | Request | Response | Status Codes |
|-------------|----------|-------------|---------|----------|--------------|
| GET | `/api/v1/users` | List all users with pagination | Query: `?page=1&limit=20` | `User[]` | 200, 400, 401 |
| GET | `/api/v1/users/{id}` | Get user by ID | Path: `id` (UUID) | `User` | 200, 404, 401 |
| POST | `/api/v1/users` | Create new user | Body: `CreateUserRequest` | `User` | 201, 400, 401, 409 |
| PUT | `/api/v1/users/{id}` | Update user | Path: `id`, Body: `UpdateUserRequest` | `User` | 200, 400, 401, 404 |
| PATCH | `/api/v1/users/{id}` | Partial update | Path: `id`, Body: `PatchUserRequest` | `User` | 200, 400, 401, 404 |
| DELETE | `/api/v1/users/{id}` | Delete user | Path: `id` | - | 204, 401, 404 |

**Request/Response Examples:**

```json
// POST /api/v1/users - Request
{
  "email": "user@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "role": "USER"
}

// POST /api/v1/users - Response (201 Created)
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "role": "USER",
  "createdAt": "2025-01-17T10:30:00Z",
  "updatedAt": "2025-01-17T10:30:00Z"
}

// Error Response (400 Bad Request)
{
  "error": "VALIDATION_ERROR",
  "message": "Email is required",
  "details": [
    {
      "field": "email",
      "message": "Must be a valid email address"
    }
  ]
}
```

### REST API Patterns

#### Pagination
```
GET /api/v1/resources?page=1&limit=20
GET /api/v1/resources?offset=40&limit=20
GET /api/v1/resources?cursor=eyJpZCI6MTIzfQ==
```

**Response:**
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrevious": false
  }
}
```

#### Filtering & Sorting
```
GET /api/v1/users?status=active&role=admin&sort=-createdAt
GET /api/v1/products?category=electronics&price[gte]=100&price[lte]=500
```

#### Versioning
- **URL versioning:** `/api/v1/users`, `/api/v2/users`
- **Header versioning:** `Accept: application/vnd.api+json; version=1`
- **Query parameter:** `/api/users?api-version=1`

### Async API Draft Table

Inspired by AsyncAPI specification:

| Channel | Producer | Message Types | Payload Name | Purpose | Known Consumers | Schema |
|---------|----------|---------------|--------------|---------|-----------------|--------|
| `orders.created` | order-service | OrderCreated | OrderCreatedEvent | Notify when order placed | notification-service, inventory-service, analytics-service | See below |
| `orders.updated` | order-service | OrderStatusChanged | OrderStatusEvent | Track order status changes | notification-service, analytics-service | See below |
| `orders.cancelled` | order-service | OrderCancelled | OrderCancelledEvent | Handle order cancellation | refund-service, inventory-service | See below |
| `inventory.reserved` | inventory-service | InventoryReserved | InventoryEvent | Confirm stock reservation | order-service | See below |
| `inventory.released` | inventory-service | InventoryReleased | InventoryEvent | Release reserved stock | analytics-service | See below |
| `payments.completed` | payment-service | PaymentCompleted | PaymentEvent | Payment processed successfully | order-service, billing-service | See below |
| `payments.failed` | payment-service | PaymentFailed | PaymentEvent | Payment processing failed | order-service, notification-service | See below |

**Event Schema Examples:**

```javascript
// Event: OrderCreated
// Channel: orders.created
{
  eventId: UUID,                    // unique event identifier
  eventType: "OrderCreated",        // event type discriminator
  timestamp: ISO8601,               // event occurrence time
  version: "1.0",                   // event schema version
  correlationId: UUID,              // request correlation
  payload: {
    orderId: UUID,
    orderNumber: String,
    userId: UUID,
    items: [{
      productId: UUID,
      quantity: Number,
      price: Decimal
    }],
    total: Decimal,
    status: "PENDING"
  },
  metadata: {
    source: "order-service",
    causationId: UUID,              // triggering event
    traceId: UUID                   // distributed tracing
  }
}

// Event: OrderStatusChanged
// Channel: orders.updated
{
  eventId: UUID,
  eventType: "OrderStatusChanged",
  timestamp: ISO8601,
  version: "1.0",
  correlationId: UUID,
  payload: {
    orderId: UUID,
    previousStatus: String,
    newStatus: String,              // CONFIRMED | SHIPPED | DELIVERED | CANCELLED
    reason: String,                 // optional
    updatedBy: UUID                 // user or system ID
  },
  metadata: {
    source: "order-service",
    causationId: UUID,
    traceId: UUID
  }
}

// Event: PaymentCompleted
// Channel: payments.completed
{
  eventId: UUID,
  eventType: "PaymentCompleted",
  timestamp: ISO8601,
  version: "1.0",
  correlationId: UUID,
  payload: {
    paymentId: UUID,
    orderId: UUID,
    amount: Decimal,
    currency: String,
    method: String,                 // CREDIT_CARD | PAYPAL | STRIPE
    transactionId: String,
    processedAt: ISO8601
  },
  metadata: {
    source: "payment-service",
    causationId: UUID,
    traceId: UUID
  }
}
```

### Async API Patterns

#### Event Envelope (Standard)
```javascript
{
  eventId: UUID,            // unique per event
  eventType: String,        // event discriminator
  timestamp: ISO8601,       // when event occurred
  version: String,          // schema version (semver)
  correlationId: UUID,      // request correlation
  payload: Object,          // domain-specific data
  metadata: {
    source: String,         // producing service
    causationId: UUID,      // triggering event ID
    traceId: UUID           // distributed tracing
  }
}
```

#### Idempotency
- Include `eventId` for deduplication
- Consumers track processed event IDs
- Use TTL for cleanup (Redis, DynamoDB)

#### Schema Evolution
- Use semantic versioning: `1.0`, `1.1`, `2.0`
- **Minor versions:** Backward compatible (add optional fields)
- **Major versions:** Breaking changes (new channel or dual publishing)

#### Dead Letter Handling
- Failed messages → DLQ (Dead Letter Queue)
- Retry with exponential backoff
- Alert on DLQ threshold

### GraphQL API Draft

**Schema Definition:**
```graphql
type Query {
  users(filter: UserFilter, page: Int, limit: Int): UserConnection!
  user(id: ID!): User
  orders(userId: ID, status: OrderStatus): [Order!]!
}

type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
  updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!
  deleteUser(id: ID!): DeleteUserPayload!
}

type Subscription {
  orderStatusChanged(orderId: ID!): Order!
  userUpdated(userId: ID!): User!
}

type User {
  id: ID!
  email: String!
  firstName: String!
  lastName: String!
  role: UserRole!
  orders: [Order!]!
  createdAt: DateTime!
  updatedAt: DateTime!
}

input CreateUserInput {
  email: String!
  firstName: String!
  lastName: String!
  role: UserRole = USER
}

enum UserRole {
  USER
  ADMIN
  MODERATOR
}
```

### gRPC API Draft

**Service Definition (proto3):**
```protobuf
syntax = "proto3";

package user.v1;

service UserService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
  rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
  rpc CreateUser(CreateUserRequest) returns (CreateUserResponse);
  rpc UpdateUser(UpdateUserRequest) returns (UpdateUserResponse);
  rpc DeleteUser(DeleteUserRequest) returns (DeleteUserResponse);
  rpc WatchUser(WatchUserRequest) returns (stream UserEvent);  // server streaming
}

message User {
  string id = 1;
  string email = 2;
  string first_name = 3;
  string last_name = 4;
  UserRole role = 5;
  google.protobuf.Timestamp created_at = 6;
  google.protobuf.Timestamp updated_at = 7;
}

message GetUserRequest {
  string id = 1;
}

message GetUserResponse {
  User user = 1;
}

enum UserRole {
  USER_ROLE_UNSPECIFIED = 0;
  USER_ROLE_USER = 1;
  USER_ROLE_ADMIN = 2;
  USER_ROLE_MODERATOR = 3;
}
```

## API Design Best Practices

### REST
- **Use nouns for resources:** `/users`, `/orders`, not `/getUsers`, `/createOrder`
- **HTTP verbs for operations:** GET (read), POST (create), PUT (replace), PATCH (update), DELETE
- **Plural nouns:** `/users/123`, not `/user/123`
- **Nested resources:** `/users/123/orders` for relationships
- **Consistent naming:** camelCase or snake_case, pick one
- **Version APIs:** `/api/v1/` to allow evolution
- **Filter, sort, paginate:** Query parameters for list operations
- **Use status codes correctly:** 2xx success, 4xx client error, 5xx server error

### GraphQL
- **Nullable by default:** Use `!` for required fields
- **Input types for mutations:** Separate `CreateUserInput` from `User` type
- **Connection pattern for pagination:** `UserConnection { edges, pageInfo }`
- **Error handling:** Return errors in `errors` array, not in payload
- **N+1 problem:** Use DataLoader for batching
- **Subscriptions for real-time:** WebSocket-based event streaming

### gRPC
- **Versioned packages:** `user.v1`, `user.v2` for breaking changes
- **Request/Response pairs:** Each RPC has dedicated messages
- **Enums start at 0:** UNSPECIFIED value for default
- **Timestamps:** Use `google.protobuf.Timestamp`
- **Streaming:** Server/client/bidirectional for real-time
- **Error details:** Use `google.rpc.Status` with details

### Async APIs
- **Event versioning:** Include version in event envelope
- **Idempotency:** Use eventId for deduplication
- **Correlation:** Link related events with correlationId
- **Dead letter queues:** Handle failures gracefully
- **Schema registry:** Validate events (Avro, Protobuf, JSON Schema)
- **Consumer groups:** Scale consumers horizontally

## Usage Examples

**Example 1: REST API for order management**
```
User: Document REST endpoints for order CRUD operations
Skill: [Generates table with GET/POST/PUT/DELETE endpoints, request/response examples, status codes]
```

**Example 2: Event-driven order pipeline**
```
User: Document async events for order lifecycle
Skill: [Creates Async API table with order.created, order.confirmed, order.shipped channels and consumers]
```

**Example 3: GraphQL subscription for real-time updates**
```
User: GraphQL schema for user management with real-time updates
Skill: [Generates schema with queries, mutations, and subscriptions including UserConnection for pagination]
```

**Example 4: gRPC service for user management**
```
User: gRPC service definition for user CRUD and streaming
Skill: [Creates proto3 definition with RPCs and server streaming for watch operations]
```

## Placement in arc42 Documentation

| API Type | Section | Purpose |
|----------|---------|---------|
| REST/GraphQL/gRPC | 5.2 or 5.3 (API Draft subsection) | Synchronous API contracts |
| Async APIs | 5.2 (Async API Draft subsection) | Event channels and messages |
| API Gateway/BFF | 5.2 Container View | API aggregation and routing |
| Message formats | 8.1 Domain Concepts | Shared schemas and types |

## Error Handling

- If API type unclear: Ask whether REST, GraphQL, gRPC, or event-driven
- If operations unclear: Request list of resources and actions
- If event flows unclear: Ask about producers and consumers
- If versioning strategy needed: Clarify compatibility requirements

## Output Format

Always provide:
1. Structured tables (REST, Async API)
2. Schema definitions (GraphQL, gRPC)
3. Request/response examples with realistic data
4. Error handling patterns
5. Versioning and compatibility strategy
6. Recommended placement in arc42 documentation
7. Integration guidance for consumers

## Cross-References

- **C4 diagrams (c4-diagram skill):** Show API boundaries in Container view
- **Sequence diagrams (sequence-diagram skill):** Illustrate API call flows
- **MongoDB schemas (mongodb-schema skill):** Link API models to data storage
- **ADRs (create-adr skill):** Document API design decisions
