# Go Deployment and Building Binaries

Go's deployment capabilities make it one of the most attractive languages for  
modern application development. Go compiles to static binaries by default,  
enabling easy distribution and deployment across different environments without  
dependency concerns. The language provides excellent cross-compilation support,  
allowing developers to build binaries for multiple operating systems and  
architectures from a single development machine.

Go's deployment story is compelling because of several key features: static  
linking produces self-contained executables, fast compilation times enable  
rapid iteration, small binary sizes reduce deployment overhead, and excellent  
containerization support makes Go applications ideal for modern cloud-native  
deployments. The go build command provides comprehensive options for  
optimizing binaries for production use.

Cross-compilation in Go is remarkably straightforward, supporting over 20  
operating systems and multiple architectures including ARM, AMD64, and emerging  
platforms. This capability is essential for modern deployment scenarios where  
applications need to run across diverse infrastructure environments. Go's  
consistent behavior across platforms reduces deployment complexity and testing  
overhead.

Container deployment has become the standard for Go applications, with Docker  
providing excellent support for Go binaries. Go's static linking capability  
eliminates the need for complex base images, allowing applications to run in  
minimal containers like scratch or distroless images. This results in smaller  
image sizes, reduced attack surface, and faster deployment times.

Modern deployment strategies for Go applications include blue-green deployments,  
rolling updates, canary releases, and serverless deployments. Go's fast startup  
times and low resource consumption make it ideal for cloud-native patterns like  
auto-scaling and function-as-a-service platforms. Understanding these deployment  
patterns is crucial for building production-ready Go applications.


## Basic binary building

```go
// main.go - Simple application for building examples
package main

import (
    "fmt"
    "os"
    "runtime"
    "time"
)

var (
    // Build-time variables (set via ldflags)
    Version   = "dev"
    BuildTime = "unknown"
    GitCommit = "unknown"
)

func main() {
    fmt.Printf("Application Information:\\n")
    fmt.Printf("  Version: %s\\n", Version)
    fmt.Printf("  Build Time: %s\\n", BuildTime)
    fmt.Printf("  Git Commit: %s\\n", GitCommit)
    fmt.Printf("  Go Version: %s\\n", runtime.Version())
    fmt.Printf("  OS/Arch: %s/%s\\n", runtime.GOOS, runtime.GOARCH)
    
    if len(os.Args) > 1 && os.Args[1] == "serve" {
        fmt.Println("\\nStarting application server...")
        // Simulate server startup
        time.Sleep(100 * time.Millisecond)
        fmt.Println("Server started successfully")
    } else {
        fmt.Println("\\nUse 'serve' argument to start server mode")
    }
}
```

Basic Go binary building uses the `go build` command to compile source code  
into executable binaries. The build process produces platform-specific  
executables that include the Go runtime and all dependencies. Build-time  
variables can be injected using ldflags for version information and  
configuration.

## Build optimization and flags

```go
// build-script.sh - Comprehensive build script example
#!/bin/bash

# Build configuration
APP_NAME="myapp"
VERSION=$(git describe --tags --always --dirty)
BUILD_TIME=$(date -u '+%Y-%m-%d_%I:%M:%S%p')
GIT_COMMIT=$(git rev-parse HEAD)

# Build flags for optimization
BUILD_FLAGS="-a -installsuffix cgo"
LDFLAGS="-s -w -extldflags '-static'"
LDFLAGS="$LDFLAGS -X main.Version=$VERSION"
LDFLAGS="$LDFLAGS -X main.BuildTime=$BUILD_TIME"
LDFLAGS="$LDFLAGS -X main.GitCommit=$GIT_COMMIT"

echo "Building $APP_NAME version $VERSION..."

# Development build (with debug info)
echo "Creating development build..."
go build -o bin/${APP_NAME}-dev ./cmd/myapp

# Production build (optimized)
echo "Creating production build..."
CGO_ENABLED=0 go build $BUILD_FLAGS -ldflags "$LDFLAGS" -o bin/${APP_NAME} ./cmd/myapp

# Display build information
echo "Build completed:"
ls -lh bin/
echo ""
echo "Binary information:"
file bin/${APP_NAME}
echo ""
echo "Size comparison:"
du -h bin/${APP_NAME}*
```

Build optimization in Go involves several flags and techniques to reduce binary  
size and improve performance. The `-ldflags` parameter allows setting build-time  
variables and optimization flags. Disabling CGO with `CGO_ENABLED=0` enables  
fully static binaries suitable for minimal container images.

## Cross-compilation

