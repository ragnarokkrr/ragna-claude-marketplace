---
name: mongodb-schema
description: Document MongoDB collection schemas with relationships, indexes, and validation rules
allowed-tools: ["Read", "Edit", "Write"]
---

# MongoDB Schema Documentation Skill

This skill generates comprehensive MongoDB collection schema documentation using JSON-like notation with relationships, indexes, validation, and sharding strategies.

## Purpose

Create clear, actionable MongoDB schema documentation at two levels:
- **Level 1 (Domain/Conceptual):** High-level data model showing entities and relationships
- **Level 2/3 (Technical/Implementation):** Detailed schema with indexes, validation, sharding

## Instructions

When this skill is invoked:

1. **Determine documentation level:**
   - **Level 1:** For arc42 Section 8.1 (Domain & Data Concepts)
   - **Level 2/3:** For arc42 Section 5.3 (Components) or 7.2 (Infrastructure)

2. **Gather collection information:**
   - Collection names and purposes
   - Field names, types, and constraints
   - Embedded vs referenced relationships
   - Array fields and their purposes
   - Indexes needed for queries
   - Validation rules
   - Sharding strategy (if applicable)

3. **Generate appropriate documentation:**
   - Use JSON-like notation with type annotations
   - Show relationships with arrow notation (→)
   - Include index and validation sections for technical level
   - Add Mermaid relationship diagram for visual representation

4. **Provide placement guidance:**
   - Explain where to place in arc42 documentation
   - Suggest related documentation needs

## Schema Templates

### Level 1: Domain/Conceptual Model

For high-level understanding in Section 8.1:

```javascript
// Collection: users
{
  _id: ObjectId,
  email: String,
  profile: {                      // embedded document
    firstName: String,
    lastName: String,
    avatar: String
  },
  addresses: [{                   // embedded array
    street: String,
    city: String,
    country: String
  }],
  organizationId: ObjectId,       // → organizations
  created_at: Date,
  updated_at: Date
}

// Collection: organizations
{
  _id: ObjectId,
  name: String,
  plan: String,
  members: [ObjectId],            // → users (array of refs)
  settings: {                     // embedded document
    billing: Object,
    features: [String]
  },
  created_at: Date
}

// Relationships:
// users → organizations (many-to-one)
// organizations → users (one-to-many via members array)
```

### Level 2/3: Technical/Implementation Model

For detailed implementation in Section 5.3 or 7.2:

```javascript
// Collection: orders
{
  _id: ObjectId,                  // PK, auto-generated
  orderNumber: String,            // unique, indexed
  userId: ObjectId,               // → users, indexed
  status: String,                 // enum: pending|confirmed|shipped|delivered|cancelled
  items: [{                       // embedded array
    productId: ObjectId,          // → products
    quantity: Number,             // min: 1
    price: Decimal128,            // currency amount
    snapshot: {                   // embedded product snapshot
      name: String,
      sku: String
    }
  }],
  totals: {                       // embedded document
    subtotal: Decimal128,
    tax: Decimal128,
    shipping: Decimal128,
    total: Decimal128
  },
  shippingAddress: {              // embedded document
    street: String,
    city: String,
    state: String,
    zipCode: String,
    country: String
  },
  paymentId: ObjectId,            // → payments (reference)
  created_at: Date,               // TTL candidate
  updated_at: Date,
  shipped_at: Date                // sparse index
}

// Indexes:
// 1. { orderNumber: 1 } - unique
// 2. { userId: 1, created_at: -1 } - user order history
// 3. { status: 1, created_at: -1 } - status-based queries
// 4. { created_at: 1 } - TTL index (expireAfterSeconds: 31536000) for old orders
// 5. { "items.productId": 1 } - multikey index for product lookup

// Validation:
// - orderNumber: required, string, unique
// - userId: required, ObjectId
// - status: required, enum values
// - items: array required, min length 1
// - items.quantity: integer, min 1
// - items.price: required, Decimal128, min 0
// - totals.total: required, Decimal128, min 0

// Sharding:
// - Shard key: { userId: 1, _id: 1 } (compound)
// - Strategy: hash on userId for even distribution
// - Zone sharding: by country for GDPR compliance
```

