---
name: sequence-diagram
description: Generate UML sequence diagrams for runtime behavior and interaction flows using Mermaid
allowed-tools: ["Read", "Edit", "Write"]
---

# UML Sequence Diagram Skill

This skill generates UML sequence diagrams using Mermaid to document runtime behavior, interaction flows, and message exchanges between system components.

## Purpose

Visualize dynamic behavior showing:
- **Message flows:** How components interact over time
- **Order of operations:** Sequential and parallel execution
- **Participants:** Actors, services, databases, external systems
- **Control flow:** Loops, conditions, error handling
- **Timing:** Synchronous vs asynchronous communication

## Instructions

When this skill is invoked:

1. **Identify the scenario:**
   - Happy path (primary success flow)
   - Error/failure scenario (degraded mode, retries)
   - Alternative flows (conditional branches)
   - Background/async processing

2. **Gather participants:**
   - External actors (users, systems)
   - Services/components
   - Databases
   - Message queues
   - External APIs

3. **Map interaction flow:**
   - Request/response pairs
   - Async messages (fire-and-forget)
   - Database queries
   - Event publications
   - Error conditions

4. **Generate Mermaid sequence diagram:**
   - Use clear participant names
   - Show message labels with protocols
   - Include activations (lifelines)
   - Add notes for important details
   - Use alt/opt/loop for control flow

5. **Provide placement guidance:**
   - Section 6.1: Happy-path scenario
   - Section 6.2: Failure/degraded scenario
   - Section 6.3: Background/batch scenario

## Mermaid Sequence Diagram Syntax

### Basic Structure

```mermaid
sequenceDiagram
    participant U as User
    participant W as Web App
    participant A as API Gateway
    participant S as Service
    participant D as Database

    U->>W: Click "Submit Order"
    W->>A: POST /api/v1/orders
    A->>S: Forward request
    S->>D: INSERT order
    D-->>S: Order ID
    S-->>A: 201 Created
    A-->>W: Order response
    W-->>U: Show confirmation
```

### Message Types

```mermaid
sequenceDiagram
    participant A as Service A
    participant B as Service B

    A->>B: Synchronous call (solid arrow, wait for response)
    A-->>B: Return value (dotted arrow, response)
    A-)B: Async message (open arrow, fire-and-forget)
    A-xB: Failed call (x marks failure)
```

### Activations (Lifelines)

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server

    C->>+S: Request (activate S)
    Note over S: Processing...
    S->>+S: Call self
    S-->>-S: Return
    S-->>-C: Response (deactivate S)
```

### Control Flow

```mermaid
sequenceDiagram
    participant U as User
    participant S as Service
    participant D as Database

    U->>S: Request data

    alt Valid request
        S->>D: Query
        D-->>S: Results
        S-->>U: 200 OK
    else Invalid request
        S-->>U: 400 Bad Request
    else Server error
        S->>D: Query
        D-xS: Timeout
        S-->>U: 500 Internal Error
    end

    opt Cache available
        S->>S: Read from cache
    end

    loop Retry up to 3 times
        S->>D: Query with backoff
        D-->>S: Response
    end
```

### Notes and Boxes

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Service

    Note over C: User clicks button
    C->>S: POST /api/users
    Note right of S: Validates input<br/>Checks duplicates

    rect rgb(200, 220, 250)
        Note over S: Transaction boundary
        S->>S: Create user
        S->>S: Send welcome email
    end

    S-->>C: 201 Created
```

### Parallel Execution

```mermaid
sequenceDiagram
    participant O as Order Service
    participant P as Payment Service
    participant I as Inventory Service
    participant N as Notification Service

    O->>P: Process payment
    par Payment and Inventory
        P->>P: Charge card
    and
        O->>I: Reserve items
        I->>I: Update stock
    end
    P-->>O: Payment confirmed
    I-->>O: Items reserved
    O-)N: Send order confirmation (async)
```

## Common Scenarios

