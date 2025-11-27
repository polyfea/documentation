# Tutorials

Welcome to the Polyfea tutorials! These hands-on guides walk you through the complete journey of building, containerizing, and deploying microfrontends to Kubernetes using Polyfea's custom resources.

## What You'll Learn

Each tutorial builds on the previous one to create a complete end-to-end workflow:

1. **[Building a Web Component](webcomponents.md)** - Create a simple, reusable web component that encapsulates functionality and styles. This is the foundation of any Polyfea microfrontend.

2. **[Containerizing Your Microfrontend](docker-image.md)** - Package your web component in a Docker image and push it to a container registry, making it ready for Kubernetes deployment.

3. **[Deploying with Polyfea CRDs](k8s-manifests.md)** - Write Kubernetes manifests including standard resources (Deployment, Service) and Polyfea Custom Resources (MicroFrontendClass, MicroFrontend, WebComponent), then deploy everything to your local cluster.

By the end, you'll have a complete microfrontend running in Kubernetes, managed by Polyfea's controller, and accessible through the Polyfea shell application.

## Prerequisites

Before starting these tutorials, make sure you have:

- Basic understanding of [web components](../core-concepts/webcomponents.md)
- Familiarity with [Polyfea's architecture](../core-concepts/introduction.md)
- Completed the [Getting Started](../getting-started.md) guide
- A local Kubernetes cluster (Kind, Minikube, or Docker Desktop)
- Docker installed for building images
- kubectl and Helm configured
