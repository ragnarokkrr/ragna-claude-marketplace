---
name: spring-config
description: Generate Spring Boot configuration files with profiles, datasources, and best practices
allowed-tools: ["Read", "Edit", "Write"]
---

# Spring Boot Configuration Skill

This skill generates production-ready Spring Boot configuration files (application.yml/properties) with proper profiles, datasource configuration, caching, security, and observability setup.

## Purpose

Create comprehensive Spring Boot configuration files with:
- **Multi-profile setup** (dev, test, prod)
- **Database configuration** (PostgreSQL, MySQL, MongoDB)
- **Virtual threads enablement** (JDK 21+)
- **Caching with Redis**
- **Security settings** (OAuth2, JWT)
- **Actuator and metrics**
- **Graceful shutdown and health checks**

## Instructions

When this skill is invoked:

1. **Determine configuration type:**
   - Ask which profile: dev, test, prod, or all
   - Identify database type: PostgreSQL, MySQL, MongoDB, or multiple
   - Check if Redis caching is needed
   - Confirm security requirements (OAuth2, JWT, basic auth)

2. **Gather application information:**
   - Application name
   - Server port (default: 8080)
   - Database connection details
   - Security provider (if applicable)
   - Monitoring/metrics requirements

3. **Generate configuration files:**
   - Main `application.yml` with common settings
   - Profile-specific configurations (`application-{profile}.yml`)
   - Include JDK 21+ virtual threads if applicable
   - Add environment variable placeholders for secrets

4. **Provide placement guidance:**
   - Files go in `src/main/resources/`
   - Test configs go in `src/test/resources/`

## Configuration Templates

### Main Application Configuration (application.yml)

**PostgreSQL + Redis + OAuth2:**
```yaml
spring:
  application:
    name: ${APPLICATION_NAME:service-name}

  # Enable virtual threads for Spring MVC (JDK 21+)
  threads:
    virtual:
      enabled: true

  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:servicedb}
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD}
    hikari:
      # With virtual threads, can support more connections efficiently
      maximum-pool-size: 50
      minimum-idle: 10
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000

  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    properties:
      hibernate:
        format_sql: true
        jdbc.batch_size: 20
        order_inserts: true
        order_updates: true
        default_schema: public

  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true

  cache:
    type: redis
    redis:
      time-to-live: 600000  # 10 minutes

  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}
      password: ${REDIS_PASSWORD:}
      timeout: 2000ms
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 2

  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: ${JWT_ISSUER_URI}
          jwk-set-uri: ${JWT_JWK_SET_URI:}

server:
  port: ${SERVER_PORT:8080}
  shutdown: graceful
  tomcat:
    threads:
      max: 200
      min-spare: 10
    connection-timeout: 20s
  error:
    include-message: always
    include-binding-errors: always

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
      base-path: /actuator
  endpoint:
    health:
      show-details: when-authorized
      probes:
        enabled: true
  health:
    livenessState:
      enabled: true
    readinessState:
      enabled: true
  metrics:
    export:
      prometheus:
        enabled: true
    tags:
      application: ${spring.application.name}
      environment: ${ENVIRONMENT:local}
  tracing:
    sampling:
      probability: 1.0

logging:
  level:
    root: INFO
    com.example: DEBUG
    org.springframework.web: INFO
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
```

**MySQL Configuration:**
```yaml
spring:
  datasource:
    url: jdbc:mysql://${DB_HOST:localhost}:${DB_PORT:3306}/${DB_NAME:servicedb}?useSSL=false&serverTimezone=UTC
    username: ${DB_USERNAME:root}
    password: ${DB_PASSWORD}
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      maximum-pool-size: 50
      minimum-idle: 10
      connection-timeout: 30000

  jpa:
    database-platform: org.hibernate.dialect.MySQL8Dialect
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        format_sql: true
        jdbc.batch_size: 20
```

**MongoDB Configuration:**
```yaml
spring:
  data:
    mongodb:
      uri: mongodb://${MONGO_USERNAME:}:${MONGO_PASSWORD:}@${MONGO_HOST:localhost}:${MONGO_PORT:27017}/${MONGO_DATABASE:servicedb}?authSource=admin
      auto-index-creation: false

  # MongoDB specific settings
  mongodb:
    embedded:
      version: 4.0.21  # For tests only
```

### Profile-Specific Configurations

**Development Profile (application-dev.yml):**
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/servicedb_dev
    hikari:
      maximum-pool-size: 10

  jpa:
    show-sql: true
    hibernate:
      ddl-auto: update  # Auto-create schema in dev

  flyway:
    enabled: false  # Disable in dev if using ddl-auto

  data:
    redis:
      host: localhost
      port: 6379

  devtools:
    restart:
      enabled: true
    livereload:
      enabled: true

logging:
  level:
    root: DEBUG
    com.example: TRACE
    org.springframework.web: DEBUG
    org.hibernate.SQL: DEBUG

management:
  endpoint:
    health:
      show-details: always
```

**Test Profile (application-test.yml):**
```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    driver-class-name: org.h2.Driver
    username: sa
    password:

  jpa:
    hibernate:
      ddl-auto: create-drop
    database-platform: org.hibernate.dialect.H2Dialect

  flyway:
    enabled: false

  cache:
    type: simple  # Use simple cache for tests

  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8081/auth

logging:
  level:
    root: WARN
    com.example: DEBUG
