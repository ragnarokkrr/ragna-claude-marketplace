---
name: spring-docker
description: Generate optimized Dockerfiles and Kubernetes manifests for Spring Boot applications
allowed-tools: ["Read", "Edit", "Write"]
---

# Spring Boot Docker & Kubernetes Skill

This skill generates production-ready Docker images and Kubernetes manifests for Spring Boot applications with multi-stage builds, layered JARs, security hardening, and cloud-native best practices.

## Purpose

Create containerization and deployment artifacts:
- **Multi-stage Dockerfiles** with build optimization
- **Layered JAR support** for better caching
- **Minimal base images** (Alpine, Distroless)
- **Security hardening** (non-root user, security scanning)
- **Kubernetes manifests** (Deployment, Service, Ingress, HPA)
- **Health check configuration** for liveness and readiness
- **Resource limits** and auto-scaling setup

## Instructions

When this skill is invoked:

1. **Determine deployment requirements:**
   - Ask about JDK version (17, 21, 24)
   - Identify build tool: Maven or Gradle
   - Check if native image (GraalVM) is needed
   - Confirm Kubernetes features: HPA, Ingress, ConfigMaps

2. **Generate Docker artifacts:**
   - Multi-stage Dockerfile with build and runtime stages
   - .dockerignore file to exclude unnecessary files
   - Optional: docker-compose.yml for local development

3. **Generate Kubernetes manifests:**
   - Deployment with health checks and resource limits
   - Service (ClusterIP, LoadBalancer, or NodePort)
   - Optional: Ingress, HPA, ConfigMap, Secret

4. **Provide deployment instructions:**
   - Build commands
   - Push to registry
   - Deploy to Kubernetes

## Docker Templates

### Multi-Stage Dockerfile (JDK 17+ with Maven)

```dockerfile
# Build stage
FROM eclipse-temurin:17-jdk-alpine AS build
WORKDIR /app

# Copy Maven wrapper and pom.xml first (better caching)
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .

# Download dependencies (cached layer)
RUN ./mvnw dependency:go-offline -B

# Copy source code
COPY src src

# Build application
RUN ./mvnw package -DskipTests -B
RUN mkdir -p target/dependency && (cd target/dependency; jar -xf ../*.jar)

# Production stage
FROM eclipse-temurin:17-jre-alpine
LABEL maintainer="your-team@example.com"
LABEL version="1.0.0"

# Security: Create non-root user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

# Create volume for temporary files
VOLUME /tmp

# Copy exploded JAR layers for better caching
WORKDIR /app
ARG DEPENDENCY=/app/target/dependency
COPY --from=build ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=build ${DEPENDENCY}/META-INF /app/META-INF
COPY --from=build ${DEPENDENCY}/BOOT-INF/classes /app

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health/liveness || exit 1

# Expose application port
EXPOSE 8080

# Run application with optimized JVM flags
ENTRYPOINT ["java", \
    "-XX:+UseContainerSupport", \
    "-XX:MaxRAMPercentage=75.0", \
    "-XX:+ExitOnOutOfMemoryError", \
    "-Djava.security.egd=file:/dev/./urandom", \
    "-cp", "app:app/lib/*", \
    "com.example.service.ServiceApplication"]
```

### Multi-Stage Dockerfile (JDK 21+ with Gradle)

```dockerfile
# Build stage
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app

# Copy Gradle wrapper and build files
COPY gradlew .
COPY gradle gradle
COPY build.gradle settings.gradle ./

# Download dependencies (cached layer)
RUN ./gradlew dependencies --no-daemon

# Copy source code
COPY src src

# Build application
RUN ./gradlew bootJar --no-daemon
RUN mkdir -p build/dependency && (cd build/dependency; jar -xf ../libs/*.jar)

# Production stage
FROM eclipse-temurin:21-jre-alpine
LABEL maintainer="your-team@example.com"
LABEL version="1.0.0"

# Security: Create non-root user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

VOLUME /tmp

WORKDIR /app
ARG DEPENDENCY=/app/build/dependency
COPY --from=build ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=build ${DEPENDENCY}/META-INF /app/META-INF
COPY --from=build ${DEPENDENCY}/BOOT-INF/classes /app

HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health/liveness || exit 1

EXPOSE 8080

# JDK 21+ with virtual threads optimizations
ENTRYPOINT ["java", \
    "-XX:+UseContainerSupport", \
    "-XX:MaxRAMPercentage=75.0", \
    "-XX:+UseZGC", \
    "-XX:+ExitOnOutOfMemoryError", \
    "-Djava.security.egd=file:/dev/./urandom", \
    "-cp", "app:app/lib/*", \
    "com.example.service.ServiceApplication"]
```

### Dockerfile with Spring Boot Layered JAR

