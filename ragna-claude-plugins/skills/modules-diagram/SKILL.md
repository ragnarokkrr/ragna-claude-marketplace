---
name: modules-diagram
description: Generate hierarchical module/package structure diagrams using Mermaid flowchart with nested subgraphs
allowed-tools: ["Read", "Edit", "Write"]
---

# Modules Hierarchy Diagram Skill

This skill generates hierarchical module and package structure diagrams using Mermaid flowcharts with nested `subgraph` blocks to visualize code organization, bounded contexts, and architectural layers.

## Purpose

Document code organization showing:
- **Package hierarchy:** Nested module/package structure (3-4 levels deep)
- **Bounded contexts:** DDD domain boundaries
- **Architectural layers:** Hexagonal ports/adapters, vertical slices, clean architecture
- **Module responsibilities:** What each package contains
- **Service decomposition:** Microservice internal structure

## Instructions

When this skill is invoked:

1. **Determine structure type:**
   - Monolith package structure
   - Microservice internal modules
   - Multi-service bounded contexts
   - Layered/hexagonal/vertical slice architecture

2. **Gather hierarchy information:**
   - Top-level domains or services
   - Sub-packages and modules
   - Layer boundaries (domain, application, infrastructure)
   - Naming conventions (lowercase dot notation)

3. **Generate Mermaid flowchart:**
   - Use nested `subgraph` blocks (no arrows!)
   - 3-4 levels deep for clarity
   - Lowercase names (e.g., `orders.payments.webhooks`)
   - Empty nodes as package placeholders

4. **Create description table:**
   - List each package with purpose
   - Use same lowercase identifiers
   - Brief, actionable descriptions

5. **Provide placement guidance:**
   - Section 5.2 or 5.3 (Building Block View - Modules subsection)

## Mermaid Flowchart Syntax

### Basic Nested Structure (No Arrows)

```mermaid
flowchart TB
    subgraph orders["orders"]
        subgraph orders.domain["orders.domain"]
            orders.domain.model["orders.domain.model"]
            orders.domain.service["orders.domain.service"]
        end

        subgraph orders.application["orders.application"]
            orders.application.api["orders.application.api"]
            orders.application.usecase["orders.application.usecase"]
        end

        subgraph orders.infrastructure["orders.infrastructure"]
            orders.infrastructure.persistence["orders.infrastructure.persistence"]
            orders.infrastructure.messaging["orders.infrastructure.messaging"]
        end
    end
```

**Key Rules:**
- **No arrows:** Structure only, not dependencies
- **Lowercase names:** `orders.payments`, not `Orders.Payments`
- **3-4 levels:** Too shallow = not useful, too deep = overwhelming
- **Empty nodes:** Just package labels, no implementation details

### Multi-Service Bounded Contexts

```mermaid
flowchart TB
    subgraph ecommerce["E-Commerce Platform"]
        subgraph catalog["catalog"]
            subgraph catalog.product["catalog.product"]
                catalog.product.api["catalog.product.api"]
                catalog.product.domain["catalog.product.domain"]
            end
            subgraph catalog.inventory["catalog.inventory"]
                catalog.inventory.tracking["catalog.inventory.tracking"]
                catalog.inventory.reservation["catalog.inventory.reservation"]
            end
        end

        subgraph orders["orders"]
            subgraph orders.cart["orders.cart"]
                orders.cart.api["orders.cart.api"]
                orders.cart.storage["orders.cart.storage"]
            end
            subgraph orders.checkout["orders.checkout"]
                orders.checkout.api["orders.checkout.api"]
                orders.checkout.workflow["orders.checkout.workflow"]
            end
            subgraph orders.fulfillment["orders.fulfillment"]
                orders.fulfillment.shipping["orders.fulfillment.shipping"]
                orders.fulfillment.tracking["orders.fulfillment.tracking"]
            end
        end

        subgraph payments["payments"]
            subgraph payments.gateway["payments.gateway"]
                payments.gateway.stripe["payments.gateway.stripe"]
                payments.gateway.paypal["payments.gateway.paypal"]
            end
            subgraph payments.processing["payments.processing"]
                payments.processing.authorization["payments.processing.authorization"]
                payments.processing.capture["payments.processing.capture"]
            end
        end
    end
```