```

**Production Profile (application-prod.yml):**
```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 100
      leak-detection-threshold: 60000

  jpa:
    show-sql: false
    hibernate:
      ddl-auto: validate  # Never auto-create in production

  flyway:
    enabled: true
    validate-on-migrate: true

logging:
  level:
    root: WARN
    com.example: INFO
    org.springframework.web: WARN
    org.hibernate.SQL: WARN

management:
  endpoint:
    health:
      show-details: when-authorized
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus  # Limit exposure
  metrics:
    export:
      prometheus:
        enabled: true

server:
  tomcat:
    threads:
      max: 400  # Higher for production load
```

## Configuration Best Practices

### 1. Environment Variables for Secrets
```yaml
# NEVER hardcode secrets - use environment variables
spring:
  datasource:
    password: ${DB_PASSWORD}  # Required env var
  data:
    redis:
      password: ${REDIS_PASSWORD:}  # Optional (empty default)
```

### 2. Connection Pool Tuning
```yaml
# With virtual threads (JDK 21+), can handle more concurrent requests
spring:
  datasource:
    hikari:
      maximum-pool-size: 50  # Higher than traditional (usually 10-20)
      minimum-idle: 10
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000  # 30 minutes
      leak-detection-threshold: 60000  # Detect connection leaks
```

### 3. Graceful Shutdown
```yaml
server:
  shutdown: graceful  # Wait for requests to complete

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s  # Max wait time
```

### 4. Health Checks for Kubernetes
```yaml
management:
  endpoint:
    health:
      probes:
        enabled: true  # Enables /actuator/health/liveness and readiness
  health:
    livenessState:
      enabled: true
    readinessState:
      enabled: true
```

### 5. Observability Setup
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  metrics:
    tags:
      application: ${spring.application.name}
      environment: ${ENVIRONMENT:local}
      instance: ${HOSTNAME:${spring.application.name}}
    distribution:
      percentiles-histogram:
        http.server.requests: true
  tracing:
    sampling:
      probability: 1.0  # 100% in dev, reduce to 0.1 (10%) in prod
```

## Configuration Checklist

When generating Spring Boot configuration, ensure:

- ✓ Virtual threads enabled for JDK 21+ projects
- ✓ All secrets use environment variables (no hardcoded passwords)
- ✓ Proper connection pool sizing (consider virtual threads)
- ✓ Graceful shutdown configured
- ✓ Health probes enabled for Kubernetes
- ✓ Actuator endpoints secured and limited in production
- ✓ Logging levels appropriate per environment
- ✓ Flyway/Liquibase enabled in prod, disabled in test
- ✓ JPA `ddl-auto` set to `validate` in prod
- ✓ Redis caching configured with TTL
- ✓ Metrics tagged with application and environment
- ✓ Error response details controlled per environment

## Common Database Configurations

### PostgreSQL with Multiple Schemas
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/servicedb?currentSchema=myschema
  jpa:
    properties:
      hibernate:
        default_schema: myschema
```

### MySQL with UTC Timezone
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/servicedb?useSSL=false&serverTimezone=UTC&useLegacyDatetimeCode=false
```

### MongoDB with Replica Set
```yaml
spring:
  data:
    mongodb:
      uri: mongodb://user:pass@host1:27017,host2:27017,host3:27017/servicedb?replicaSet=rs0&authSource=admin
```

### SQL Server Configuration
```yaml
spring:
  datasource:
    url: jdbc:sqlserver://${DB_HOST:localhost}:${DB_PORT:1433};databaseName=${DB_NAME:servicedb}
    driver-class-name: com.microsoft.sqlserver.jdbc.SQLServerDriver
  jpa:
    database-platform: org.hibernate.dialect.SQLServer2012Dialect
```

## Integration Examples

### Kafka Integration
```yaml
spring:
  kafka:
    bootstrap-servers: ${KAFKA_BOOTSTRAP_SERVERS:localhost:9092}
    consumer:
      group-id: ${spring.application.name}
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "*"
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```

### RabbitMQ Integration
```yaml
spring:
  rabbitmq:
    host: ${RABBITMQ_HOST:localhost}
    port: ${RABBITMQ_PORT:5672}
    username: ${RABBITMQ_USERNAME:guest}
    password: ${RABBITMQ_PASSWORD:guest}
    listener:
      simple:
        concurrency: 5
        max-concurrency: 10
```

## Error Handling

- If database type is unclear: Ask user to specify PostgreSQL, MySQL, MongoDB, or other
- If security requirements unknown: Ask about OAuth2, JWT, or basic auth
- If JDK version < 21: Omit virtual threads configuration
- If Redis not needed: Use `spring.cache.type: simple` or `none`

## Output Format

Provide:
1. Main `application.yml` file with common configuration
2. Profile-specific files as requested (dev, test, prod)
3. List of required environment variables with descriptions
4. Placement instructions (src/main/resources/ or src/test/resources/)
5. Any additional configuration recommendations

## Usage Examples

**Example 1: Microservice with PostgreSQL and Redis**
```
User: Generate Spring Boot config for a microservice with PostgreSQL and Redis
Skill: [Creates application.yml with datasource, Redis cache, actuator, virtual threads]
```

**Example 2: MongoDB-based service**
```
User: Create config for MongoDB service with dev and prod profiles
Skill: [Generates application.yml with MongoDB URI, plus application-dev.yml and application-prod.yml]
```

**Example 3: Kafka event-driven service**
```
User: Config for service consuming from Kafka with OAuth2 security
Skill: [Creates complete config with Kafka consumer/producer, OAuth2 resource server, health checks]
```
