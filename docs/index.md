# Welcome to Polyfea Documentation

Polyfea is a powerful microfrontend framework that enables clean, decoupled development of web applications. It consists of a core browser library and a Kubernetes controller that work together to manage the lifecycle of microfrontends.

<!-- Hot reload test - if you see this, it's working! -->

## What is Polyfea?

Polyfea provides a complete platform for building microfrontend architectures:

- **[@polyfea/core](https://www.npmjs.com/package/@polyfea/core)** - The browser-side microfrontend framework and driver
- **polyfea-controller** - A Kubernetes operator for managing microfrontend deployments

## Key Components

### Core Library (@polyfea/core)

The backbone of the Polyfea framework that manages microfrontend lifecycle in the browser:

- **polyfea-context element** - Loads microfrontends dynamically, replacing itself with the microfrontend's content
- **Polyfea class** - Advanced API for programmatic control over microfrontend loading
- **Navigation polyfill** - Intercepts navigation events for browsers without native Navigation API support
- **href function** - Simplifies navigation in single-page applications

### Kubernetes Controller

A Kubernetes operator built with the Operator SDK that enables cloud-native microfrontend management:

- **MicroFrontendClass** - Defines shared configuration (routing, CSP headers, PWA settings)
- **MicroFrontend** - Describes individual microfrontends and their deployment configuration
- **WebComponent** - Represents the web components that compose a microfrontend

## Key Features

### Browser Framework
- **Dynamic Loading** - Load microfrontends on-demand based on context
- **Dependency Management** - Automatically handle microfrontend dependencies
- **Custom Elements** - First-class support for web components
- **Single Page Navigation** - Seamless navigation without page reloads

### Kubernetes Controller
- **Namespace Policies** - Multi-tenancy and security isolation
- **CSP Management** - Content Security Policy configuration per frontend class
- **Progressive Web Apps** - Optional PWA capabilities with service worker support
- **Status Reporting** - Comprehensive Kubernetes-style status conditions
- **Automatic Reconciliation** - Self-healing resource management

## Architecture Benefits

- **Strong Separation of Concerns** - Independent development and deployment
- **Team Autonomy** - Different teams can own different microfrontends
- **Technology Freedom** - Mix different frameworks and technologies
- **Scalability** - Scale individual microfrontends independently
- **Kubernetes-Native** - Built for cloud-native environments

## Documentation Structure

- **[Getting Started](getting-started.md)** - Installation and basic setup
- **[Polyfea 101](polyfea-101.md)** - In-depth concepts and architecture

## Use Cases

Polyfea is ideal for:

- Large-scale applications with multiple teams
- Migrating from monolithic frontends incrementally
- Building multi-tenant SaaS platforms
- Organizations using Kubernetes for orchestration
- Teams needing strong isolation and independent releases

!!! tip "Getting Started"
    New to Polyfea? Start with the [Getting Started](getting-started.md) guide to set up your first microfrontend application.

!!! info "Kubernetes Users"
    If you're deploying to Kubernetes, check out the controller documentation for managing microfrontends as native Kubernetes resources.

## Quick Start

### Browser-Side Development

The Polyfea core library enables dynamic loading of microfrontends through the `<polyfea-context>` element. This element acts as a placeholder that gets replaced with actual microfrontend content based on your configuration.

#### Installation

Install the core library:

```bash
npm install @polyfea/core
```

#### Basic Setup

Set up your HTML with the Polyfea boot script:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <base href="/ui/">
  <title>My Polyfea Application</title>
  <meta name="polyfea.backend" content="static://"> 
  <script type="module" src="node_modules/@polyfea/core/dist/boot.mjs"></script>
</head>
<body>
  <!-- The polyfea-context element will be automatically inserted here -->
</body>
</html>
```

Or load from CDN:

```html
<script type="module" src="https://cdn.jsdelivr.net/npm/@polyfea/core@1/dist/boot.mjs"></script>
```

#### Using Context Areas

Load microfrontends dynamically using context areas:

```html
<body>
  <!-- Load the application shell (takes first matching element) -->
  <polyfea-context name="shell" take="1"></polyfea-context>
  
  <!-- Load navigation items (takes up to 5 elements) -->
  <nav>
    <polyfea-context name="navigation" take="5"></polyfea-context>
  </nav>
  
  <!-- Load main content area -->
  <main>
    <polyfea-context name="main-content"></polyfea-context>
  </main>
</body>
```

#### Using the Material Design Shell

For a complete Material Design 3 application shell with built-in navigation:

```html
<polyfea-md-shell 
  application-headline="My App"
  topbar-variant="centered">
  
  <!-- Navigation drawer items -->
  <div slot="drawer">
    <polyfea-context name="drawer-items"></polyfea-context>
  </div>
  
  <!-- Navigation rail items (medium screens) -->
  <div slot="rail">
    <polyfea-context name="rail-items" take="7"></polyfea-context>
  </div>
  
  <!-- Bottom navigation (mobile) -->
  <div slot="navigation">
    <polyfea-context name="nav-items" take="5"></polyfea-context>
  </div>
  
  <!-- Main content -->
  <polyfea-context name="page-content"></polyfea-context>
</polyfea-md-shell>
```

Available slots:
- `topbar-leading` / `topbar-trailing` - Top bar icons
- `topbar-menu` - Top bar menu items
- `drawer` - Navigation drawer content
- `rail` - Navigation rail content (7 items max)
- `navigation` - Bottom navigation bar (5 items max)
- `rail-primary-action` - Floating action button area
- Default slot - Main content area

#### Configuration Example

Create a static configuration file at `/ui/polyfea/static-config`:

```json
{
  "microfrontends": {
    "my-shell": {
      "module": "./dist/shell.esm.js",
      "resources": [
        {
          "kind": "stylesheet",
          "href": "./dist/shell.css"
        }
      ]
    },
    "user-module": {
      "dependsOn": ["my-shell"],
      "module": "./dist/user.esm.js"
    }
  },
  "contextAreas": [
    { 
      "name": "shell",
      "contextArea": {
        "elements": [
          {
            "tagName": "polyfea-md-shell",
            "microfrontend": "my-shell",
            "attributes": {
              "application-headline": "My Application"
            }
          }
        ]
      }
    },
    {
      "name": "drawer-items",
      "contextArea": {
        "elements": [
          {
            "tagName": "user-profile",
            "microfrontend": "user-module"
          }
        ]
      }
    }
  ]
}
```

### Kubernetes Deployment

The easiest way to deploy Polyfea to Kubernetes is using Helm charts:

#### Add the Helm Repository

```bash
helm repo add polyfea https://polyfea.github.io/charts
helm repo update
```

#### Install the Controller

```bash
helm install polyfea-controller polyfea/polyfea-controller \
  --namespace polyfea-system \
  --create-namespace
```

#### Install with Sample Applications

The samples chart includes a dependency on the controller, so it can be installed standalone:

```bash
# Install both controller and samples
helm install polyfea polyfea/polyfea-md-shell-samples \
  --namespace polyfea \
  --create-namespace
```

Customize the samples installation:

```bash
# Disable the Earth sample application
helm install polyfea polyfea/polyfea-md-shell-samples \
  --set samples.earthSample=false \
  --namespace polyfea \
  --create-namespace
```