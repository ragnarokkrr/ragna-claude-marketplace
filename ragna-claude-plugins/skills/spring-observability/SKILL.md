---
name: spring-observability
description: Generate Spring Boot observability setup with Actuator, Micrometer, Prometheus, Grafana, and distributed tracing
allowed-tools: ["Read", "Edit", "Write"]
---

# Spring Boot Observability Skill

This skill generates comprehensive observability infrastructure for Spring Boot applications including health checks, metrics, distributed tracing, logging, and monitoring dashboards.

## Purpose

Enable production-grade observability:
- **Spring Boot Actuator** for health and metrics endpoints
- **Micrometer** for metrics collection
- **Prometheus** for metrics storage
- **Grafana** dashboards for visualization
- **Zipkin/Jaeger** for distributed tracing
- **Structured logging** with correlation IDs
- **Custom metrics** and health indicators

## Actuator Configuration

### Application Configuration

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus,loggers,env
      base-path: /actuator
  endpoint:
    health:
      show-details: when-authorized
      probes:
        enabled: true  # For Kubernetes liveness/readiness
    metrics:
      enabled: true
  health:
    livenessState:
      enabled: true
    readinessState:
      enabled: true
    db:
      enabled: true
    redis:
      enabled: true
  metrics:
    export:
      prometheus:
        enabled: true
    tags:
      application: ${spring.application.name}
      environment: ${ENVIRONMENT:local}
      instance: ${HOSTNAME:localhost}
    distribution:
      percentiles-histogram:
        http.server.requests: true
      slo:
        http.server.requests: 50ms,100ms,200ms,400ms,1s
  tracing:
    sampling:
      probability: 1.0  # 100% for dev, 0.1 (10%) for prod
  observations:
    http:
      server:
        requests:
          name: http.server.requests

info:
  app:
    name: ${spring.application.name}
    version: @project.version@
    description: @project.description@
```

### Maven Dependencies

```xml
<dependencies>
    <!-- Actuator -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <!-- Micrometer Prometheus -->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-prometheus</artifactId>
    </dependency>

    <!-- Distributed Tracing -->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-tracing-bridge-brave</artifactId>
    </dependency>
    <dependency>
        <groupId>io.zipkin.reporter2</groupId>
        <artifactId>zipkin-reporter-brave</artifactId>
    </dependency>

    <!-- Logback for structured logging -->
    <dependency>
        <groupId>net.logstash.logback</groupId>
        <artifactId>logstash-logback-encoder</artifactId>
        <version>7.4</version>
    </dependency>
</dependencies>
```

## Custom Health Indicators

```java
package com.example.health;

import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class ExternalServiceHealthIndicator implements HealthIndicator {

    private final ExternalServiceClient externalServiceClient;

    public ExternalServiceHealthIndicator(ExternalServiceClient externalServiceClient) {
        this.externalServiceClient = externalServiceClient;
    }

    @Override
    public Health health() {
        try {
            boolean isHealthy = externalServiceClient.healthCheck();

            if (isHealthy) {
                return Health.up()
                    .withDetail("service", "external-api")
                    .withDetail("status", "reachable")
                    .build();
            } else {
                return Health.down()
                    .withDetail("service", "external-api")
                    .withDetail("status", "unreachable")
                    .build();
            }
        } catch (Exception e) {
            return Health.down()
                .withDetail("service", "external-api")
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}
```

## Custom Metrics

```java
package com.example.metrics;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import org.springframework.stereotype.Component;

@Component
public class BusinessMetrics {

    private final Counter resourceCreatedCounter;
    private final Counter resourceDeletedCounter;
    private final Timer resourceProcessingTimer;

    public BusinessMetrics(MeterRegistry meterRegistry) {
        this.resourceCreatedCounter = Counter.builder("resources.created")
            .description("Total resources created")
            .tag("type", "resource")
            .register(meterRegistry);

        this.resourceDeletedCounter = Counter.builder("resources.deleted")
            .description("Total resources deleted")
            .tag("type", "resource")
            .register(meterRegistry);

        this.resourceProcessingTimer = Timer.builder("resources.processing.time")
            .description("Time taken to process resources")
            .tag("operation", "process")
            .register(meterRegistry);
    }

    public void recordResourceCreated() {
        resourceCreatedCounter.increment();
    }

    public void recordResourceDeleted() {
        resourceDeletedCounter.increment();
    }

    public Timer.Sample startProcessingTimer() {
        return Timer.start();
    }

    public void recordProcessingTime(Timer.Sample sample) {
        sample.stop(resourceProcessingTimer);
    }
}
```

## Structured Logging Configuration

```xml
<!-- src/main/resources/logback-spring.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

    <springProperty scope="context" name="applicationName" source="spring.application.name"/>

    <!-- Console appender with JSON formatting -->
    <appender name="CONSOLE_JSON" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <customFields>{"application":"${applicationName}"}</customFields>
            <includeMdcKeyName>traceId</includeMdcKeyName>
            <includeMdcKeyName>spanId</includeMdcKeyName>
            <includeMdcKeyName>userId</includeMdcKeyName>
        </encoder>
    </appender>

    <!-- File appender -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/application-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>

    <springProfile name="dev,local">
        <root level="INFO">
            <appender-ref ref="CONSOLE_JSON"/>
        </root>
        <logger name="com.example" level="DEBUG"/>
    </springProfile>

    <springProfile name="prod">
        <root level="WARN">
            <appender-ref ref="CONSOLE_JSON"/>
            <appender-ref ref="FILE"/>
        </root>
        <logger name="com.example" level="INFO"/>
    </springProfile>
</configuration>
```

## Prometheus Configuration

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'spring-boot-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['app:8080']
```

## Grafana Dashboard JSON

```json
{
  "dashboard": {
    "title": "Spring Boot Application Metrics",
    "panels": [
      {
        "title": "HTTP Requests Rate",
        "targets": [
          {
            "expr": "rate(http_server_requests_seconds_count[5m])"
          }
        ]
      },
      {
        "title": "HTTP Request Duration",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_server_requests_seconds_bucket[5m]))"
          }
        ]
      },
      {
        "title": "JVM Memory Usage",
        "targets": [
          {
            "expr": "jvm_memory_used_bytes{area=\"heap\"}"
          }
        ]
      },
      {
        "title": "Database Connection Pool",
        "targets": [
          {
            "expr": "hikaricp_connections_active"
          },
          {
            "expr": "hikaricp_connections_idle"
          }
        ]
      }
    ]
  }
}
```

## Docker Compose with Observability Stack

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - MANAGEMENT_ZIPKIN_TRACING_ENDPOINT=http://zipkin:9411
    depends_on:
      - prometheus
      - zipkin

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana
    depends_on:
      - prometheus

  zipkin:
    image: openzipkin/zipkin:latest
    ports:
      - "9411:9411"

volumes:
  prometheus-data:
  grafana-data:
```

## Best Practices

- ✓ Enable health probes for Kubernetes
- ✓ Use Prometheus for metrics (industry standard)
- ✓ Implement custom business metrics
- ✓ Add correlation IDs to all logs
- ✓ Use structured logging (JSON format)
- ✓ Monitor JVM metrics (heap, GC, threads)
- ✓ Track database connection pool metrics
- ✓ Set up alerts for critical metrics
- ✓ Sample traces (10% in prod to reduce overhead)
- ✓ Secure actuator endpoints in production
