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
- **[Core Concepts](core-concepts/introduction.md)** - In-depth concepts and architecture

## Use Cases

Polyfea is ideal for:

- Large-scale applications with multiple teams
- Migrating from monolithic frontends incrementally
- Building multi-tenant SaaS platforms
- Organizations using Kubernetes for orchestration
- Teams needing strong isolation and independent releases