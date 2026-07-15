# Kubernetes Custom Resources: Declarative Configuration

## What Are Custom Resources?

See also: [Kubernetes Custom Resources - Official Docs](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)

Kubernetes Custom Resource Definitions (CRDs) extend Kubernetes to manage application-specific resources. They let you treat your microfrontends as first-class Kubernetes objects.

**Standard Kubernetes Resources:**
```yaml
kind: Deployment
kind: Service
kind: Ingress
```

**Polyfea Custom Resources:**
```yaml
kind: MicroFrontendClass
kind: MicroFrontend
kind: WebComponent
```

## The Three Polyfea Resources

### 1. MicroFrontendClass

Defines shared configuration for a group of microfrontends - like a "tenant" or "application":

```yaml
apiVersion: polyfea.github.io/v1alpha1
kind: MicroFrontendClass
metadata:
  name: my-app
spec:
  baseUri: "/app"
  title: "My Application"
  cspHeader: "default-src 'self'; script-src 'self' 'unsafe-inline'"
  namespacePolicy:
    from: FromNamespaces
    namespaces:
      - team-a
      - team-b
```

**Purpose:** 

- Define routing and base URL
- Set security policies (CSP headers)
- Configure PWA capabilities
- Control which namespaces can attach microfrontends

### 2. MicroFrontend

Describes an individual microfrontend - its location, dependencies, and loading strategy:

```yaml
apiVersion: polyfea.github.io/v1alpha1
kind: MicroFrontend
metadata:
  name: user-module
  namespace: team-a
spec:
  frontendClass: my-app
  modulePath: "/dist/user.esm.js"
  service:
    name: user-service
    port: 80
  dependsOn:
    - shared-components
  staticPaths:
    - kind: stylesheet
      path: "/dist/styles.css"
```

**Purpose:**

- Point to the JavaScript module and assets
- Declare dependencies on other microfrontends
- Configure caching strategy
- Link to the hosting service

### 3. WebComponent

Represents individual web components and where they should appear:

```yaml
apiVersion: polyfea.github.io/v1alpha1
kind: WebComponent
metadata:
  name: user-nav-item
  namespace: team-a
spec:
  element: user-nav-item
  microFrontend:
    name: user-module
  displayRules:
    - allOf:
      - context-name: navigation
  attributes:
    - name: label
      value: "Profile"
    - name: icon
      value: "person"
```

**Purpose:**

- Map component tags to microfrontends
- Define where components appear (context areas)
- Set conditional rendering rules
- Configure component attributes

## How Kubernetes Controllers Work

Controllers continuously reconcile desired state (your YAML) with actual state (running system):

The Polyfea controller watches these resources and:

1. Validates configurations
2. Resolves service URLs
3. Checks namespace policies
4. Builds metadata for the browser
5. Updates status conditions

## Controller Resolution: Building the Metadata

### The Reconciliation Loop

When you create or update resources, the controller:

1. Validates the Resource
2. Resolves Service URLs
3. Builds Context Area Metadata
4. Updates Status Conditions

### Namespace Policy Enforcement

The controller enforces multi-tenancy through namespace policies:

```yaml
# Cluster-scoped MicroFrontendClass
namespacePolicy:
  from: FromNamespaces
  namespaces:
    - team-a
    - team-b
```

**When team-c tries to attach a MicroFrontend:**

```yaml
# This will be rejected
apiVersion: polyfea.github.io/v1alpha1
kind: MicroFrontend
metadata:
  name: unauthorized-mfe
  namespace: team-c  # ❌ Not in allowed list
spec:
  frontendClass: my-app
```

**Controller action:**
```yaml
status:
  phase: Rejected
  rejectionReason: "Namespace not allowed by MicroFrontendClass namespace policy"
  conditions:
    - type: Accepted
      status: "False"
      reason: NamespaceNotAllowed
```