```dockerfile
# Build stage
FROM eclipse-temurin:17-jdk-alpine AS build
WORKDIR /app

COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
RUN ./mvnw dependency:go-offline -B

COPY src src
RUN ./mvnw package -DskipTests -B

# Extract Spring Boot layers
RUN java -Djarmode=layertools -jar target/*.jar extract

# Production stage
FROM eclipse-temurin:17-jre-alpine

RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

VOLUME /tmp
WORKDIR /app

# Copy layers in order of least to most frequently changing
COPY --from=build /app/dependencies/ ./
COPY --from=build /app/spring-boot-loader/ ./
COPY --from=build /app/snapshot-dependencies/ ./
COPY --from=build /app/application/ ./

HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health/liveness || exit 1

EXPOSE 8080

ENTRYPOINT ["java", \
    "-XX:+UseContainerSupport", \
    "-XX:MaxRAMPercentage=75.0", \
    "org.springframework.boot.loader.JarLauncher"]
```

### Dockerfile with GraalVM Native Image

```dockerfile
# Build stage with GraalVM
FROM ghcr.io/graalvm/graalvm-ce:ol9-java17-22.3.3 AS build
WORKDIR /app

# Install native-image
RUN gu install native-image

COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
RUN ./mvnw dependency:go-offline -B

COPY src src

# Build native image (takes 5-10 minutes)
RUN ./mvnw -Pnative native:compile -DskipTests

# Production stage - distroless for minimal attack surface
FROM gcr.io/distroless/base-debian11

COPY --from=build /app/target/service-name /app/service-name

EXPOSE 8080

# Native image - no JVM needed
ENTRYPOINT ["/app/service-name"]
```

### Dockerfile for Multi-Architecture Builds

```dockerfile
# Supports linux/amd64, linux/arm64, linux/arm/v7
FROM --platform=$BUILDPLATFORM eclipse-temurin:17-jdk-alpine AS build
ARG TARGETPLATFORM
ARG BUILDPLATFORM

WORKDIR /app

COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
RUN ./mvnw dependency:go-offline -B

COPY src src
RUN ./mvnw package -DskipTests -B
RUN mkdir -p target/dependency && (cd target/dependency; jar -xf ../*.jar)

FROM eclipse-temurin:17-jre-alpine

RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

VOLUME /tmp
WORKDIR /app

ARG DEPENDENCY=/app/target/dependency
COPY --from=build ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=build ${DEPENDENCY}/META-INF /app/META-INF
COPY --from=build ${DEPENDENCY}/BOOT-INF/classes /app

HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health/liveness || exit 1

EXPOSE 8080

ENTRYPOINT ["java", \
    "-XX:+UseContainerSupport", \
    "-XX:MaxRAMPercentage=75.0", \
    "-cp", "app:app/lib/*", \
    "com.example.service.ServiceApplication"]
```

## .dockerignore

```
# Ignore version control
.git
.gitignore

# Ignore IDE files
.idea
.vscode
*.iml
.classpath
.project
.settings

# Ignore build artifacts (will be built in container)
target/
build/
*.jar
*.war

# Ignore test files
**/test/
**/*Test.java
**/*Tests.java

# Ignore documentation
*.md
docs/

# Ignore CI/CD files
.github
.gitlab-ci.yml
Jenkinsfile

# Ignore local environment files
.env
.env.local
application-local.yml

# Ignore logs
*.log
logs/

# Ignore OS files
.DS_Store
Thumbs.db
```

## Kubernetes Manifests

### Deployment with Health Checks and Resources

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: service-name
  namespace: default
  labels:
    app: service-name
    version: v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: service-name
  template:
    metadata:
      labels:
        app: service-name
        version: v1
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/actuator/prometheus"
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000

      containers:
      - name: service-name
        image: your-registry/service-name:1.0.0
        imagePullPolicy: Always

        ports:
        - name: http
          containerPort: 8080
          protocol: TCP

        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: JAVA_TOOL_OPTIONS
          value: "-XX:MaxRAMPercentage=75.0 -XX:+UseContainerSupport"
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: service-config
              key: db-host
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password

        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
            ephemeral-storage: "1Gi"
          limits:
            memory: "1Gi"
            cpu: "1000m"
            ephemeral-storage: "2Gi"

        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: http
          initialDelaySeconds: 60
          periodSeconds: 10
          timeoutSeconds: 3
          failureThreshold: 3

        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: http
          initialDelaySeconds: 30
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3

        startupProbe:
          httpGet:
            path: /actuator/health/liveness
            port: http
          initialDelaySeconds: 0
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 30

        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: logs
          mountPath: /app/logs

      volumes:
      - name: tmp
        emptyDir: {}
      - name: logs
        emptyDir: {}

      restartPolicy: Always
      terminationGracePeriodSeconds: 30
```

### Service (ClusterIP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-name
  namespace: default
  labels:
    app: service-name
spec:
  type: ClusterIP
  selector:
    app: service-name
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: http
  sessionAffinity: None
```