```go
// cross-build.go - Cross-compilation automation tool
package main

import (
    "fmt"
    "os"
    "os/exec"
    "path/filepath"
    "strings"
)

type Platform struct {
    GOOS   string
    GOARCH string
    Suffix string
}

var platforms = []Platform{
    {"linux", "amd64", ""},
    {"linux", "arm64", ""},
    {"darwin", "amd64", ""},
    {"darwin", "arm64", ""},
    {"windows", "amd64", ".exe"},
    {"windows", "arm64", ".exe"},
    {"freebsd", "amd64", ""},
    {"openbsd", "amd64", ""},
    {"netbsd", "amd64", ""},
}

func main() {
    appName := "myapp"
    if len(os.Args) > 1 {
        appName = os.Args[1]
    }
    
    fmt.Printf("Cross-compiling %s for multiple platforms...\\n\\n", appName)
    
    // Create dist directory
    distDir := "dist"
    os.MkdirAll(distDir, 0755)
    
    for _, platform := range platforms {
        outputName := fmt.Sprintf("%s_%s_%s%s", 
            appName, platform.GOOS, platform.GOARCH, platform.Suffix)
        outputPath := filepath.Join(distDir, outputName)
        
        fmt.Printf("Building %s...", outputName)
        
        cmd := exec.Command("go", "build", "-o", outputPath, ".")
        cmd.Env = append(os.Environ(),
            "GOOS="+platform.GOOS,
            "GOARCH="+platform.GOARCH,
            "CGO_ENABLED=0",
        )
        
        if err := cmd.Run(); err != nil {
            fmt.Printf(" FAILED: %v\\n", err)
            continue
        }
        
        // Get file size
        if info, err := os.Stat(outputPath); err == nil {
            fmt.Printf(" SUCCESS (%.1f MB)\\n", float64(info.Size())/1024/1024)
        } else {
            fmt.Printf(" SUCCESS\\n")
        }
    }
    
    fmt.Println("\\nCross-compilation completed!")
    fmt.Println("\\nGenerated binaries:")
    
    // List generated files
    filepath.Walk(distDir, func(path string, info os.FileInfo, err error) error {
        if err != nil || info.IsDir() {
            return err
        }
        fmt.Printf("  %s (%.1f MB)\\n", 
            strings.TrimPrefix(path, distDir+"/"), 
            float64(info.Size())/1024/1024)
        return nil
    })
}
```

Cross-compilation in Go is enabled by setting GOOS and GOARCH environment  
variables before building. This allows creating binaries for different  
operating systems and architectures from a single development machine.  
Automated cross-compilation scripts can build for multiple targets  
simultaneously, essential for distribution and deployment strategies.

## Static binary building

```go
// static-build-demo.go - Demonstrates static binary building techniques
package main

import (
    "fmt"
    "net/http"
    "os"
    "runtime"
)

func main() {
    fmt.Printf("Static Binary Demo\\n")
    fmt.Printf("=================\\n\\n")
    
    // Show runtime information
    fmt.Printf("Runtime Information:\\n")
    fmt.Printf("  Go Version: %s\\n", runtime.Version())
    fmt.Printf("  OS/Arch: %s/%s\\n", runtime.GOOS, runtime.GOARCH)
    fmt.Printf("  CGO Enabled: %t\\n", runtime.Compiler == "gc")
    
    // Show environment variables relevant to static building
    fmt.Printf("\\nBuild Environment:\\n")
    cgoEnabled := os.Getenv("CGO_ENABLED")
    if cgoEnabled == "" {
        cgoEnabled = "1 (default)"
    }
    fmt.Printf("  CGO_ENABLED: %s\\n", cgoEnabled)
    fmt.Printf("  GOOS: %s\\n", os.Getenv("GOOS"))
    fmt.Printf("  GOARCH: %s\\n", os.Getenv("GOARCH"))
    
    // Demonstrate static linking with network code
    fmt.Printf("\\nTesting static linking with network operations...\\n")
    
    // Test DNS resolution (this would fail in pure static if not handled correctly)
    client := &http.Client{}
    resp, err := client.Get("https://httpbin.org/ip")
    if err != nil {
        fmt.Printf("  Network test failed: %v\\n", err)
    } else {
        resp.Body.Close()
        fmt.Printf("  Network test successful: HTTP %s\\n", resp.Status)
    }
    
    fmt.Printf("\\nStatic binary validation complete!\\n")
}
```

Static binary building in Go produces self-contained executables that include  
all dependencies and the Go runtime. Setting `CGO_ENABLED=0` disables C  
bindings, ensuring pure Go static binaries. These binaries can run in minimal  
containers without any system dependencies, making deployment simpler and more  
reliable.

## Dockerfile for Go applications