### 1. Happy Path: Create Order

```mermaid
sequenceDiagram
    actor U as Customer
    participant W as Web UI
    participant A as API Gateway
    participant O as Order Service
    participant P as Payment Service
    participant I as Inventory Service
    participant DB as Database
    participant Q as Message Queue

    U->>+W: Submit order
    W->>+A: POST /api/v1/orders
    A->>+O: Create order

    O->>+DB: Begin transaction
    O->>DB: INSERT order
    DB-->>O: Order ID

    O->>+I: Reserve inventory
    I->>DB: UPDATE stock
    DB-->>I: Rows affected
    I-->>-O: Reserved

    O->>+P: Process payment
    P->>P: Charge credit card
    P-->>-O: Payment ID

    O->>DB: UPDATE order status=CONFIRMED
    O->>DB: Commit transaction
    DB-->>-O: Success

    O-)Q: Publish OrderCreated event

    O-->>-A: 201 Created
    A-->>-W: Order details
    W-->>-U: Show confirmation
```

### 2. Error Scenario: Payment Failure

```mermaid
sequenceDiagram
    participant O as Order Service
    participant P as Payment Service
    participant I as Inventory Service
    participant DB as Database

    O->>+DB: Begin transaction
    O->>DB: INSERT order
    O->>+I: Reserve inventory
    I-->>-O: Reserved

    O->>+P: Process payment
    P-xO: Payment declined

    Note over O: Rollback transaction

    O->>I: Release inventory
    I->>DB: UPDATE stock (restore)
    I-->>O: Released

    O->>DB: UPDATE order status=FAILED
    O->>DB: Commit
    DB-->>-O: Success

    O-->>O: Return error
```

### 3. Async Processing: Background Job

```mermaid
sequenceDiagram
    participant A as API
    participant Q as Job Queue
    participant W as Worker
    participant E as Email Service
    participant DB as Database

    A-)Q: Enqueue email job
    A-->>A: Return 202 Accepted

    Note over Q,W: Async processing

    Q->>+W: Dequeue job
    W->>+DB: Get user data
    DB-->>-W: User info

    W->>+E: Send email

    alt Email sent
        E-->>W: Success
        W->>DB: Mark job complete
    else Email failed
        E-xW: SMTP error
        W->>Q: Requeue with backoff
        Note over W: Retry in 5 minutes
    end

    W-->>-Q: ACK message
```

### 4. Saga Pattern: Distributed Transaction

```mermaid
sequenceDiagram
    participant O as Order Service
    participant P as Payment Service
    participant I as Inventory Service
    participant S as Shipping Service

    Note over O,S: Saga: Create Order

    O->>O: Create order (local tx)
    O->>+P: Reserve payment
    P-->>-O: Reserved

    O->>+I: Reserve inventory
    I-xO: Out of stock

    Note over O: Compensate

    O->>P: Release payment
    P-->>O: Released
    O->>O: Mark order failed
    O-->>O: Saga failed
```

### 5. Circuit Breaker Pattern

```mermaid
sequenceDiagram
    participant S as Service
    participant CB as Circuit Breaker
    participant E as External API

    loop Healthy
        S->>+CB: Call external API
        CB->>+E: Forward request
        E-->>-CB: Response
        CB-->>-S: Response
    end

    Note over CB: Failures exceed threshold

    CB->>CB: Open circuit

    S->>+CB: Call external API
    CB-xS: Circuit open (fast fail)
    S->>S: Use fallback

    Note over CB: After cooldown period

    CB->>CB: Half-open (test)
    S->>+CB: Call external API
    CB->>+E: Test request
    E-->>-CB: Success
    CB-->>-S: Response
    CB->>CB: Close circuit (recovered)
```

## Best Practices

### Participant Naming
- **Use aliases:** `participant U as User` for readability
- **Show types:** `actor`, `participant`, `database`
- **Consistent naming:** Match C4 diagram element names

