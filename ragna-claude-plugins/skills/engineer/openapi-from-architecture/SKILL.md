---
name: openapi-from-architecture
description: Generate OpenAPI 3.x YAML specifications from architecture documentation (arc42, ADRs, API drafts)
allowed-tools: ["Read", "Write", "Glob", "Grep"]
---

# OpenAPI from Architecture Skill

This skill converts API documentation from architecture specifications (arc42 Section 5.2, API drafts, ADRs) into formal OpenAPI 3.x YAML files suitable for contract-first API development.

## Purpose

Transform architectural API designs into:
- **OpenAPI 3.x YAML** specification files
- **Contract-first** API definitions for code generation
- **Documentation-ready** specs for Swagger UI / Redoc
- **Client SDK generation** ready specifications

## When to Use

- After backend-architect creates API documentation in arc42
- During principal-engineer scaffolding phase
- Before generating REST controllers (use with controllers-from-openapi skill)
- When converting architectural designs to executable contracts

## Workflow Integration

```
backend-architect          principal-engineer         principal-engineer
(API draft in arc42) → (openapi-from-architecture) → (controllers-from-openapi)
                           ↓
                       openapi.yaml
```

## Instructions

### Phase 1: Locate API Documentation

1. **Read architecture documentation:**
   - Primary: `.spec/architecture/application-architecture.md` (arc42 Section 5.2)
   - Supporting: `.spec/architecture/adrs/*.md` for API versioning, security, error handling decisions

2. **Extract API information:**
   - REST endpoints (methods, paths, parameters, responses)
   - Request/response schemas (DTOs, entities)
   - Authentication/authorization requirements
   - Error response formats
   - API versioning strategy
   - Rate limiting and pagination patterns

3. **Identify missing information:**
   - If schema details are incomplete, infer reasonable defaults
   - Document assumptions in OpenAPI description fields
   - Add TODO comments for items requiring clarification

### Phase 2: Generate OpenAPI 3.x YAML

Create `openapi.yaml` (or `{service-name}-openapi.yaml` for microservices) with complete specification.

**File location:**
- Single service: `./openapi.yaml` (project root)
- Microservices: `./{service-name}/openapi.yaml` or `./api-specs/{service-name}-openapi.yaml`

**OpenAPI 3.x Structure:**