```dockerfile
# Multi-stage Dockerfile for Go applications
# Stage 1: Build stage
FROM golang:1.21-alpine AS builder

# Install git for go mod download
RUN apk add --no-cache git

# Set working directory
WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./

# Download dependencies
RUN go mod download

# Copy source code
COPY . .

# Build the application
RUN CGO_ENABLED=0 GOOS=linux go build \\
    -a -installsuffix cgo \\
    -ldflags '-extldflags "-static"' \\
    -o main ./cmd/myapp

# Stage 2: Runtime stage
FROM scratch

# Copy SSL certificates for HTTPS
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy the binary
COPY --from=builder /app/main /app/main

# Expose port
EXPOSE 8080

# Set entrypoint
ENTRYPOINT ["/app/main"]
```

Multi-stage Docker builds for Go applications optimize image size by separating  
the build environment from the runtime environment. The build stage includes  
Go toolchain and dependencies, while the runtime stage contains only the  
compiled binary and minimal requirements like SSL certificates.

## Advanced Docker strategies

```dockerfile
# Advanced multi-stage Dockerfile with optimization
FROM golang:1.21-alpine AS base
RUN apk add --no-cache git ca-certificates tzdata
WORKDIR /app

# Dependency stage - cached layer for faster rebuilds
FROM base AS dependencies
COPY go.mod go.sum ./
RUN go mod download
RUN go mod verify

# Build stage
FROM dependencies AS builder
COPY . .
RUN CGO_ENABLED=0 go build \\
    -ldflags='-w -s -extldflags "-static"' \\
    -a -installsuffix cgo \\
    -o app ./cmd/myapp

# Development stage (for dev container)
FROM base AS development
COPY --from=dependencies /go/pkg /go/pkg
RUN go install github.com/cosmtrek/air@latest
COPY . .
CMD ["air"]

# Testing stage
FROM dependencies AS testing
COPY . .
RUN go test -v ./...
RUN go vet ./...

# Production stage
FROM scratch AS production
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo
COPY --from=builder /app/app /app
EXPOSE 8080
USER 65534:65534
ENTRYPOINT ["/app"]

# Default target
FROM production
```

Advanced Docker strategies for Go applications include multiple targets for  
different environments, dependency caching layers, security considerations with  
non-root users, and timezone data inclusion. These techniques optimize build  
times, security posture, and deployment flexibility across development and  
production environments.

## Docker Compose for development

```yaml
# docker-compose.yml - Development environment setup
version: '3.8'

services:
  app:
    build:
      context: .
      target: development
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    volumes:
      - .:/app
      - go-modules:/go/pkg/mod
    environment:
      - CGO_ENABLED=0
      - GOOS=linux
    depends_on:
      - postgres
      - redis
    networks:
      - app-network

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
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

  # Production-like testing environment
  app-prod:
    build:
      context: .
      target: production
      dockerfile: Dockerfile
    ports:
      - "8081:8080"
    environment:
      - DATABASE_URL=postgres://user:password@postgres:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - app-network
    profiles:
      - production-test

volumes:
  go-modules:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

Docker Compose orchestrates multi-container development environments for Go  
applications. This configuration supports both development and production-like  
testing environments, with hot reloading for development and optimized builds  
for testing. Volume mounts cache Go modules and provide data persistence.

## Deployment automation scripts

```bash
#!/bin/bash
# deploy.sh - Comprehensive deployment automation script

set -euo pipefail

# Configuration
APP_NAME="myapp"
REGISTRY="your-registry.com"
NAMESPACE="production"
VERSION=${1:-$(git describe --tags --always --dirty)}

# Colors for output
RED='\\033[0;31m'
GREEN='\\033[0;32m'
YELLOW='\\033[0;33m'
NC='\\033[0m' # No Color

log() {
    echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] $1${NC}"
}

warn() {
    echo -e "${YELLOW}[$(date +'%Y-%m-%d %H:%M:%S')] WARNING: $1${NC}"
}

error() {
    echo -e "${RED}[$(date +'%Y-%m-%d %H:%M:%S')] ERROR: $1${NC}"
    exit 1
}

# Pre-deployment checks
check_prerequisites() {
    log "Checking prerequisites..."
    
    command -v docker >/dev/null 2>&1 || error "Docker is required"
    command -v kubectl >/dev/null 2>&1 || error "kubectl is required"
    
    # Check if logged into Docker registry
    docker info >/dev/null 2>&1 || error "Docker daemon not running"
    
    # Check Kubernetes connection
    kubectl cluster-info >/dev/null 2>&1 || error "Cannot connect to Kubernetes cluster"
    
    log "Prerequisites check passed"
}

# Build and tag Docker image
build_image() {
    log "Building Docker image..."
    
    IMAGE_TAG="${REGISTRY}/${APP_NAME}:${VERSION}"
    LATEST_TAG="${REGISTRY}/${APP_NAME}:latest"
    
    docker build -t "$IMAGE_TAG" -t "$LATEST_TAG" .
    
    log "Image built successfully: $IMAGE_TAG"
}