### Mermaid Relationship Diagram

Use Mermaid `erDiagram` syntax for MongoDB collection relationships:

```mermaid
erDiagram
    USERS ||--o{ ORDERS : "places"
    USERS }o--|| ORGANIZATIONS : "belongs to"
    ORDERS ||--|{ ORDER_ITEMS : "contains"
    ORDERS ||--|| PAYMENTS : "paid by"
    PRODUCTS ||--o{ ORDER_ITEMS : "ordered as"

    USERS {
        ObjectId _id PK
        String email UNIQUE
        Object profile EMBED
        Array addresses EMBED
        ObjectId organizationId FK-REF
        Date created_at
    }

    ORDERS {
        ObjectId _id PK
        String orderNumber UNIQUE
        ObjectId userId FK-REF
        String status ENUM
        Array items EMBED
        Object totals EMBED
        ObjectId paymentId FK-REF
        Date created_at
    }

    PRODUCTS {
        ObjectId _id PK
        String sku UNIQUE
        String name
        Decimal128 price
    }

    ORGANIZATIONS {
        ObjectId _id PK
        String name
        Array members FK-REF-ARRAY
    }
```

**Notation Key:**
- `PK` - Primary key (_id)
- `UNIQUE` - Unique index
- `FK-REF` - Reference to another collection (ObjectId)
- `FK-REF-ARRAY` - Array of references
- `EMBED` - Embedded document or array
- `ENUM` - Enumerated values

## Relationship Patterns

### Embedded Document (One-to-One)
```javascript
// Use when: Data always accessed together, no independent queries
{
  _id: ObjectId,
  userId: String,
  profile: {              // EMBED
    firstName: String,
    lastName: String,
    bio: String
  }
}
```

### Reference (One-to-Many)
```javascript
// Use when: Related data accessed independently, large subdocuments
// Parent
{ _id: ObjectId, name: String }

// Child
{ _id: ObjectId, parentId: ObjectId }  // → parent (FK-REF)
```

### Array of References (Many-to-Many)
```javascript
// Use when: Many-to-many, need to query from both sides
// Collection A
{ _id: ObjectId, relatedBs: [ObjectId] }  // → collectionB (FK-REF-ARRAY)

// Collection B
{ _id: ObjectId, relatedAs: [ObjectId] }  // → collectionA (FK-REF-ARRAY)
```

### Embedded Array (One-to-Many, bounded)
```javascript
// Use when: Small, finite number of subdocuments, always accessed together
{
  _id: ObjectId,
  items: [{            // EMBED array
    id: UUID,
    name: String,
    quantity: Number
  }]
}
```

## Design Decision Guide

| Factor | Embed | Reference |
|--------|-------|-----------|
| Access pattern | Always together | Independent queries |
| Subdocument size | Small (<16MB limit) | Large or unbounded |
| Update frequency | Rare | Frequent |
| Data duplication | Acceptable | Must avoid |
| Query complexity | Simple | Complex joins needed |
| Consistency | Atomic updates | Eventual consistency OK |

## Index Strategy Guide

### Common Index Types

```javascript
// Single field
{ field: 1 }                    // ascending
{ field: -1 }                   // descending

// Compound index
{ field1: 1, field2: -1 }       // multi-field queries

// Unique index
{ email: 1 }, { unique: true }

// Sparse index (only docs with field)
{ optional_field: 1 }, { sparse: true }

// Multikey index (array fields)
{ "items.productId": 1 }        // index array elements

// TTL index (auto-delete old docs)
{ created_at: 1 }, { expireAfterSeconds: 2592000 }

// Text index (full-text search)
{ description: "text", tags: "text" }

// Geospatial index
{ location: "2dsphere" }
```

### Index Selection Criteria