## Common Architecture Patterns

### 1. Hexagonal Architecture (Ports & Adapters)

```mermaid
flowchart TB
    subgraph order_service["order-service"]
        subgraph order.domain["order.domain"]
            order.domain.model["order.domain.model"]
            order.domain.service["order.domain.service"]
            order.domain.event["order.domain.event"]
        end

        subgraph order.application["order.application"]
            order.application.port_in["order.application.port.in"]
            order.application.port_out["order.application.port.out"]
            order.application.usecase["order.application.usecase"]
        end

        subgraph order.adapter["order.adapter"]
            subgraph order.adapter.in["order.adapter.in"]
                order.adapter.in.rest["order.adapter.in.rest"]
                order.adapter.in.messaging["order.adapter.in.messaging"]
            end
            subgraph order.adapter.out["order.adapter.out"]
                order.adapter.out.persistence["order.adapter.out.persistence"]
                order.adapter.out.notification["order.adapter.out.notification"]
                order.adapter.out.payment["order.adapter.out.payment"]
            end
        end
    end
```

**Package Descriptions:**

| Package | Description |
|---------|-------------|
| `order.domain.model` | Core entities, value objects, aggregates |
| `order.domain.service` | Domain logic and business rules |
| `order.domain.event` | Domain events for state changes |
| `order.application.port.in` | Inbound port interfaces (use cases) |
| `order.application.port.out` | Outbound port interfaces (dependencies) |
| `order.application.usecase` | Use case implementations |
| `order.adapter.in.rest` | REST API controllers |
| `order.adapter.in.messaging` | Message queue consumers |
| `order.adapter.out.persistence` | Database adapters (JPA, MongoDB) |
| `order.adapter.out.notification` | Email/SMS notification adapters |
| `order.adapter.out.payment` | Payment gateway adapters |

### 2. Vertical Slice Architecture

```mermaid
flowchart TB
    subgraph user_service["user-service"]
        subgraph user.registration["user.registration"]
            user.registration.api["user.registration.api"]
            user.registration.workflow["user.registration.workflow"]
            user.registration.persistence["user.registration.persistence"]
        end

        subgraph user.authentication["user.authentication"]
            user.authentication.api["user.authentication.api"]
            user.authentication.token["user.authentication.token"]
            user.authentication.session["user.authentication.session"]
        end

        subgraph user.profile["user.profile"]
            user.profile.api["user.profile.api"]
            user.profile.update["user.profile.update"]
            user.profile.storage["user.profile.storage"]
        end

        subgraph user.shared["user.shared"]
            user.shared.domain["user.shared.domain"]
            user.shared.database["user.shared.database"]
        end
    end
```

**Package Descriptions:**

| Package | Description |
|---------|-------------|
| `user.registration.api` | Registration endpoints |
| `user.registration.workflow` | Sign-up and verification flow |
| `user.registration.persistence` | User creation data access |
| `user.authentication.api` | Login/logout endpoints |
| `user.authentication.token` | JWT generation and validation |
| `user.authentication.session` | Session management |
| `user.profile.api` | Profile CRUD endpoints |
| `user.profile.update` | Profile update logic |
| `user.profile.storage` | Profile data access |
| `user.shared.domain` | Shared user entities |
| `user.shared.database` | Database configuration |

### 3. Clean Architecture (Onion)

```mermaid
flowchart TB
    subgraph payment_service["payment-service"]
        subgraph payment.core["payment.core"]
            subgraph payment.core.domain["payment.core.domain"]
                payment.core.domain.entity["payment.core.domain.entity"]
                payment.core.domain.valueobject["payment.core.domain.valueobject"]
            end
            subgraph payment.core.usecase["payment.core.usecase"]
                payment.core.usecase.process["payment.core.usecase.process"]
                payment.core.usecase.refund["payment.core.usecase.refund"]
            end
        end

        subgraph payment.interface["payment.interface"]
            payment.interface.rest["payment.interface.rest"]
            payment.interface.grpc["payment.interface.grpc"]
            payment.interface.messaging["payment.interface.messaging"]
        end

        subgraph payment.infrastructure["payment.infrastructure"]
            payment.infrastructure.gateway["payment.infrastructure.gateway"]
            payment.infrastructure.persistence["payment.infrastructure.persistence"]
            payment.infrastructure.notification["payment.infrastructure.notification"]
        end
    end
```