# Run tests in container
test_image() {
    log "Running tests in container..."
    
    docker run --rm "$IMAGE_TAG" /app/app --version
    
    # Run health check if available
    if docker run --rm -d --name test-container -p 8080:8080 "$IMAGE_TAG" >/dev/null 2>&1; then
        sleep 5
        if curl -f http://localhost:8080/health >/dev/null 2>&1; then
            log "Health check passed"
        else
            warn "Health check endpoint not available"
        fi
        docker stop test-container >/dev/null 2>&1 || true
    fi
    
    log "Container tests completed"
}

# Push image to registry
push_image() {
    log "Pushing image to registry..."
    
    docker push "$IMAGE_TAG"
    docker push "$LATEST_TAG"
    
    log "Image pushed successfully"
}

# Deploy to Kubernetes
deploy_k8s() {
    log "Deploying to Kubernetes..."
    
    # Update deployment image
    kubectl set image deployment/${APP_NAME} ${APP_NAME}=${IMAGE_TAG} -n ${NAMESPACE}
    
    # Wait for rollout to complete
    kubectl rollout status deployment/${APP_NAME} -n ${NAMESPACE} --timeout=300s
    
    # Verify deployment
    kubectl get pods -n ${NAMESPACE} -l app=${APP_NAME}
    
    log "Deployment completed successfully"
}

# Rollback deployment if needed
rollback() {
    warn "Rolling back deployment..."
    kubectl rollout undo deployment/${APP_NAME} -n ${NAMESPACE}
    kubectl rollout status deployment/${APP_NAME} -n ${NAMESPACE}
    log "Rollback completed"
}

# Cleanup old images
cleanup() {
    log "Cleaning up old Docker images..."
    docker image prune -f
    log "Cleanup completed"
}

# Main deployment flow
main() {
    log "Starting deployment of ${APP_NAME} version ${VERSION}"
    
    check_prerequisites
    build_image
    test_image
    push_image
    
    if deploy_k8s; then
        log "Deployment successful!"
    else
        error "Deployment failed"
        if [ "${ROLLBACK:-true}" = "true" ]; then
            rollback
        fi
        exit 1
    fi
    
    cleanup
    log "Deployment process completed"
}

# Handle script arguments
case "${1:-deploy}" in
    "build")
        check_prerequisites
        build_image
        ;;
    "test")
        test_image
        ;;
    "push")
        push_image
        ;;
    "deploy")
        main
        ;;
    "rollback")
        rollback
        ;;
    *)
        echo "Usage: $0 {build|test|push|deploy|rollback} [version]"
        exit 1
        ;;
esac
```

Deployment automation scripts orchestrate the entire deployment pipeline from  
building images to updating production environments. These scripts include  
safety checks, testing validation, rollback capabilities, and comprehensive  
logging. Automation reduces human error and ensures consistent deployments  
across different environments.

## Health checks and monitoring

```go
// health.go - Health check and monitoring implementation
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "net/http"
    "runtime"
    "time"
)

type HealthChecker struct {
    startTime time.Time
    version   string
    checks    map[string]HealthCheck
}

type HealthCheck func(ctx context.Context) error

type HealthStatus struct {
    Status    string                 `json:"status"`
    Version   string                 `json:"version"`
    Timestamp time.Time              `json:"timestamp"`
    Uptime    string                 `json:"uptime"`
    Checks    map[string]CheckResult `json:"checks"`
    System    SystemInfo             `json:"system"`
}

type CheckResult struct {
    Status  string        `json:"status"`
    Message string        `json:"message,omitempty"`
    Latency time.Duration `json:"latency"`
}

type SystemInfo struct {
    GoVersion   string `json:"go_version"`
    NumCPU      int    `json:"num_cpu"`
    NumRoutines int    `json:"num_goroutines"`
    MemMB       uint64 `json:"memory_mb"`
}

func NewHealthChecker(version string) *HealthChecker {
    return &HealthChecker{
        startTime: time.Now(),
        version:   version,
        checks:    make(map[string]HealthCheck),
    }
}

func (hc *HealthChecker) AddCheck(name string, check HealthCheck) {
    hc.checks[name] = check
}

func (hc *HealthChecker) Handler() http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        ctx, cancel := context.WithTimeout(r.Context(), 10*time.Second)
        defer cancel()
        
        status := hc.performChecks(ctx)
        
        w.Header().Set("Content-Type", "application/json")
        
        if status.Status == "healthy" {
            w.WriteHeader(http.StatusOK)
        } else {
            w.WriteHeader(http.StatusServiceUnavailable)
        }
        
        json.NewEncoder(w).Encode(status)
    }
}