```yaml
openapi: 3.0.3
info:
  title: {Service Name} API
  description: |
    {Service description from architecture}

    Generated from architecture specification:
    - Architecture: .spec/architecture/application-architecture.md
    - ADRs: .spec/architecture/adrs/

    **Architecture Pattern:** {Hexagonal|Vertical Slice|DDD|Layered}
    **Authentication:** {OAuth2|JWT|API Key|Session}
    **Versioning:** {URI|Header|Query Parameter}
  version: 1.0.0
  contact:
    name: {Team Name}
    email: {team@example.com}
  license:
    name: {License}
    url: {License URL}

servers:
  - url: http://localhost:8080
    description: Local development
  - url: https://api-dev.example.com
    description: Development environment
  - url: https://api-staging.example.com
    description: Staging environment
  - url: https://api.example.com
    description: Production environment

tags:
  - name: {Resource1}
    description: {Resource1 operations}
  - name: {Resource2}
    description: {Resource2 operations}

paths:
  /api/v1/{resources}:
    get:
      tags:
        - {Resource}
      summary: List all {resources}
      description: |
        Returns paginated list of {resources}.
        Supports filtering, sorting, and pagination.
      operationId: list{Resources}
      parameters:
        - name: page
          in: query
          description: Page number (zero-based)
          required: false
          schema:
            type: integer
            default: 0
            minimum: 0
        - name: size
          in: query
          description: Number of items per page
          required: false
          schema:
            type: integer
            default: 20
            minimum: 1
            maximum: 100
        - name: sort
          in: query
          description: Sort field and direction (e.g., "name,asc" or "createdAt,desc")
          required: false
          schema:
            type: string
            example: "name,asc"
      responses:
        '200':
          description: Successful operation
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/{Resource}Page'
        '400':
          description: Bad request - invalid parameters
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '401':
          description: Unauthorized - authentication required
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '403':
          description: Forbidden - insufficient permissions
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
      security:
        - bearerAuth: []

    post:
      tags:
        - {Resource}
      summary: Create new {resource}
      description: Creates a new {resource} with the provided data
      operationId: create{Resource}
      requestBody:
        description: {Resource} object to create
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Create{Resource}Request'
      responses:
        '201':
          description: Resource created successfully
          headers:
            Location:
              description: URI of the created resource
              schema:
                type: string
                format: uri
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/{Resource}Dto'
        '400':
          description: Bad request - validation failed
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '401':
          description: Unauthorized - authentication required
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '403':
          description: Forbidden - insufficient permissions
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '409':
          description: Conflict - resource already exists
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
      security:
        - bearerAuth: []

  /api/v1/{resources}/{id}:
    get:
      tags:
        - {Resource}
      summary: Get {resource} by ID
      description: Returns a single {resource} by its unique identifier
      operationId: get{Resource}ById
      parameters:
        - name: id
          in: path
          description: {Resource} unique identifier
          required: true
          schema:
            type: integer
            format: int64
      responses:
        '200':
          description: Successful operation
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/{Resource}Dto'
        '401':
          description: Unauthorized - authentication required
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '403':
          description: Forbidden - insufficient permissions
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '404':
          description: Resource not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
      security:
        - bearerAuth: []

    put:
      tags:
        - {Resource}
      summary: Update {resource}
      description: Updates an existing {resource} with the provided data
      operationId: update{Resource}
      parameters:
        - name: id
          in: path
          description: {Resource} unique identifier
          required: true
          schema:
            type: integer
            format: int64
      requestBody:
        description: Updated {resource} object
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Update{Resource}Request'
      responses:
        '200':
          description: Resource updated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/{Resource}Dto'
        '400':
          description: Bad request - validation failed
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '401':
          description: Unauthorized - authentication required
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '403':
          description: Forbidden - insufficient permissions
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '404':
          description: Resource not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
      security:
        - bearerAuth: []

    patch:
      tags:
        - {Resource}
      summary: Partially update {resource}
      description: Partially updates an existing {resource}. Only provided fields are updated.
      operationId: patch{Resource}
      parameters:
        - name: id
          in: path
          description: {Resource} unique identifier
          required: true
          schema:
            type: integer
            format: int64
      requestBody:
        description: Partial {resource} update
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Patch{Resource}Request'
      responses:
        '200':
          description: Resource updated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/{Resource}Dto'
        '400':
          description: Bad request - validation failed
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '401':
          description: Unauthorized - authentication required
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '403':
          description: Forbidden - insufficient permissions
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '404':
          description: Resource not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
      security:
        - bearerAuth: []

    delete:
      tags:
        - {Resource}
      summary: Delete {resource}
      description: Deletes a {resource} by its unique identifier
      operationId: delete{Resource}
      parameters:
        - name: id
          in: path
          description: {Resource} unique identifier
          required: true
          schema:
            type: integer
            format: int64
      responses:
        '204':
          description: Resource deleted successfully
        '401':
          description: Unauthorized - authentication required
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '403':
          description: Forbidden - insufficient permissions
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '404':
          description: Resource not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
      security:
        - bearerAuth: []

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: |
        JWT token-based authentication.
        Obtain token from /api/v1/auth/login endpoint.

        Example: Authorization: Bearer <token>

  schemas:
    # Request DTOs
    Create{Resource}Request:
      type: object
      required:
        - {requiredField1}
        - {requiredField2}
      properties:
        {field1}:
          type: string
          minLength: 1
          maxLength: 255
          description: {Field description}
          example: "{Example value}"
        {field2}:
          type: string
          description: {Field description}
          example: "{Example value}"
        {field3}:
          type: integer
          format: int32
          minimum: 1
          description: {Field description}
          example: 42

    Update{Resource}Request:
      type: object
      required:
        - {requiredField1}
      properties:
        {field1}:
          type: string
          minLength: 1
          maxLength: 255
          description: {Field description}
        {field2}:
          type: string
          description: {Field description}

    Patch{Resource}Request:
      type: object
      description: All fields are optional for partial update
      properties:
        {field1}:
          type: string
          minLength: 1
          maxLength: 255
          description: {Field description}
        {field2}:
          type: string
          description: {Field description}

    # Response DTOs
    {Resource}Dto:
      type: object
      properties:
        id:
          type: integer
          format: int64
          description: Unique identifier
          example: 1
        {field1}:
          type: string
          description: {Field description}
          example: "{Example value}"
        {field2}:
          type: string
          description: {Field description}
          example: "{Example value}"
        createdAt:
          type: string
          format: date-time
          description: Timestamp when resource was created
          example: "2025-01-17T10:30:00Z"
        updatedAt:
          type: string
          format: date-time
          description: Timestamp when resource was last updated
          example: "2025-01-17T10:30:00Z"

    {Resource}Page:
      type: object
      properties:
        content:
          type: array
          items:
            $ref: '#/components/schemas/{Resource}Dto'
        page:
          type: integer
          description: Current page number (zero-based)
          example: 0
        size:
          type: integer
          description: Number of items per page
          example: 20
        totalElements:
          type: integer
          format: int64
          description: Total number of items across all pages
          example: 100
        totalPages:
          type: integer
          description: Total number of pages
          example: 5
        last:
          type: boolean
          description: Whether this is the last page
          example: false

    # Error Response
    ErrorResponse:
      type: object
      required:
        - timestamp
        - status
        - error
        - message
        - path
      properties:
        timestamp:
          type: string
          format: date-time
          description: Timestamp when error occurred
          example: "2025-01-17T10:30:00Z"
        status:
          type: integer
          description: HTTP status code
          example: 400
        error:
          type: string
          description: HTTP status text
          example: "Bad Request"
        message:
          type: string
          description: Error message describing what went wrong
          example: "Validation failed for field 'name': must not be blank"
        path:
          type: string
          description: Request path that caused the error
          example: "/api/v1/resources"
        errors:
          type: array
          description: List of validation errors (for 400 Bad Request)
          items:
            $ref: '#/components/schemas/ValidationError'

    ValidationError:
      type: object
      properties:
        field:
          type: string
          description: Field that failed validation
          example: "name"
        rejectedValue:
          type: string
          description: Value that was rejected
          example: ""
        message:
          type: string
          description: Validation error message
          example: "must not be blank"

security:
  - bearerAuth: []
```

