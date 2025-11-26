# Polyfea 101: Core Concepts

Polyfea combines modern web standards with cloud-native Kubernetes patterns to create a powerful microfrontend architecture. This guide explains the fundamental concepts that make Polyfea work.

## Introduction

Polyfea bridges two worlds: **browser-based web components** and **Kubernetes-managed deployments**. Instead of hardcoding microfrontends into your application, Polyfea lets you declare them as Kubernetes resources that are dynamically assembled at runtime.

This architecture enables:

- **Independent deployment** - Teams deploy microfrontends without coordinating releases  
- **Runtime composition** - Applications adapt based on available components  
- **Declarative configuration** - Infrastructure-as-code for your frontend  
- **Multi-tenancy** - Safe isolation between teams and applications  

Let's explore the key concepts that power this architecture.

## Web Components: The Building Blocks

### What Are Web Components?

See also: [Web Components - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_components)

Web Components are a set of web platform APIs that allow you to create custom, reusable HTML elements. They consist of three main technologies:

**1. Custom Elements**
Define new HTML tags with custom behavior:

```javascript
class UserProfile extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `<div>Hello, ${this.getAttribute('username')}</div>`;
  }
}
customElements.define('user-profile', UserProfile);
```

Usage:
```html
<user-profile username="john"></user-profile>
```

**2. Shadow DOM**
Encapsulates styles and markup, preventing conflicts:

