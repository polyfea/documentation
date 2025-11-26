# Polyfea Core: Browser-Side Magic

## The polyfea-context Element

The heart of Polyfea's browser integration is the `<polyfea-context>` element. It's a custom element that:

1. **Fetches metadata** from the controller
2. **Loads microfrontends** dynamically
3. **Fills itself** with the actual components (using `display: contents` to remain transparent)
4. **Manages dependencies** automatically

## How It Works

### Step 1: Placeholder in HTML

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

### Step 2: Query the Backend

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

### Step 3: Load Dependencies

Polyfea recursively resolves the dependency graph using `loadMicrofrontendRecursive()`:

```
shared-components (load first)
    ├── user-module (depends on shared-components)
    └── settings-module (independent)
```

### Step 4: Insert Elements

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