### Phase 3: Validation and Enhancement

1. **Validate OpenAPI spec:**
   - Use online validator: https://editor.swagger.io/
   - Or local validator: `npx @apidevtools/swagger-cli validate openapi.yaml`

2. **Enhance with examples:**
   - Add realistic example values for all schema properties
   - Include example request/response payloads
   - Document common error scenarios

3. **Security configuration:**
   - Configure security schemes based on ADRs (OAuth2, JWT, API Key)
   - Apply security requirements to endpoints
   - Document authentication flow

4. **Add OpenAPI extensions:**
   - `x-spring-paginated: true` for paginated endpoints
   - `x-java-class-name` for custom class names
   - `x-implements` for interface generation

### Phase 4: Documentation

Add metadata to OpenAPI file:

```yaml
info:
  x-architecture-reference:
    arc42: ".spec/architecture/application-architecture.md"
    adrs:
      - ".spec/architecture/adrs/adr-0001-rest-api-design.md"
      - ".spec/architecture/adrs/adr-0002-api-versioning.md"
  x-generation-info:
    generated-by: "principal-engineer agent"
    source: "architecture specification"
    date: "2025-01-17"
```

## Best Practices

### Schema Design

1. **Use consistent naming:**
   - Request DTOs: `Create{Resource}Request`, `Update{Resource}Request`, `Patch{Resource}Request`
   - Response DTOs: `{Resource}Dto`, `{Resource}Page`
   - Error responses: `ErrorResponse`, `ValidationError`

