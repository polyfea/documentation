# Web Components: The Building Blocks

## What Are Web Components?

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

## Why Web Components for Microfrontends?

Web Components are ideal for microfrontend architecture because they:

- **Framework agnostic** - Work with React, Vue, Angular, or vanilla JS
- **Truly encapsulated** - Styles and scripts don't leak or conflict
- **Standard-based** - Native browser support, no framework lock-in
- **Composable** - Elements can contain other elements naturally
- **Progressive** - Load only what you need, when you need it

## Polyfea's Approach

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