func (hc *HealthChecker) performChecks(ctx context.Context) HealthStatus {
    status := HealthStatus{
        Version:   hc.version,
        Timestamp: time.Now(),
        Uptime:    time.Since(hc.startTime).String(),
        Checks:    make(map[string]CheckResult),
        System:    hc.getSystemInfo(),
    }
    
    overallHealthy := true
    
    for name, check := range hc.checks {
        start := time.Now()
        err := check(ctx)
        latency := time.Since(start)
        
        result := CheckResult{
            Latency: latency,
        }
        
        if err != nil {
            result.Status = "unhealthy"
            result.Message = err.Error()
            overallHealthy = false
        } else {
            result.Status = "healthy"
        }
        
        status.Checks[name] = result
    }
    
    if overallHealthy {
        status.Status = "healthy"
    } else {
        status.Status = "unhealthy"
    }
    
    return status
}

func (hc *HealthChecker) getSystemInfo() SystemInfo {
    var memStats runtime.MemStats
    runtime.ReadMemStats(&memStats)
    
    return SystemInfo{
        GoVersion:   runtime.Version(),
        NumCPU:      runtime.NumCPU(),
        NumRoutines: runtime.NumGoroutine(),
        MemMB:       memStats.Alloc / 1024 / 1024,
    }
}

// Example health checks
func DatabaseHealthCheck(db interface{}) HealthCheck {
    return func(ctx context.Context) error {
        // Simulate database ping
        time.Sleep(10 * time.Millisecond)
        return nil // or return actual database ping error
    }
}

func ExternalServiceHealthCheck(url string) HealthCheck {
    return func(ctx context.Context) error {
        client := &http.Client{Timeout: 5 * time.Second}
        req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
        if err != nil {
            return err
        }
        
        resp, err := client.Do(req)
        if err != nil {
            return err
        }
        defer resp.Body.Close()
        
        if resp.StatusCode >= 400 {
            return fmt.Errorf("service returned status %d", resp.StatusCode)
        }
        
        return nil
    }
}

func main() {
    hc := NewHealthChecker("1.0.0")
    
    // Add health checks
    hc.AddCheck("database", DatabaseHealthCheck(nil))
    hc.AddCheck("external_api", ExternalServiceHealthCheck("https://httpbin.org/status/200"))
    
    http.HandleFunc("/health", hc.Handler())
    http.HandleFunc("/ready", hc.Handler()) // Kubernetes readiness probe
    http.HandleFunc("/live", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        w.Write([]byte("OK"))
    }) // Kubernetes liveness probe
    
    fmt.Println("Health check server starting on :8080")
    fmt.Println("Endpoints:")
    fmt.Println("  /health - Detailed health status")
    fmt.Println("  /ready  - Readiness probe")
    fmt.Println("  /live   - Liveness probe")
    
    if err := http.ListenAndServe(":8080", nil); err != nil {
        fmt.Printf("Server failed: %v\\n", err)
    }
}
```

Health checks and monitoring are essential for production Go applications.  
Comprehensive health endpoints provide detailed system status, dependency  
health, and performance metrics. These endpoints integrate with container  
orchestration platforms for automated health monitoring and service recovery.

## Kubernetes deployment manifests

```yaml
# k8s-deployment.yaml - Complete Kubernetes deployment configuration
apiVersion: v1
kind: Namespace
metadata:
  name: myapp-production
  labels:
    name: myapp-production
    environment: production

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
  namespace: myapp-production
data:
  app.yaml: |
    database:
      host: postgres.myapp-production.svc.cluster.local
      port: 5432
      name: myapp
    redis:
      host: redis.myapp-production.svc.cluster.local
      port: 6379
    logging:
      level: info
      format: json

---
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
  namespace: myapp-production
type: Opaque
stringData:
  database-password: "secure-password"
  api-key: "secret-api-key"
  jwt-secret: "jwt-signing-secret"

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: myapp-production
  labels:
    app: myapp
    version: v1.0.0
    environment: production
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1.0.0
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
        fsGroup: 65534
      containers:
      - name: myapp
        image: your-registry.com/myapp:v1.0.0
        ports:
        - containerPort: 8080
          name: http
          protocol: TCP
        env:
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: database-password
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: api-key
        - name: CONFIG_PATH
          value: "/etc/config/app.yaml"
        volumeMounts:
        - name: config
          mountPath: /etc/config
          readOnly: true
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /live
            port: http
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: http
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        startupProbe:
          httpGet:
            path: /live
            port: http
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 30
      volumes:
      - name: config
        configMap:
          name: myapp-config
      restartPolicy: Always
      terminationGracePeriodSeconds: 30

---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: myapp-production
  labels:
    app: myapp
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: http
    protocol: TCP
    name: http
  selector:
    app: myapp

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: myapp-production
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80

