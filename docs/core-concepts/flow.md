# The Complete Flow

Let's trace a complete request from browser to components:

## 1. User Opens the Application

```
Browser → https://my-app.example.com/
```

## 2. HTML with polyfea-context Loads

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

## 3. Polyfea Core Queries Controller

```
Browser GET → /polyfea/context-area?name=shell&take=1
Controller → Searches WebComponents where context="shell"
```

## 4. Controller Resolves Resources

```yaml
# Found WebComponent
apiVersion: polyfea.github.io/v1alpha1
kind: WebComponent
metadata:
  name: app-shell
spec:
  element: "polyfea-md-shell"
  microFrontend: 
    name: shell-module
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
  frontendClass: 
    name: my-app
  service:
    name: shell-service
  modulePath: "/dist/shell.esm.js"
  dependsOn:
    - material-components
```

## 5. Controller Returns Metadata

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

## 6. Browser Loads Dependencies

```javascript
// Load in dependency order
await import('http://material-service/dist/material.esm.js');
await import('http://shell-service/dist/shell.esm.js');
```

## 7. Custom Elements Register

```javascript
// Inside shell.esm.js
customElements.define('polyfea-md-shell', PolyfeaMdShell);
```

## 8. polyfea-context Fills with Components

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

## 9. Nested Contexts Load

The shell contains more `polyfea-context` elements:

```html
<polyfea-md-shell>
  <nav slot="drawer">
    <polyfea-context name="drawer-items"></polyfea-context>
  </nav>
</polyfea-md-shell>
```

These recursively load their own components, building the complete application.