### Ingress (with TLS)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: service-name
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - api.example.com
    secretName: service-name-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service-name
            port:
              name: http
```

### Horizontal Pod Autoscaler (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: service-name-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: service-name
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
```

### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: service-config
  namespace: default
data:
  db-host: "postgres.database.svc.cluster.local"
  db-port: "5432"
  db-name: "servicedb"
  redis-host: "redis.cache.svc.cluster.local"
  redis-port: "6379"
  application.yml: |
    logging:
      level:
        root: INFO
        com.example: DEBUG
```

### Secret (example - use sealed secrets or external secrets in production)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
  namespace: default
type: Opaque
stringData:
  username: "service_user"
  password: "CHANGE_ME"  # Use external secrets manager in production
```

## Docker Compose for Local Development

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=dev
      - DB_HOST=postgres
      - DB_USERNAME=postgres
      - DB_PASSWORD=postgres
      - REDIS_HOST=redis
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - app-network

  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=servicedb
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

## Build and Deployment Commands

### Docker Build Commands

```bash
# Build standard image
docker build -t your-registry/service-name:1.0.0 .

# Build with build arguments
docker build \
  --build-arg JAR_FILE=target/service-name-1.0.0.jar \
  -t your-registry/service-name:1.0.0 .

# Build multi-architecture image
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t your-registry/service-name:1.0.0 \
  --push .

# Build with cache
docker build \
  --cache-from your-registry/service-name:latest \
  -t your-registry/service-name:1.0.0 .
```

### Docker Push Commands

```bash
# Tag and push
docker tag service-name:1.0.0 your-registry/service-name:1.0.0
docker push your-registry/service-name:1.0.0

# Push latest
docker tag service-name:1.0.0 your-registry/service-name:latest
docker push your-registry/service-name:latest
```

### Kubernetes Deployment Commands

```bash
# Apply manifests
kubectl apply -f k8s/

# Or apply individually
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
kubectl apply -f k8s/hpa.yaml

# Check deployment
kubectl rollout status deployment/service-name

# View pods
kubectl get pods -l app=service-name

# View logs
kubectl logs -f deployment/service-name

# Scale manually
kubectl scale deployment service-name --replicas=5

# Update image
kubectl set image deployment/service-name \
  service-name=your-registry/service-name:1.0.1
```

## Best Practices

### Docker Best Practices
- ✓ Use multi-stage builds to minimize image size
- ✓ Leverage layer caching (copy dependencies before source)
- ✓ Run as non-root user for security
- ✓ Use specific base image versions (not `latest`)
- ✓ Include health checks in Dockerfile
- ✓ Optimize JVM flags for containers
- ✓ Use .dockerignore to exclude unnecessary files
- ✓ Label images with metadata (maintainer, version)

### Kubernetes Best Practices
- ✓ Always define resource requests and limits
- ✓ Use readiness and liveness probes
- ✓ Add startup probe for slow-starting apps
- ✓ Configure HPA for auto-scaling
- ✓ Use ConfigMaps for configuration, Secrets for credentials
- ✓ Enable rolling updates with zero downtime
- ✓ Set appropriate termination grace period (30s+)
- ✓ Use labels and selectors consistently
- ✓ Configure pod security policies
- ✓ Enable Prometheus scraping annotations

### Security Hardening
- ✓ Scan images for vulnerabilities (Trivy, Snyk, Grype)
- ✓ Use minimal base images (Alpine, Distroless)
- ✓ Run as non-root user (UID 1000+)
- ✓ Set read-only root filesystem where possible
- ✓ Drop all capabilities, add only what's needed
- ✓ Use network policies to restrict traffic
- ✓ Keep base images and dependencies updated
- ✓ Never include secrets in images

## Output Format

When invoked, provide:
1. Dockerfile (multi-stage, appropriate for JDK version and build tool)
2. .dockerignore file
3. Kubernetes manifests (Deployment, Service, and optional Ingress/HPA)
4. ConfigMap and Secret templates
5. Docker Compose for local development (optional)
6. Build and deployment commands
7. Best practices checklist
8. File placement instructions (Dockerfile at root, k8s/ directory for manifests)

## Common Scenarios

**Scenario 1: Basic REST API with PostgreSQL**
- Multi-stage Dockerfile with JDK 17
- Kubernetes Deployment with health checks
- Service (ClusterIP) for internal access
- ConfigMap for database connection

**Scenario 2: High-traffic microservice with HPA**
- Layered JAR Dockerfile for fast rebuilds
- Deployment with 3-10 replicas
- HPA based on CPU and memory
- Ingress with rate limiting

**Scenario 3: Native image for fast startup**
- GraalVM Dockerfile with native-image
- Reduced resource limits (256MB memory)
- Startup time < 100ms
- Distroless base image for security