### 4. DDD Bounded Contexts

```mermaid
flowchart TB
    subgraph platform["Platform"]
        subgraph sales["sales"]
            subgraph sales.lead["sales.lead"]
                sales.lead.qualification["sales.lead.qualification"]
                sales.lead.nurturing["sales.lead.nurturing"]
            end
            subgraph sales.opportunity["sales.opportunity"]
                sales.opportunity.pipeline["sales.opportunity.pipeline"]
                sales.opportunity.forecasting["sales.opportunity.forecasting"]
            end
        end

        subgraph fulfillment["fulfillment"]
            subgraph fulfillment.warehouse["fulfillment.warehouse"]
                fulfillment.warehouse.inventory["fulfillment.warehouse.inventory"]
                fulfillment.warehouse.picking["fulfillment.warehouse.picking"]
            end
            subgraph fulfillment.shipping["fulfillment.shipping"]
                fulfillment.shipping.carrier["fulfillment.shipping.carrier"]
                fulfillment.shipping.tracking["fulfillment.shipping.tracking"]
            end
        end

        subgraph accounting["accounting"]
            subgraph accounting.billing["accounting.billing"]
                accounting.billing.invoice["accounting.billing.invoice"]
                accounting.billing.payment["accounting.billing.payment"]
            end
            subgraph accounting.revenue["accounting.revenue"]
                accounting.revenue.recognition["accounting.revenue.recognition"]
                accounting.revenue.reporting["accounting.revenue.reporting"]
            end
        end
    end
```

### 5. Microservices with Shared Kernel

```mermaid
flowchart TB
    subgraph services["Microservices"]
        subgraph shared["shared"]
            shared.domain["shared.domain"]
            shared.event["shared.event"]
            shared.security["shared.security"]
        end

        subgraph product_service["product-service"]
            product.api["product.api"]
            product.catalog["product.catalog"]
            product.pricing["product.pricing"]
        end

        subgraph order_service["order-service"]
            order.api["order.api"]
            order.placement["order.placement"]
            order.fulfillment["order.fulfillment"]
        end

        subgraph inventory_service["inventory-service"]
            inventory.api["inventory.api"]
            inventory.tracking["inventory.tracking"]
            inventory.reservation["inventory.reservation"]
        end
    end
```

## Module Description Tables

### Hexagonal Example

| Package | Responsibility |
|---------|----------------|
| `order.domain.model` | Order aggregate, OrderItem value object, status enum |
| `order.domain.service` | Order validation, pricing calculation, state transitions |
| `order.domain.event` | OrderCreated, OrderConfirmed, OrderShipped events |
| `order.application.port.in` | CreateOrderUseCase, CancelOrderUseCase interfaces |
| `order.application.port.out` | LoadOrderPort, SaveOrderPort, SendEmailPort interfaces |
| `order.application.usecase` | Use case implementations orchestrating domain logic |
| `order.adapter.in.rest` | Spring MVC controllers exposing REST endpoints |
| `order.adapter.in.messaging` | Kafka listeners for events from other services |
| `order.adapter.out.persistence` | JPA repositories implementing outbound ports |
| `order.adapter.out.notification` | Email service adapter for notifications |
| `order.adapter.out.payment` | Payment gateway integration adapter |

### DDD Bounded Context Example

| Package | Responsibility |
|---------|----------------|
| `sales.lead.qualification` | Lead scoring, qualification rules, contact validation |
| `sales.lead.nurturing` | Email campaigns, drip sequences, engagement tracking |
| `sales.opportunity.pipeline` | Deal stages, probability tracking, sales process |
| `sales.opportunity.forecasting` | Revenue projections, win/loss analysis, quotas |
| `fulfillment.warehouse.inventory` | Stock levels, SKU management, location tracking |
| `fulfillment.warehouse.picking` | Pick list generation, wave picking, batch processing |
| `fulfillment.shipping.carrier` | Carrier integrations (FedEx, UPS, USPS) |
| `fulfillment.shipping.tracking` | Shipment tracking, delivery confirmation, exceptions |
| `accounting.billing.invoice` | Invoice generation, line items, tax calculation |
| `accounting.billing.payment` | Payment processing, reconciliation, dunning |

