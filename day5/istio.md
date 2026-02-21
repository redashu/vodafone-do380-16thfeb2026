# OpenShift Service Mesh - Complete Implementation Guide

Step-by-step guide to deploying and configuring Istio-based service mesh in OpenShift.

---

## 🎯 What We Will Do (Big Picture)

We will:

1. **Install Service Mesh**
2. **Create mesh control plane**
3. **Enable namespace injection**
4. **Deploy sample webapp**
5. **Expose it via Gateway**
6. **Apply traffic rules**
7. **Enable mTLS**
8. **Do traffic shifting (canary)**
9. **Apply security policies**

Step-by-step concept + demo style.

---

## 🏗️ Service Mesh Architecture

```
┌─────────────────────────────────────────────────────────┐
│              OpenShift Cluster                           │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  ┌───────────────────────────────────────────────────┐  │
│  │         istio-system namespace                    │  │
│  │                                                    │  │
│  │  ┌──────────────────────────────────────────┐   │  │
│  │  │  ServiceMeshControlPlane (SMCP)          │   │  │
│  │  │                                           │   │  │
│  │  │  • Istiod (control plane)                │   │  │
│  │  │  • Ingress Gateway                       │   │  │
│  │  │  • Egress Gateway                        │   │  │
│  │  │  • Kiali (observability)                 │   │  │
│  │  │  • Jaeger (tracing)                      │   │  │
│  │  │  • Prometheus (metrics)                  │   │  │
│  │  └──────────────────────────────────────────┘   │  │
│  │                                                    │  │
│  │  ServiceMeshMemberRoll                            │  │
│  │  (defines member namespaces)                      │  │
│  └───────────────────────────────────────────────────┘  │
│                          ↓                               │
│  ┌───────────────────────────────────────────────────┐  │
│  │         webapp namespace (mesh member)            │  │
│  │                                                    │  │
│  │  ┌─────────────────────────────────────────┐     │  │
│  │  │  Pod: web-v1                            │     │  │
│  │  │  ┌──────────┐  ┌──────────────────┐   │     │  │
│  │  │  │  nginx   │  │  envoy-sidecar   │   │     │  │
│  │  │  │  (app)   │  │  (proxy)         │   │     │  │
│  │  │  └──────────┘  └──────────────────┘   │     │  │
│  │  └─────────────────────────────────────────┘     │  │
│  │                                                    │  │
│  │  Gateway, VirtualService, DestinationRule         │  │
│  └───────────────────────────────────────────────────┘  │
│                                                           │
└─────────────────────────────────────────────────────────┘
```

---

## 🔷 Step 1 – Install Service Mesh Operator

From OperatorHub install:

1. **Red Hat OpenShift Service Mesh**
2. **Kiali Operator**
3. **Red Hat OpenShift distributed tracing platform** (Jaeger)
4. **OpenShift Elasticsearch Operator** (optional, for tracing storage)

### Installation Order

Install in this sequence:

```bash
# 1. Elasticsearch Operator (optional)
# Install in: openshift-operators-redhat namespace

# 2. Jaeger Operator
# Install in: openshift-distributed-tracing namespace

# 3. Kiali Operator
# Install in: openshift-operators namespace

# 4. Service Mesh Operator
# Install in: openshift-operators namespace
```

### Verify Installation

```bash
oc get csv -n openshift-operators | grep -E "servicemesh|kiali|jaeger"
```

---

## 🔷 Step 2 – Create ServiceMeshControlPlane

After installation, create:

**ServiceMeshControlPlane (SMCP)**

Create namespace:

```bash
oc new-project istio-system
```

Create SMCP:

```yaml
apiVersion: maistra.io/v2
kind: ServiceMeshControlPlane
metadata:
  name: basic
  namespace: istio-system
spec:
  version: v2.5
  tracing:
    type: Jaeger
    sampling: 10000
  addons:
    jaeger:
      install:
        storage:
          type: Memory
    kiali:
      enabled: true
    prometheus:
      enabled: true
  security:
    dataPlane:
      mtls: true
```

Apply:

```bash
oc apply -f smcp.yaml
```

This creates:

- ✅ **Istiod** (control plane)
- ✅ **Ingress gateway**
- ✅ **Egress gateway**
- ✅ **Kiali**
- ✅ **Jaeger**
- ✅ **Prometheus**

### Verify Control Plane

```bash
# Check SMCP status
oc get smcp -n istio-system

# Should show: ComponentsReady
oc get smcp basic -n istio-system -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}'

# Check all pods
oc get pods -n istio-system
```

---

## 🔷 Step 3 – Create ServiceMeshMemberRoll

This tells mesh:

> Which namespaces are part of mesh