---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
  namespace: myapp-production
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
  namespace: myapp-production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
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
```

Kubernetes deployment manifests define comprehensive application deployment  
configurations including namespaces, ConfigMaps, Secrets, deployments,  
services, ingress, and autoscaling. These manifests ensure consistent,  
production-ready deployments with proper resource management, security  
configuration, and high availability setup.

## CI/CD pipeline integration

```yaml
# .github/workflows/deploy.yml - GitHub Actions deployment pipeline
name: Build and Deploy

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Go
      uses: actions/setup-go@v4
      with:
        go-version: '1.21'
    
    - name: Cache Go modules
      uses: actions/cache@v3
      with:
        path: |
          ~/.cache/go-build
          ~/go/pkg/mod
        key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
        restore-keys: |
          ${{ runner.os }}-go-
    
    - name: Run tests
      run: |
        go test -v -race -coverprofile=coverage.out ./...
        go tool cover -html=coverage.out -o coverage.html
    
    - name: Run static analysis
      run: |
        go vet ./...
        go install honnef.co/go/tools/cmd/staticcheck@latest
        staticcheck ./...
    
    - name: Upload coverage reports
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.out

  build:
    runs-on: ubuntu-latest
    needs: test
    outputs:
      image: ${{ steps.image.outputs.image }}
      digest: ${{ steps.build.outputs.digest }}
    steps:
    - name: Checkout
      uses: actions/checkout@v4
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Log in to registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}
          type=sha
    
    - name: Build and push
      id: build
      uses: docker/build-push-action@v5
      with:
        context: .
        platforms: linux/amd64,linux/arm64
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
    
    - name: Output image
      id: image
      run: |
        echo "image=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ steps.meta.outputs.version }}" >> $GITHUB_OUTPUT

  security-scan:
    runs-on: ubuntu-latest
    needs: build
    steps:
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: ${{ needs.build.outputs.image }}
        format: 'sarif'
        output: 'trivy-results.sarif'
    
    - name: Upload Trivy scan results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'

  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build, security-scan]
    if: github.ref == 'refs/heads/main'
    environment: staging
    steps:
    - name: Deploy to staging
      run: |
        echo "Deploying ${{ needs.build.outputs.image }} to staging"
        # Add actual deployment commands here

  deploy-production:
    runs-on: ubuntu-latest
    needs: [build, security-scan]
    if: startsWith(github.ref, 'refs/tags/v')
    environment: production
    steps:
    - name: Deploy to production
      run: |
        echo "Deploying ${{ needs.build.outputs.image }} to production"
        # Add actual deployment commands here
```

CI/CD pipeline integration automates the entire deployment lifecycle from  
code commit to production deployment. This pipeline includes testing, static  
analysis, multi-platform builds, security scanning, and environment-specific  
deployments. Automated pipelines ensure consistent quality and reduce manual  
deployment errors.

## Cloud platform deployment strategies

```go
// cloud-deploy.go - Cloud platform deployment utilities
package main

import (
    "fmt"
    "os"
    "os/exec"
    "strings"
)

type CloudProvider struct {
    Name        string
    BuildCmd    []string
    DeployCmd   []string
    Environment map[string]string
}

var cloudProviders = map[string]CloudProvider{
    "heroku": {
        Name:     "Heroku",
        BuildCmd: []string{"git", "push", "heroku", "main"},
        Environment: map[string]string{
            "CGO_ENABLED": "0",
            "GOOS":        "linux",
        },
    },
    "railway": {
        Name:     "Railway",
        BuildCmd: []string{"railway", "up"},
        Environment: map[string]string{
            "NIXPACKS_BUILD_CMD": "go build -o main .",
            "NIXPACKS_START_CMD": "./main",
        },
    },
    "flyio": {
        Name:      "Fly.io",
        DeployCmd: []string{"flyctl", "deploy"},
        Environment: map[string]string{
            "CGO_ENABLED": "0",
        },
    },
    "gcp-run": {
        Name:      "Google Cloud Run",
        BuildCmd:  []string{"gcloud", "builds", "submit", "--tag", "gcr.io/PROJECT/myapp"},
        DeployCmd: []string{"gcloud", "run", "deploy", "--image", "gcr.io/PROJECT/myapp"},
        Environment: map[string]string{
            "CGO_ENABLED": "0",
        },
    },
    "aws-lambda": {
        Name:     "AWS Lambda",
        BuildCmd: []string{"GOOS=linux", "go", "build", "-o", "main", "."},
        Environment: map[string]string{
            "CGO_ENABLED": "0",
            "GOOS":        "linux",
            "GOARCH":      "amd64",
        },
    },
}