## Best Practices

### Naming Conventions
- **Lowercase dot notation:** `orders.payments.stripe`
- **Plural for collections:** `orders`, `users`, `products`
- **Singular for concepts:** `authentication`, `authorization`
- **No abbreviations:** `inventory` not `inv`, `customer` not `cust`

### Depth Guidelines
- **Level 1:** Top-level service or bounded context
- **Level 2:** Major functional areas or layers
- **Level 3:** Specific modules or features
- **Level 4:** Sub-modules (use sparingly, avoid if possible)

### Structure Clarity
- **No arrows:** Dependency flow belongs in sequence or component diagrams
- **Consistent grouping:** Layer, feature, or domain - pick one strategy
- **Balanced hierarchy:** Similar depth across branches
- **Avoid orphans:** Every package has a clear parent

### Table Quality
- **Match diagram:** Use exact same lowercase identifiers
- **Be specific:** "User profile CRUD endpoints" not "Profile stuff"
- **Show contents:** What files/classes live here
- **Indicate patterns:** "JPA repositories", "REST controllers", "Event handlers"

## Placement in arc42 Documentation

| Context | Section | Purpose |
|---------|---------|---------|
| Service modules | 5.2 Building Block View (Modules subsection) | Internal package structure |
| Bounded contexts | 5.1 Building Block View (Level 1) | High-level domain decomposition |
| Layered architecture | 5.2 or 5.3 | Architectural layer breakdown |
| Feature slices | 5.3 Building Block View (Level 3) | Vertical slice organization |

## Usage Examples

**Example 1: Hexagonal microservice**
```
User: Show module structure for order service using hexagonal architecture
Skill: [Generates diagram with domain, application (ports), adapter layers]
```

**Example 2: DDD bounded contexts**
```
User: Diagram bounded contexts for e-commerce platform
Skill: [Creates diagram with catalog, orders, payments, fulfillment contexts]
```

**Example 3: Vertical slices**
```
User: Module structure for user service organized by feature slices
Skill: [Shows registration, authentication, profile slices with shared domain]
```

**Example 4: Monolith decomposition**
```
User: Current monolith package structure before microservices migration
Skill: [Documents existing module hierarchy for strangler pattern planning]
```

## Common Anti-Patterns to Avoid

### ❌ Including Arrows
```mermaid
flowchart LR
    orders --> payments
```
**Why wrong:** Modules diagram shows structure, not dependencies

### ❌ Mixed Case Names
```mermaid
subgraph Orders.Payments
```
**Why wrong:** Convention is lowercase dot notation

### ❌ Too Flat (1-2 levels)
```mermaid
subgraph service
    api
    domain
    infra
end
```
**Why wrong:** Not enough detail to guide implementation

### ❌ Too Deep (5+ levels)
```mermaid
subgraph a.b.c.d.e.f
```
**Why wrong:** Overwhelming, hard to comprehend

### ✅ Correct Structure (3-4 levels, lowercase, no arrows)
```mermaid
flowchart TB
    subgraph orders["orders"]
        subgraph orders.domain["orders.domain"]
            orders.domain.model["orders.domain.model"]
            orders.domain.service["orders.domain.service"]
        end
        subgraph orders.adapter["orders.adapter"]
            orders.adapter.rest["orders.adapter.rest"]
            orders.adapter.persistence["orders.adapter.persistence"]
        end
    end
```

## Error Handling

- If architecture style unclear: Ask whether hexagonal, layered, vertical slice, or DDD
- If depth too shallow: Suggest breaking into sub-modules
- If depth too deep: Recommend consolidating packages
- If naming inconsistent: Standardize to lowercase dot notation

## Output Format

Always provide:
1. Mermaid flowchart with nested subgraphs (no arrows)
2. Module description table with matching lowercase identifiers
3. Architecture style explanation (hexagonal, vertical slice, etc.)
4. Recommended placement in arc42 Section 5
5. Implementation guidance (what files go where)

## Cross-References

- **C4 Component diagrams (c4-diagram skill):** Show dependencies between modules
- **API documentation (api-draft skill):** APIs exposed by adapter packages
- **Sequence diagrams (sequence-diagram skill):** Show interaction across modules
- **ADRs (create-adr skill):** Document architectural style decisions