```yaml
apiVersion: maistra.io/v1
kind: ServiceMeshMemberRoll
metadata:
  name: default
  namespace: istio-system
spec:
  members:
    - webapp
```

Apply:

```bash
oc apply -f smmr.yaml
```

**Now namespace `webapp` joins mesh.**

### Verify Membership

```bash
oc get smmr -n istio-system
oc describe smmr default -n istio-system
```

---

## 🔷 Step 4 – Enable Sidecar Injection

Label namespace:

```bash
oc label namespace webapp istio-injection=enabled
```

**Alternative:** Use annotation on specific deployments:

```yaml
metadata:
  annotations:
    sidecar.istio.io/inject: "true"
```

**Now when pods deploy:**

👉 **Envoy sidecar is automatically injected.**

---

## 🔷 Step 5 – Deploy Sample WebApp

### Create namespace:

```bash
oc new-project webapp
```

### Deploy sample nginx app:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-v1
  namespace: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
      version: v1
  template:
    metadata:
      labels:
        app: web
        version: v1
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

### Create Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
  namespace: webapp
spec:
  ports:
  - port: 80
    name: http
  selector:
    app: web
```

Apply:

```bash
oc apply -f deployment.yaml
oc apply -f service.yaml
```

### Verify Sidecar Injection

After deployment:

```bash
oc get pods -n webapp
```

You will see:

```
NAME                      READY   STATUS    RESTARTS   AGE
web-v1-xxxxx              2/2     Running   0          1m
```

**2 containers = app + envoy sidecar**

**Mesh is active.**

Verify sidecar:

```bash
oc get pod <pod-name> -n webapp -o jsonpath='{.spec.containers[*].name}'
```

Output should show: `nginx istio-proxy`

---

## 🔷 Step 6 – Expose App Using Istio Gateway

### Create Gateway:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: web-gateway
  namespace: webapp
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
```

### Create VirtualService:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: web
  namespace: webapp
spec:
  hosts:
  - "*"
  gateways:
  - web-gateway
  http:
  - route:
    - destination:
        host: web
        port:
          number: 80
```

Apply:

```bash
oc apply -f gateway.yaml
oc apply -f virtualservice.yaml
```

### Get Gateway URL:

```bash
oc get route -n istio-system | grep istio-ingressgateway
```

**Now traffic flows through mesh ingress gateway.**

### Test Access:

```bash
GATEWAY_URL=$(oc get route -n istio-system istio-ingressgateway -o jsonpath='{.spec.host}')
curl http://$GATEWAY_URL
```

---

## 🔷 Step 7 – Enable mTLS

By default OpenShift Service Mesh supports mTLS.

To enforce **strict mTLS**:

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: webapp
spec:
  mtls:
    mode: STRICT
```

Apply:

```bash
oc apply -f mtls.yaml
```

**Now:**

✅ All traffic between pods must be encrypted.

### Verify mTLS:

Check in Kiali:

```bash
oc get route kiali -n istio-system
```

In Kiali UI: **Graph → Display → Security**

You'll see padlock icons 🔒 on connections.

---

## 🔷 Step 8 – Canary Deployment (Traffic Shifting)

### Deploy v2:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-v2
  namespace: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
      version: v2
  template:
    metadata:
      labels:
        app: web
        version: v2
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
```

### Create DestinationRule:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: web
  namespace: webapp
spec:
  host: web
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

### Modify VirtualService for Traffic Split:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: web
  namespace: webapp
spec:
  hosts:
  - "*"
  gateways:
  - web-gateway
  http:
  - route:
    - destination:
        host: web
        subset: v1
      weight: 80
    - destination:
        host: web
        subset: v2
      weight: 20
```

Apply:

```bash
oc apply -f deployment-v2.yaml
oc apply -f destinationrule.yaml
oc apply -f virtualservice-canary.yaml
```

**Now:**

- 80% traffic → v1
- 20% traffic → v2

**That is canary rollout.**

### Test Traffic Distribution:

```bash
for i in {1..10}; do curl -s http://$GATEWAY_URL | grep -o "nginx.*"; done
```

---

## 🔷 Step 9 – Fault Injection (Testing)

Add delay to test resilience:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: web
  namespace: webapp
spec:
  hosts:
  - "*"
  gateways:
  - web-gateway
  http:
  - fault:
      delay:
        percentage:
          value: 100
        fixedDelay: 5s
    route:
    - destination:
        host: web
        subset: v1
```

**Now requests delay 5 seconds.**

Used for resilience testing.

### Inject Abort (HTTP 500):

```yaml
fault:
  abort:
    percentage:
      value: 50
    httpStatus: 500