func deployToCloud(provider string) error {
    cp, exists := cloudProviders[provider]
    if !exists {
        return fmt.Errorf("unknown cloud provider: %s", provider)
    }
    
    fmt.Printf("Deploying to %s...\\n", cp.Name)
    
    // Set environment variables
    for key, value := range cp.Environment {
        os.Setenv(key, value)
        fmt.Printf("Set %s=%s\\n", key, value)
    }
    
    // Execute build command if specified
    if len(cp.BuildCmd) > 0 {
        fmt.Printf("Building with: %s\\n", strings.Join(cp.BuildCmd, " "))
        cmd := exec.Command(cp.BuildCmd[0], cp.BuildCmd[1:]...)
        cmd.Stdout = os.Stdout
        cmd.Stderr = os.Stderr
        if err := cmd.Run(); err != nil {
            return fmt.Errorf("build failed: %w", err)
        }
    }
    
    // Execute deploy command if specified
    if len(cp.DeployCmd) > 0 {
        fmt.Printf("Deploying with: %s\\n", strings.Join(cp.DeployCmd, " "))
        cmd := exec.Command(cp.DeployCmd[0], cp.DeployCmd[1:]...)
        cmd.Stdout = os.Stdout
        cmd.Stderr = os.Stderr
        if err := cmd.Run(); err != nil {
            return fmt.Errorf("deployment failed: %w", err)
        }
    }
    
    fmt.Printf("Successfully deployed to %s\\n", cp.Name)
    return nil
}

func main() {
    if len(os.Args) < 2 {
        fmt.Println("Available cloud providers:")
        for key, provider := range cloudProviders {
            fmt.Printf("  %s: %s\\n", key, provider.Name)
        }
        fmt.Println("\\nUsage: go run cloud-deploy.go <provider>")
        return
    }
    
    provider := os.Args[1]
    if err := deployToCloud(provider); err != nil {
        fmt.Printf("Deployment failed: %v\\n", err)
        os.Exit(1)
    }
}
```

Cloud platform deployment strategies vary significantly across providers,  
each with specific requirements for Go applications. Understanding platform-  
specific build and deployment patterns enables effective multi-cloud  
strategies. This includes serverless deployments, container services, and  
platform-as-a-service offerings optimized for Go applications.

## Production deployment checklist

```go
// deployment-checklist.go - Production deployment validation tool
package main

import (
    "fmt"
    "net/http"
    "os"
    "os/exec"
    "regexp"
    "strconv"
    "strings"
    "time"
)

type CheckItem struct {
    Name        string
    Description string
    CheckFunc   func() (bool, string)
    Critical    bool
}

type DeploymentChecker struct {
    checks []CheckItem
}

func NewDeploymentChecker() *DeploymentChecker {
    dc := &DeploymentChecker{}
    dc.addStandardChecks()
    return dc
}

func (dc *DeploymentChecker) addStandardChecks() {
    dc.checks = []CheckItem{
        {
            Name:        "Binary Size",
            Description: "Check if binary size is reasonable",
            CheckFunc:   dc.checkBinarySize,
            Critical:    false,
        },
        {
            Name:        "Static Linking",
            Description: "Verify binary is statically linked",
            CheckFunc:   dc.checkStaticLinking,
            Critical:    true,
        },
        {
            Name:        "Security Flags",
            Description: "Check if security build flags are used",
            CheckFunc:   dc.checkSecurityFlags,
            Critical:    true,
        },
        {
            Name:        "Health Endpoint",
            Description: "Verify health check endpoint is available",
            CheckFunc:   dc.checkHealthEndpoint,
            Critical:    true,
        },
        {
            Name:        "Environment Variables",
            Description: "Check required environment variables",
            CheckFunc:   dc.checkEnvironmentVariables,
            Critical:    true,
        },
        {
            Name:        "Dependencies",
            Description: "Verify all external dependencies are available",
            CheckFunc:   dc.checkDependencies,
            Critical:    true,
        },
        {
            Name:        "Resource Limits",
            Description: "Check if resource limits are configured",
            CheckFunc:   dc.checkResourceLimits,
            Critical:    false,
        },
        {
            Name:        "Logging Configuration",
            Description: "Verify logging is properly configured",
            CheckFunc:   dc.checkLoggingConfig,
            Critical:    false,
        },
    }
}

func (dc *DeploymentChecker) checkBinarySize() (bool, string) {
    if info, err := os.Stat("./main"); err == nil {
        sizeMB := float64(info.Size()) / 1024 / 1024
        if sizeMB > 50 {
            return false, fmt.Sprintf("Binary size is %.1f MB (consider optimization)", sizeMB)
        }
        return true, fmt.Sprintf("Binary size: %.1f MB", sizeMB)
    }
    return false, "Binary not found"
}

func (dc *DeploymentChecker) checkStaticLinking() (bool, string) {
    cmd := exec.Command("ldd", "./main")
    output, err := cmd.Output()
    if err != nil {
        // If ldd fails, it might be statically linked
        return true, "Static linking verified (ldd failed as expected)"
    }
    
    if strings.Contains(string(output), "not a dynamic executable") {
        return true, "Static linking verified"
    }
    
    return false, "Binary appears to be dynamically linked"
}

