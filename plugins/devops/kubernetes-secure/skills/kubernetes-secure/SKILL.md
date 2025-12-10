# Kubernetes Secure Generator

Generate Kubernetes manifests with **security hardening by default**.

## Security Principles

### 1. Pod Security Context - REQUIRED

**ALWAYS include securityContext in every Pod/Deployment:**

```yaml
# ✅ CORRECT - Secure by default
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
  containers:
    - name: app
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
```

### 2. Resource Limits - REQUIRED

**ALWAYS define resource requests and limits:**

```yaml
containers:
  - name: app
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "512Mi"
```

### 3. Network Policies - REQUIRED

**ALWAYS include NetworkPolicy for isolation:**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-network-policy
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              role: database
      ports:
        - protocol: TCP
          port: 5432
```

### 4. Service Types - USE CLUSTERIP

**Default to ClusterIP, use Ingress for external access:**

```yaml
# ✅ CORRECT - Internal service
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080

---
# Use Ingress for external access
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
    - hosts:
        - app.example.com
      secretName: app-tls
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-service
                port:
                  number: 80

# ❌ AVOID - Direct public exposure
spec:
  type: LoadBalancer  # Only when necessary, prefer Ingress
```

### 5. Probes - REQUIRED

**ALWAYS include liveness and readiness probes:**

```yaml
containers:
  - name: app
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      timeoutSeconds: 3
      failureThreshold: 3
```

### 6. Secrets Management

**NEVER hardcode secrets, use Kubernetes Secrets or external managers:**

```yaml
# ✅ CORRECT - Reference from Secret
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-credentials
        key: password

# ✅ BETTER - Use external secrets (e.g., AWS Secrets Manager)
# With External Secrets Operator
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: db-credentials
  data:
    - secretKey: password
      remoteRef:
        key: production/db/password

# ❌ NEVER - Hardcoded
env:
  - name: DB_PASSWORD
    value: "supersecret123"
```

### 7. Image Security

**Use specific tags and verified images:**

```yaml
containers:
  - name: app
    image: myregistry.com/app:v1.2.3@sha256:abc123...
    imagePullPolicy: Always
```

## Complete Secure Deployment Template

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
  labels:
    app: secure-app
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: secure-app
  template:
    metadata:
      labels:
        app: secure-app
        version: v1
    spec:
      # Pod-level security
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
        seccompProfile:
          type: RuntimeDefault

      # Service account
      serviceAccountName: secure-app-sa
      automountServiceAccountToken: false

      containers:
        - name: app
          image: myregistry.com/app:v1.2.3
          imagePullPolicy: Always

          # Container-level security
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL

          # Ports
          ports:
            - containerPort: 8080
              protocol: TCP

          # Environment from secrets
          envFrom:
            - secretRef:
                name: app-secrets
            - configMapRef:
                name: app-config

          # Resource limits
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"

          # Health checks
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3

          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3

          # Volumes for tmp (read-only root)
          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: cache
              mountPath: /app/.cache

      volumes:
        - name: tmp
          emptyDir: {}
        - name: cache
          emptyDir: {}

---
apiVersion: v1
kind: Service
metadata:
  name: secure-app
spec:
  type: ClusterIP
  selector:
    app: secure-app
  ports:
    - port: 80
      targetPort: 8080

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: secure-app-network-policy
spec:
  podSelector:
    matchLabels:
      app: secure-app
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress-nginx
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              name: database
      ports:
        - protocol: TCP
          port: 5432
    # Allow DNS
    - to:
        - namespaceSelector: {}
      ports:
        - protocol: UDP
          port: 53
```

## Security Validation Checklist

Before generating any Kubernetes manifest, verify:

- [ ] `securityContext.runAsNonRoot: true`
- [ ] `securityContext.allowPrivilegeEscalation: false`
- [ ] `securityContext.readOnlyRootFilesystem: true` (with tmpfs for writable dirs)
- [ ] `capabilities.drop: [ALL]`
- [ ] `resources.limits` defined for CPU and memory
- [ ] `livenessProbe` and `readinessProbe` configured
- [ ] NetworkPolicy restricts ingress and egress
- [ ] Service type is `ClusterIP` unless explicitly needed
- [ ] No hardcoded secrets
- [ ] Image tag is specific (not `latest`)
- [ ] `automountServiceAccountToken: false` unless needed

## Pod Security Standards

This skill follows Kubernetes Pod Security Standards (PSS):

### Restricted (Default)

Heavily restricted policies following security best practices:

```yaml
securityContext:
  runAsNonRoot: true
  seccompProfile:
    type: RuntimeDefault
  allowPrivilegeEscalation: false
  capabilities:
    drop: [ALL]
```

### Baseline (When Needed)

Minimally restrictive, preventing known privilege escalations:

```yaml
# Only when application requires specific capabilities
securityContext:
  runAsNonRoot: true
  capabilities:
    drop: [ALL]
    add: [NET_BIND_SERVICE]  # Only if needed
```

## Common Patterns

### Database (PostgreSQL)

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 70  # postgres user
        fsGroup: 70
      containers:
        - name: postgres
          image: postgres:16-alpine
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop: [ALL]
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "1000m"
              memory: "1Gi"
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
          livenessProbe:
            exec:
              command: ["pg_isready", "-U", "postgres"]
            initialDelaySeconds: 30
            periodSeconds: 10
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

### Redis

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 999
        fsGroup: 999
      containers:
        - name: redis
          image: redis:7-alpine
          command: ["redis-server", "--requirepass", "$(REDIS_PASSWORD)"]
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: [ALL]
          ports:
            - containerPort: 6379
          env:
            - name: REDIS_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: redis-secret
                  key: password
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
          volumeMounts:
            - name: data
              mountPath: /data
          livenessProbe:
            exec:
              command: ["redis-cli", "ping"]
            initialDelaySeconds: 10
            periodSeconds: 5
      volumes:
        - name: data
          emptyDir: {}
```

## References

- [Kubernetes Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)
- [NSA Kubernetes Hardening Guide](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)