```javascript
class MyCard extends HTMLElement {
  connectedCallback() {
    const shadow = this.attachShadow({mode: 'open'});
    shadow.innerHTML = `
      <style>
        .card { padding: 20px; border: 1px solid #ccc; }
      </style>
      <div class="card"><slot></slot></div>
    `;
  }
}
```

**3. HTML Templates**
Reusable markup that isn't rendered until cloned:

```html
<template id="my-template">
  <div class="item">
    <slot name="title"></slot>
  </div>
</template>
```

### Why Web Components for Microfrontends?

Web Components are ideal for microfrontend architecture because they:

- **Framework agnostic** - Work with React, Vue, Angular, or vanilla JS
- **Truly encapsulated** - Styles and scripts don't leak or conflict
- **Standard-based** - Native browser support, no framework lock-in
- **Composable** - Elements can contain other elements naturally
- **Progressive** - Load only what you need, when you need it

### Polyfea's Approach

In Polyfea, each microfrontend exposes one or more web components. For example:

```javascript
// Microfrontend A exports:
<navigation-menu></navigation-menu>

// Microfrontend B exports:
<user-dashboard></user-dashboard>

// Microfrontend C exports:
<settings-panel></settings-panel>
```

These components are then composed together to form your complete application.

## Kubernetes Custom Resources: Declarative Configuration

### What Are Custom Resources?

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

### The Three Polyfea Resources

#### 1. MicroFrontendClass

Defines shared configuration for a group of microfrontends - like a "tenant" or "application":

```yaml
apiVersion: polyfea.github.io/v1alpha1
kind: MicroFrontendClass
metadata:
  name: my-app
  namespace: platform
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

#### 2. MicroFrontend

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
      href: "/dist/styles.css"
```

**Purpose:**

- Point to the JavaScript module and assets
- Declare dependencies on other microfrontends
- Configure caching strategy
- Link to the hosting service

#### 3. WebComponent

Represents individual web components and where they should appear:

```yaml
apiVersion: polyfea.github.io/v1alpha1
kind: WebComponent
metadata:
  name: user-nav-item
  namespace: team-a
spec:
  element: user-nav-item
  microFrontend: user-module
  displayRules:
    allOf:
    - context-name: navigation
  attributes:
    label: "Profile"
    icon: "person"
```

**Purpose:**

- Map component tags to microfrontends
- Define where components appear (context areas)
- Set conditional rendering rules
- Configure component attributes

### How Kubernetes Controllers Work

Controllers continuously reconcile desired state (your YAML) with actual state (running system):

The Polyfea controller watches these resources and:

1. Validates configurations
2. Resolves service URLs
3. Checks namespace policies
4. Builds metadata for the browser
5. Updates status conditions

## Polyfea Core: Browser-Side Magic

### The polyfea-context Element

The heart of Polyfea's browser integration is the `<polyfea-context>` element. It's a custom element that:

1. **Fetches metadata** from the controller
2. **Loads microfrontends** dynamically
3. **Fills itself** with the actual components (using `display: contents` to remain transparent)
4. **Manages dependencies** automatically

### How It Works

#### Step 1: Placeholder in HTML

```html
<body>
  <polyfea-context name="shell" take="1"></polyfea-context>
  
  <nav>
    <polyfea-context name="navigation" take="5"></polyfea-context>
  </nav>
  
  <main>
    <polyfea-context name="main-content"></polyfea-context>
  </main>
</body>
```

#### Step 2: Query the Backend

When the page loads, `polyfea-context` queries the controller:

```http
GET /polyfea/context-area?name=navigation&path=./
```

The `path` parameter is the current location path relative to `document.baseURI`. This allows the controller to return different components based on the current route.

Response:
```json
{
  "elements": [
    {
      "tagName": "user-nav-item",
      "microfrontend": "user-module",
      "attributes": { "label": "Profile" }
    },
    {
      "tagName": "settings-nav-item",
      "microfrontend": "settings-module",
      "attributes": { "label": "Settings" }
    }
  ],
  "microfrontends": {
    "user-module": {
      "module": "http://user-service/dist/user.esm.js",
      "dependsOn": ["shared-components"]
    },
    "settings-module": {
      "module": "http://settings-service/dist/settings.esm.js"
    }
  }
}
```

#### Step 3: Load Dependencies

Polyfea recursively resolves the dependency graph using `loadMicrofrontendRecursive()`:

```
shared-components (load first)
    ├── user-module (depends on shared-components)
    └── settings-module (independent)
```

#### Step 4: Insert Elements

The `<polyfea-context>` element fills itself with the actual web components and uses `display: contents` CSS to become transparent in the layout:

```html
<nav>
  <polyfea-context name="navigation" style="display: contents;">
    <user-nav-item label="Profile"></user-nav-item>
    <settings-nav-item label="Settings"></settings-nav-item>
  </polyfea-context>
</nav>
```

The `display: contents` style means the `<polyfea-context>` element doesn't generate a box in the layout - its children render as if they were direct children of the parent `<nav>` element, while the `<polyfea-context>` element remains in the DOM for potential updates.

### Display Rules and Conditional Rendering

WebComponents can specify when they should appear using display rules. Each WebComponent can have multiple display rules, and if **ANY** rule matches, the component is shown.

Within a single display rule, there are three matcher lists that work together with **AND** logic:

```yaml
displayRules:
  # Rule 1: Show in navigation context for users with specific roles
  - allOf:
      - contextName: navigation
    anyOf:
      - role: user
      - role: admin
    noneOf:
      - path: "^/public/.*"
  
  # Rule 2: Show in drawer context on specific paths
  - allOf:
      - contextName: drawer
      - path: "^/dashboard/.*"
```

### How Display Rules Are Evaluated

The controller evaluates display rules using this logic:

**For each display rule (OR between rules):**

1. **ALL of `allOf` matchers must match** (AND logic)
2. **At least ONE of `anyOf` matchers must match** (OR logic) - if `anyOf` is present
3. **NONE of `noneOf` matchers can match** (NOT logic)

If all three conditions are satisfied, that display rule matches and the component is included.

### Matcher Types

Each matcher can check three properties:

**1. Context Name** - Where the `polyfea-context` element is located
```yaml
allOf:
  - contextName: navigation  # Exact match
```

**2. Path** - URL path as a regular expression
```yaml
allOf:
  - path: "^/dashboard/.*"  # Regex pattern
```

**3. Role** - User role from HTTP headers (e.g., `x-auth-request-roles`)
```yaml
anyOf:
  - role: admin
  - role: editor
```

### Examples

**Example 1: Admin-only component in navigation**
```yaml
displayRules:
  - allOf:
      - contextName: navigation
      - role: admin
```
Shows only if context is "navigation" AND user has "admin" role.

**Example 2: Multi-role component with path restriction**
```yaml
displayRules:
  - allOf:
      - contextName: toolbar
    anyOf:
      - role: admin
      - role: editor
    noneOf:
      - path: "^/public/.*"
```
Shows if context is "toolbar" AND (user is admin OR editor) AND path doesn't start with "/public/".

**Example 3: Multiple rules for different contexts**
```yaml
displayRules:
  # Show in desktop navigation for all users
  - allOf:
      - contextName: navigation
  
  # Or show in mobile menu for authenticated users
  - allOf:
      - contextName: mobile-menu
    anyOf:
      - role: user
      - role: admin
```
Shows if EITHER rule matches (context is "navigation" OR context is "mobile-menu" with a user role).

**Example 4: Path-based conditional rendering**
```yaml
displayRules:
  # Show settings only on settings pages
  - allOf:
      - contextName: sidebar
      - path: "^/settings/.*"
```

Shows only when in "sidebar" context AND on a path starting with "/settings/".

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
# MicroFrontendClass in platform namespace
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

## How It All Ties Together

### The Complete Flow

Let's trace a complete request from browser to components:

#### 1. User Opens the Application

```
Browser → https://my-app.example.com/
```

#### 2. HTML with polyfea-context Loads

```html
<!DOCTYPE html>
<html>
<head>
  <script type="module" src="https://cdn.jsdelivr.net/npm/@polyfea/core@1/dist/boot.mjs"></script>
</head>
<body>
  <!-- This will be populated -->
  <polyfea-context name="shell" take="1"></polyfea-context>
</body>
</html>
```

#### 3. Polyfea Core Queries Controller

```
Browser GET → /polyfea/context-area?name=shell&take=1
Controller → Searches WebComponents where context="shell"
```

#### 4. Controller Resolves Resources

```yaml
# Found WebComponent
apiVersion: polyfea.github.io/v1alpha1
kind: WebComponent
metadata:
  name: app-shell
spec:
  element: "polyfea-md-shell"
  microFrontend: shell-module
  displayRules:
    - contexts:
        - name: shell
```

```yaml
# Finds referenced MicroFrontend
apiVersion: polyfea.github.io/v1alpha1
kind: MicroFrontend
metadata:
  name: shell-module
spec:
  frontendClass: my-app
  service:
    name: shell-service
  modulePath: "/dist/shell.esm.js"
  dependsOn:
    - material-components
```

#### 5. Controller Returns Metadata

```json
{
  "elements": [
    {
      "tagName": "polyfea-md-shell",
      "attributes": {
        "application-headline": "My App"
      }
    }
  ],
  "microfrontends": {
    "material-components": {
      "module": "http://material-service/dist/material.esm.js"
    },
    "shell-module": {
      "module": "http://shell-service/dist/shell.esm.js",
      "dependsOn": ["material-components"],
      "resources": [
        {
          "kind": "stylesheet",
          "href": "http://shell-service/dist/shell.css"
        }
      ]
    }
  }
}
```

#### 6. Browser Loads Dependencies

```javascript
// Load in dependency order
await import('http://material-service/dist/material.esm.js');
await import('http://shell-service/dist/shell.esm.js');
```

#### 7. Custom Elements Register

```javascript
// Inside shell.esm.js
customElements.define('polyfea-md-shell', PolyfeaMdShell);
```

#### 8. polyfea-context Fills with Components

```html
<body>
  <!-- Before -->
  <polyfea-context name="shell" take="1"></polyfea-context>
  
  <!-- After (polyfea-context remains but is transparent) -->
  <polyfea-context name="shell" take="1" style="display: contents;">
    <polyfea-md-shell application-headline="My App">
      <!-- Shell renders nested content -->
    </polyfea-md-shell>
  </polyfea-context>
</body>
```

The `<polyfea-context>` element stays in the DOM but with `display: contents`, so it doesn't affect the layout. This allows it to react to navigation events and update its children dynamically.

#### 9. Nested Contexts Load

The shell contains more `polyfea-context` elements:

```html
<polyfea-md-shell>
  <nav slot="drawer">
    <polyfea-context name="drawer-items"></polyfea-context>
  </nav>
</polyfea-md-shell>
```

These recursively load their own components, building the complete application.

### Key Benefits of This Architecture

**1. Decoupled Development**
- Teams define WebComponents in their namespace
- No coordination needed for deployments
- Independent release cycles

**2. Runtime Composition**
- Application structure determined at runtime
- Add/remove features without redeploying shell
- A/B testing and feature flags via display rules

**3. Security & Isolation**
- Namespace policies prevent unauthorized access
- CSP headers protect against injection
- Each microfrontend loads from its own service

**4. Kubernetes-Native**
- GitOps-friendly YAML definitions
- Standard kubectl commands
- Integrates with existing CI/CD

**5. Progressive Enhancement**
- Load only what's needed
- Dependency management automatic
- Failed components don't break the app

## Next Steps

Now that you understand the core concepts:

1. **Try the [Getting Started](getting-started.md) guide** - Deploy your first Polyfea application
2. **Explore display rules** - Learn conditional rendering patterns
3. **Build a microfrontend** - Create your own web components
4. **Configure namespace policies** - Set up multi-tenancy
5. **Enable PWA features** - Add offline support and caching

The combination of web components, Kubernetes resources, and dynamic loading creates a powerful platform for building scalable microfrontend applications.