1. **Query patterns:** Index fields in WHERE clauses
2. **Sort optimization:** Include sort fields in compound indexes
3. **Covered queries:** Include all query fields for in-memory results
4. **Write performance:** Fewer indexes = faster writes
5. **Selectivity:** Index high-cardinality fields first

## Sharding Strategies

### Hash Sharding
```javascript
// Shard key: { userId: "hashed" }
// Pros: Even distribution
// Cons: No range queries on shard key
// Use when: Uniform access, no range queries needed
```

### Range Sharding
```javascript
// Shard key: { created_at: 1 }
// Pros: Efficient range queries, time-series optimization
// Cons: Hotspot risk on newest data
// Use when: Time-series data, archival patterns
```

### Compound Sharding
```javascript
// Shard key: { tenantId: 1, _id: 1 }
// Pros: Tenant isolation, good distribution
// Cons: Routing complexity
// Use when: Multi-tenant SaaS, tenant-scoped queries
```

### Zone Sharding
```javascript
// Zones based on country field
// Pros: Data locality, compliance (GDPR)
// Cons: Configuration overhead
// Use when: Geographic compliance, data residency requirements
```

## Usage Examples

**Example 1: E-commerce order schema**
```
User: Document the orders collection with items and payments
Skill: [Generates Level 2 schema with embedded items, payment reference, indexes for userId and status]
```

**Example 2: Multi-tenant SaaS**
```
User: Schema for organizations and users with tenant isolation
Skill: [Creates schemas with tenantId shard key, compound indexes, zone sharding strategy]
```

**Example 3: Event sourcing**
```
User: Event log collection with TTL and time-series optimization
Skill: [Designs schema with timestamp-based sharding, TTL index, sparse indexes]
```

## Best Practices

### Schema Design
- **Favor embedding for 1-to-1 and small 1-to-few relationships**
- **Use references for 1-to-many and many-to-many**
- **Denormalize strategically for read performance**
- **Keep documents under 16MB limit**
- **Use schema validation for data quality**

### Indexing
- **Index query patterns, not all fields**
- **Compound indexes: equality → sort → range**
- **Monitor index usage with `$indexStats`**
- **Remove unused indexes to improve write performance**
- **Use covered queries when possible**

### References vs Embedding
- **Embed when:** Data accessed together, atomic updates needed, bounded size
- **Reference when:** Data queried independently, large subdocs, many-to-many

### Naming Conventions
- **Collections:** Plural lowercase (users, orders, products)
- **Fields:** camelCase (firstName, orderNumber)
- **Embedded docs:** Descriptive objects (shippingAddress, profile)
- **References:** Suffix with Id (userId, productId, organizationId)

## Placement in arc42 Documentation

| Level | Section | Content |
|-------|---------|---------|
| Level 1 (Domain) | 8.1 Domain & Data Concepts | Conceptual MongoDB schema, embedded vs referenced relationships |
| Level 2/3 (Technical) | 5.3 Components | Detailed MongoDB schema with indexes, validation rules |
| Level 2/3 (Infrastructure) | 7.2 Infrastructure Map | MongoDB sharding strategy, replica set topology |

## Error Handling

- If relationship unclear: Ask whether to embed or reference
- If index strategy needed: Ask about common query patterns
- If sharding required: Ask about scale, tenant isolation, data locality
- If validation rules unclear: Ask about business constraints

## Output Format

Always provide:
1. JSON-like MongoDB schema notation with type annotations
2. Relationship arrows (→) for document references
3. MongoDB index specifications with rationale
4. Validation rules for data quality
5. Sharding strategy (if needed for scale)
6. Mermaid relationship diagram showing collections and references
7. Recommended placement in arc42 documentation

## Cross-References

- **C4 diagrams (c4-diagram skill):** MongoDB collections complement container/component views
- **API documentation (api-draft skill):** APIs map to MongoDB collections and operations
- **Sequence diagrams (sequence-diagram skill):** Show MongoDB read/write transaction flows
- **ADRs (create-adr skill):** Document MongoDB vs SQL decision, schema design choices
- **SQL schemas (sql-schema skill):** Use for relational databases instead of MongoDB