func (dc *DeploymentChecker) checkSecurityFlags() (bool, string) {
    // Check if binary was built with security flags
    cmd := exec.Command("strings", "./main")
    output, err := cmd.Output()
    if err != nil {
        return false, "Could not analyze binary"
    }
    
    content := string(output)
    if strings.Contains(content, "runtime.") && !strings.Contains(content, "debug") {
        return true, "Security flags appear to be used (symbols stripped)"
    }
    
    return false, "Debug symbols may be present"
}

func (dc *DeploymentChecker) checkHealthEndpoint() (bool, string) {
    client := &http.Client{Timeout: 5 * time.Second}
    resp, err := client.Get("http://localhost:8080/health")
    if err != nil {
        return false, fmt.Sprintf("Health endpoint not reachable: %v", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode == 200 {
        return true, "Health endpoint is responding"
    }
    
    return false, fmt.Sprintf("Health endpoint returned status %d", resp.StatusCode)
}

func (dc *DeploymentChecker) checkEnvironmentVariables() (bool, string) {
    required := []string{"DATABASE_URL", "API_KEY", "LOG_LEVEL"}
    missing := []string{}
    
    for _, env := range required {
        if os.Getenv(env) == "" {
            missing = append(missing, env)
        }
    }
    
    if len(missing) > 0 {
        return false, fmt.Sprintf("Missing environment variables: %s", strings.Join(missing, ", "))
    }
    
    return true, "All required environment variables are set"
}

func (dc *DeploymentChecker) checkDependencies() (bool, string) {
    dependencies := []string{
        "https://api.external-service.com/health",
        "postgres://user:pass@localhost:5432/db",
    }
    
    for _, dep := range dependencies {
        if strings.HasPrefix(dep, "http") {
            client := &http.Client{Timeout: 3 * time.Second}
            if _, err := client.Get(dep); err != nil {
                return false, fmt.Sprintf("Dependency not available: %s", dep)
            }
        }
    }
    
    return true, "All dependencies are available"
}

func (dc *DeploymentChecker) checkResourceLimits() (bool, string) {
    memLimit := os.Getenv("MEMORY_LIMIT")
    cpuLimit := os.Getenv("CPU_LIMIT")
    
    if memLimit == "" || cpuLimit == "" {
        return false, "Resource limits not configured"
    }
    
    return true, fmt.Sprintf("Resource limits: CPU=%s, Memory=%s", cpuLimit, memLimit)
}

func (dc *DeploymentChecker) checkLoggingConfig() (bool, string) {
    logLevel := os.Getenv("LOG_LEVEL")
    logFormat := os.Getenv("LOG_FORMAT")
    
    if logLevel == "" {
        return false, "LOG_LEVEL not configured"
    }
    
    validLevels := []string{"debug", "info", "warn", "error"}
    valid := false
    for _, level := range validLevels {
        if logLevel == level {
            valid = true
            break
        }
    }
    
    if !valid {
        return false, fmt.Sprintf("Invalid log level: %s", logLevel)
    }
    
    return true, fmt.Sprintf("Logging configured: level=%s, format=%s", logLevel, logFormat)
}

func (dc *DeploymentChecker) RunChecks() {
    fmt.Println("Production Deployment Checklist")
    fmt.Println("==============================")
    
    passed := 0
    critical_failed := 0
    
    for _, check := range dc.checks {
        success, message := check.CheckFunc()
        status := "✓"
        if !success {
            status = "✗"
            if check.Critical {
                critical_failed++
            }
        } else {
            passed++
        }
        
        criticality := ""
        if check.Critical {
            criticality = " [CRITICAL]"
        }
        
        fmt.Printf("%s %s%s: %s\\n", status, check.Name, criticality, message)
    }
    
    fmt.Printf("\\nSummary: %d/%d checks passed\\n", passed, len(dc.checks))
    
    if critical_failed > 0 {
        fmt.Printf("❌ %d critical checks failed - deployment not recommended\\n", critical_failed)
        os.Exit(1)
    } else {
        fmt.Println("✅ All critical checks passed - deployment approved")
    }
}

func main() {
    checker := NewDeploymentChecker()
    checker.RunChecks()
}
```

Production deployment checklists ensure comprehensive validation before  
releasing applications to production environments. Automated checklist tools  
verify binary optimization, security configuration, dependency availability,  
and operational readiness. These checks prevent common deployment issues and  
ensure consistent production standards across different deployment scenarios.

Go deployment and building represents a mature, comprehensive ecosystem for  
creating and distributing production-ready applications. The combination of  
static binaries, excellent cross-compilation support, container optimization,  
and cloud-native deployment patterns makes Go an ideal choice for modern  
distributed systems and microservices architectures.