### Message Labels
- **Be specific:** "POST /orders" not "Create"
- **Include protocols:** "HTTPS", "AMQP", "gRPC", "JDBC"
- **Show status codes:** "200 OK", "404 Not Found"

### Control Flow
- **alt/else:** For error handling and branches
- **opt:** For optional steps (caching, logging)
- **loop:** For retries and iterations
- **par:** For parallel/concurrent operations

### Clarity
- **Focus on one scenario:** Don't mix happy path with errors
- **Limit participants:** 5-7 max for readability
- **Add notes:** Explain non-obvious logic
- **Use boxes:** Group related operations (transactions, bounded contexts)

### Performance Indicators
- **Async calls:** Use `-)`for fire-and-forget
- **Activations:** Show processing time with `+`/`-`
- **Failures:** Mark with `x` to show errors

## Placement in arc42 Documentation

| Scenario Type | Section | Purpose |
|---------------|---------|---------|
| Happy path | 6.1 Happy-Path Scenario | Primary success flow |
| Error handling | 6.2 Failure/Degraded Scenario | Error recovery, retries, fallbacks |
| Background jobs | 6.3 Background/Batch Scenario | Async processing, scheduled tasks |
| Alternative flows | 6.1 or 6.2 | Conditional branches, variations |

## Usage Examples

**Example 1: REST API flow**
```
User: Sequence diagram for user registration with email verification
Skill: [Generates diagram showing API call, database insert, async email job]
```

**Example 2: Microservices orchestration**
```
User: Show order creation with payment, inventory, and shipping services
Skill: [Creates diagram with parallel calls, transaction coordination, event publishing]
```

**Example 3: Error handling with retry**
```
User: Diagram showing retry logic for external API call with circuit breaker
Skill: [Illustrates loop for retries, alt for success/failure, circuit breaker state]
```

**Example 4: Event-driven saga**
```
User: Saga pattern for distributed transaction with compensations
Skill: [Shows choreography with events, compensation flows for failures]
```

## Common Patterns

### Request-Response (Synchronous)
```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    C->>+S: Request
    S-->>-C: Response
```

### Fire-and-Forget (Asynchronous)
```mermaid
sequenceDiagram
    participant S as Service
    participant Q as Queue
    S-)Q: Publish event
    S-->>S: Continue processing
```

### Request-Reply with Queue
```mermaid
sequenceDiagram
    participant C as Client
    participant RQ as Request Queue
    participant S as Server
    participant ReQ as Reply Queue

    C-)RQ: Send request
    RQ->>+S: Dequeue
    S->>S: Process
    S-)ReQ: Send reply
    S-->>-RQ: ACK
    ReQ->>C: Deliver reply
```

### Retry with Exponential Backoff
```mermaid
sequenceDiagram
    participant S as Service
    participant E as External API

    loop Retry up to 3 times
        S->>+E: Request
        alt Success
            E-->>-S: 200 OK
        else Failure
            E-x-S: Timeout
            Note over S: Wait 2^attempt seconds
        end
    end
```

## Error Handling

- If scenario unclear: Ask for specific flow (happy path, error, alternative)
- If participants unclear: Request list of systems/components involved
- If too complex: Suggest breaking into multiple diagrams
- If timing matters: Use notes to indicate durations or SLAs

## Output Format

Always provide:
1. Complete Mermaid sequence diagram code block
2. Scenario description (what flow is shown)
3. Key interactions explained
4. Error handling strategy (if applicable)
5. Recommended placement in arc42 Section 6
6. Related diagrams to consider (state diagram, C4 component)

## Cross-References

- **C4 diagrams (c4-diagram skill):** Sequence diagrams show runtime view of containers/components
- **State diagrams (state-diagram skill):** Complement with lifecycle state transitions
- **API documentation (api-draft skill):** Sequence diagrams illustrate API usage
- **ADRs (create-adr skill):** Document decisions about interaction patterns