2. **Validation constraints:**
   - Add `required` fields
   - Use `minLength`, `maxLength` for strings
   - Use `minimum`, `maximum` for numbers
   - Use `format` for dates, emails, URIs
   - Use `pattern` for regex validation
   - Use `enum` for fixed value sets

3. **Pagination pattern:**
   ```yaml
   parameters:
     - $ref: '#/components/parameters/PageParam'
     - $ref: '#/components/parameters/SizeParam'
     - $ref: '#/components/parameters/SortParam'
   ```

4. **Reusable components:**
   - Extract common schemas to `components/schemas`
   - Extract common parameters to `components/parameters`
   - Extract common responses to `components/responses`
   - Extract common headers to `components/headers`

### API Versioning

Based on ADR decisions, implement versioning:

**URI Versioning (recommended):**
```yaml
paths:
  /api/v1/resources:
    # v1 endpoints
  /api/v2/resources:
    # v2 endpoints
```

**Header Versioning:**
```yaml
parameters:
  - name: API-Version
    in: header
    required: true
    schema:
      type: string
      enum: ["1.0", "2.0"]
```

### Security Schemes

**JWT Bearer Token (most common):**
```yaml
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

**OAuth2:**
```yaml
components:
  securitySchemes:
    oauth2:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://auth.example.com/oauth/authorize
          tokenUrl: https://auth.example.com/oauth/token
          scopes:
            read: Read access
            write: Write access
            admin: Admin access
```

**API Key:**
```yaml
components:
  securitySchemes:
    apiKey:
      type: apiKey
      in: header
      name: X-API-Key
```

## Output Location

**File:** `{project-root}/openapi.yaml` or `{project-root}/api-specs/{service-name}-openapi.yaml`

**Structure for microservices:**
```
project-root/
├── api-specs/
│   ├── user-service-openapi.yaml
│   ├── order-service-openapi.yaml
│   └── product-service-openapi.yaml
└── .spec/
    └── architecture/
        └── application-architecture.md
```

## Integration with Controllers Generation

After generating OpenAPI spec, use the `controllers-from-openapi` skill to generate Spring Boot controllers:

```
1. openapi-from-architecture → generates openapi.yaml
2. controllers-from-openapi   → reads openapi.yaml, generates controllers
```

## Example Output

For a User Management API:

```yaml
openapi: 3.0.3
info:
  title: User Management API
  description: |
    User management service for authentication and authorization.

    Generated from architecture specification:
    - Architecture: .spec/architecture/application-architecture.md
    - Pattern: Hexagonal Architecture
    - Authentication: JWT Bearer Token
  version: 1.0.0

servers:
  - url: http://localhost:8080
    description: Local development

tags:
  - name: Users
    description: User management operations

paths:
  /api/v1/users:
    get:
      tags: [Users]
      summary: List all users
      operationId: listUsers
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 0
        - name: size
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: Successful operation
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserPage'
      security:
        - bearerAuth: []

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    UserDto:
      type: object
      properties:
        id:
          type: integer
          format: int64
        email:
          type: string
          format: email
        firstName:
          type: string
        lastName:
          type: string
        role:
          type: string
          enum: [USER, ADMIN]
        createdAt:
          type: string
          format: date-time
```

## Checklist

Before completing this skill, verify:

- [ ] OpenAPI version is 3.0.3 or 3.1.0
- [ ] All paths follow RESTful conventions
- [ ] All request/response schemas are defined in components
- [ ] All required fields are marked as required
- [ ] Validation constraints are applied (minLength, maxLength, pattern, etc.)
- [ ] Security schemes are configured per ADRs
- [ ] Error responses are documented (400, 401, 403, 404, 500)
- [ ] Pagination parameters are consistent across list endpoints
- [ ] Examples are provided for all schemas
- [ ] Operation IDs are unique and follow naming convention
- [ ] Tags are used to group related endpoints
- [ ] Multiple server environments are defined
- [ ] Contact and license information is provided
- [ ] File is saved in correct location

---

Generated OpenAPI specs serve as the single source of truth for API contracts and enable contract-first development with Spring Boot.
