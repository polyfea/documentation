# Polyfea Projects

Polyfea consists of multiple repositories working together to provide a complete microfrontend solution.

## Core Repositories

### [Polyfea Controller](https://github.com/polyfea/polyfea-controller)
Kubernetes operator that manages microfrontend deployments as native Kubernetes resources.

**Features:**
- Custom Resource Definitions (CRDs)
- Namespace policies for multi-tenancy
- Service URL resolution
- Progressive Web App support

### [Polyfea Browser Core](https://github.com/polyfea/core)
Browser-side library that loads and manages microfrontends dynamically.

**Features:**
- Dynamic microfrontend loading
- Dependency management
- Context area system
- Navigation API integration

### [NPM Package: @polyfea/core](https://www.npmjs.com/package/@polyfea/core)
The core library published on NPM for easy integration into your projects.

```bash
npm install @polyfea/core
```

## Additional Resources

- [Helm Charts](https://github.com/polyfea/charts)
- [Browser API](https://github.com/polyfea/browser-api)