```

50% of requests will fail with 500 error.

---

## 🔷 Step 10 – Observability

Service Mesh gives:

- **Kiali dashboard** (topology, health)
- **Jaeger tracing** (request tracing)
- **Prometheus metrics** (performance)

### Access Kiali:

```bash
oc get route kiali -n istio-system
```

You can see:

- ✅ Traffic graph
- ✅ Latency
- ✅ mTLS status
- ✅ Error rates

### Access Jaeger:

```bash
oc get route jaeger -n istio-system
```

Trace individual requests across services.

### Prometheus Metrics:

```bash
oc get route prometheus -n istio-system
```

Query metrics like:

```promql
istio_requests_total
istio_request_duration_milliseconds
```

---

## 🔧 Advanced Configurations

### Request Timeouts

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: web
spec:
  http:
  - timeout: 10s
    route:
    - destination:
        host: web
```

### Retries

```yaml
http:
- retries:
    attempts: 3
    perTryTimeout: 2s
  route:
  - destination:
      host: web
```

### Circuit Breaking

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: web
spec:
  host: web
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        maxRequestsPerConnection: 2
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
```

---

## 🔥 What DO380 May Test

They may ask you to:

- ✅ Enable sidecar injection
- ✅ Create Gateway + VirtualService
- ✅ Enable mTLS
- ✅ Perform traffic split
- ✅ Verify mesh status

**Not deep production tuning.**

---

## 🎯 Final Simple Flow

```
Install Operator
   ↓
Create ControlPlane (SMCP)
   ↓
Add Namespace to Mesh (SMMR)
   ↓
Deploy App (sidecar injected)
   ↓
Create Gateway
   ↓
Create VirtualService
   ↓
Apply traffic policy (mTLS / Canary / Fault)
```

---

## 🔧 Troubleshooting

### Sidecar Not Injected

```bash
# Check namespace label
oc get namespace webapp --show-labels

# Check SMMR membership
oc get smmr default -n istio-system

# Manually inject sidecar (for testing)
istioctl kube-inject -f deployment.yaml | oc apply -f -
```

### Gateway Not Working

```bash
# Check gateway
oc get gateway -n webapp

# Check virtualservice
oc get virtualservice -n webapp

# Check ingress gateway logs
oc logs -n istio-system -l app=istio-ingressgateway
```

### mTLS Issues

```bash
# Check peer authentication
oc get peerauthentication -n webapp

# Verify in Kiali
# Graph → Display → Security (should show locks)
```

### Check Istio Config

```bash
# Validate configuration
istioctl analyze -n webapp

# Check proxy status
istioctl proxy-status

# Check proxy config
istioctl proxy-config routes <pod-name> -n webapp
```

---

## 📝 Summary Table

| Component | Purpose | Resource Type |
|-----------|---------|---------------|
| **ServiceMeshControlPlane** | Creates control plane | SMCP |
| **ServiceMeshMemberRoll** | Defines member namespaces | SMMR |
| **Gateway** | Exposes services externally | Gateway |
| **VirtualService** | Routes traffic | VirtualService |
| **DestinationRule** | Defines subsets, policies | DestinationRule |
| **PeerAuthentication** | Enforces mTLS | PeerAuthentication |
| **Envoy Sidecar** | Service proxy | Auto-injected |

---

## 🧠 One-Line Summary

**Service Mesh adds:**

- ✅ Traffic control
- ✅ Security (mTLS)
- ✅ Observability
- ✅ Canary deployments
- ✅ Fault injection

**Without modifying application code.**

---

## 📚 Quick Reference Commands

```bash
# Install operators (via OperatorHub UI)

# Create control plane
oc apply -f smcp.yaml

# Create member roll
oc apply -f smmr.yaml

# Enable injection
oc label namespace webapp istio-injection=enabled

# Deploy app
oc apply -f deployment.yaml
oc apply -f service.yaml

# Create gateway and routing
oc apply -f gateway.yaml
oc apply -f virtualservice.yaml

# Enable mTLS
oc apply -f peerauthentication.yaml

# Traffic splitting
oc apply -f destinationrule.yaml
oc apply -f virtualservice-canary.yaml

# Access observability
oc get route kiali -n istio-system
oc get route jaeger -n istio-system

# Troubleshooting
istioctl analyze
istioctl proxy-status
oc logs -n istio-system -l app=istiod
```

---

## 🏁 Key Takeaways

1. **Service Mesh provides traffic management, security, and observability** without code changes
2. **SMCP creates the control plane** in istio-system namespace
3. **SMMR defines which namespaces join the mesh**
4. **Sidecar injection** happens automatically when namespace is labeled
5. **Gateway + VirtualService** expose services externally
6. **DestinationRule** defines traffic subsets and policies
7. **mTLS encrypts** all inter-service communication
8. **Canary deployments** are achieved through weight-based routing
9. **Kiali provides visibility** into service mesh topology
10. **DO380 focuses on basic operations**, not deep tuning