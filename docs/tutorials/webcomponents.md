# Building a Web Component for Polyfea

In this tutorial, you'll create a reusable web component using the [Lit](https://lit.dev/) framework. This component will serve as a microfrontend container with two `polyfea-context` elements, demonstrating how to build composable interfaces for the Polyfea platform.

## What You'll Build

- A grid-based container component with two Polyfea context areas
- A reusable tile component that can be dynamically loaded into context areas
- TypeScript-based web components with proper type declarations

## Prerequisites

- Node.js (v16+) and npm installed
- Basic understanding of TypeScript and web components
- Familiarity with [Lit framework](https://lit.dev/docs/) (recommended)

## Step 1: Set Up the Lit Starter Kit

Clone the Lit TypeScript starter:

```bash
git clone https://github.com/lit/lit-element-starter-ts.git my-polyfea-component
cd my-polyfea-component
```

Install dependencies:

```bash
npm install
```

## Step 2: Build and Test the Starter

Verify the setup works:

```bash
npm run build
npm run start
```

You should see the **Component Demo** page in your browser. The default component demonstrates Lit's basic functionality.

## Step 3: Create the Container Component

We'll modify `my-element.ts` to create a grid layout with two `polyfea-context` elements.

Update `src/my-element.ts`:

```typescript
/**
 * @license
 * Copyright 2019 Google LLC
 * SPDX-License-Identifier: BSD-3-Clause
 */

import {LitElement, html, css} from 'lit';
import {customElement} from 'lit/decorators.js';

/**
 * A container element with two polyfea-context areas arranged in a grid.
 *
 * @slot - This element has a slot
 */
@customElement('my-element')
export class MyElement extends LitElement {
  static override styles = css`
    .grid-container {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 16px;
    }

    .grid-container > div {
      width: 300px;
      height: 200px;
      border: 1px solid #ccc;
      padding: 8px;
    }
  `;

  override render() {
    return html`
      <div class="grid-container">
        <div>
          <polyfea-context name="left"></polyfea-context>
        </div>
        <div>
          <polyfea-context name="right"></polyfea-context>
        </div>
      </div>
    `;
  }
}

declare global {
  interface HTMLElementTagNameMap {
    'my-element': MyElement;
  }
}
```

### What's Happening Here?

- **Grid Layout**: Creates a 2-column responsive grid
- **polyfea-context Elements**: These are Polyfea's dynamic loading zones where microfrontends will be injected at runtime
- **Named Contexts**: The `name` attribute (`left` and `right`) identifies each context area for targeted component loading

## Step 4: Build a Tile Component

Create a reusable tile component that can be dynamically loaded into the context areas.

Create `src/my-tile-element.ts`:

```typescript
/**
 * @license
 * Copyright 2019 Google LLC
 * SPDX-License-Identifier: BSD-3-Clause
 */

import {LitElement, html, css} from 'lit';
import {customElement, property} from 'lit/decorators.js';

/**
 * A simple tile/card component with centered bold text.
 *
 * @property {string} text - The text to display in the tile
 */
@customElement('my-tile')
export class MyTile extends LitElement {
  @property({type: String})
  text = '';

  static override styles = css`
    :host {
      display: block;
    }

    .tile {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 200px;
      height: 150px;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      border-radius: 12px;
      box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
      padding: 20px;
      transition: transform 0.2s, box-shadow 0.2s;
    }

    .tile:hover {
      transform: translateY(-4px);
      box-shadow: 0 8px 12px rgba(0, 0, 0, 0.15);
    }

    .text {
      font-weight: bold;
      font-size: 1.5rem;
      color: white;
      text-align: center;
      word-break: break-word;
    }
  `;

  override render() {
    return html`
      <div class="tile">
        <div class="text">${this.text}</div>
      </div>
    `;
  }
}

declare global {
  interface HTMLElementTagNameMap {
    'my-tile': MyTile;
  }
}
```

### Tile Component Features

- **Reactive Property**: The `text` property updates the display when changed
- **Shadow DOM Styling**: Styles are encapsulated and won't conflict with other components
- **Hover Effects**: Smooth animations enhance user interaction
- **Gradient Background**: Modern visual design with purple gradient

## Step 5: Test Locally (Optional)

To see your components in action during development, update your `index.html` to import the tile component:

```html
<script type="module" src="./src/my-element.js"></script>
<script type="module" src="./src/my-tile-element.js"></script>

<my-element></my-element>
```

Start the dev server:

```bash
npm run start
```

**Note**: For this tutorial, we're focusing on building the components. Local testing with polyfea-context requires the Polyfea runtime, which we'll set up in the Kubernetes deployment tutorial.

## Step 6: Build the Project

Build the components for production:

We will do a little bit of hacking here to set up rollup for bundling our components.
We want to simplify it for the purpose of this tutorial.

So first create a file in the root of your project named `index.ts` with the following content:

```typescript
// Entry point that exports all web components
export { MyElement } from './my-element.js';
export { MyTile } from './my-tile-element.js';
```

Install rollup if not already installed:

```bash
npm install --save-dev rollup \
  @web/rollup-plugin-html \
  @web/rollup-plugin-copy \
  @rollup/plugin-node-resolve \
  @rollup/plugin-terser \
  rollup-plugin-minify-html-literals \
  rollup-plugin-summary
```

We need to add a rollup configuration to bundle our components properly. Create or update `rollup.config.js` with the following content:

```javascript
import resolve from '@rollup/plugin-node-resolve';
import { eventOptions } from 'lit/decorators.js';
import summary from 'rollup-plugin-summary';

export default {
  input: 'index.js',
  output: {
    file: 'dist/webcomponents.bundled.js',
    format: 'esm',
  },
  plugins: [
    // Resolve bare module specifiers to relative paths and bundle dependencies
    resolve(),
    // Print bundle summary
    summary(),
  ],
};
```

This configuration simply bundles our components and their dependencies into a single file.

Run the build:

```bash
npx rollup -c
```

This will generate a bundled file at `dist/webcomponents.bundled.js` containing both `my-element` and `my-tile` components along with their dependencies.

Building like this is not optimal for production use, but it works for our tutorial purposes. Currently polyfea does not support import maps so we need to bundle everything into a single file. This tutorial will be updated in the future when polyfea supports import maps.

## What's Next?

You've successfully created two web components:

- `my-element`: A container with Polyfea context areas
- `my-tile`: A reusable tile component with properties

In the [next tutorial](docker-image.md), you'll learn how to:

- Containerize these components in a Docker image
- Serve them with nginx
- Push the image to a container registry

## Summary

This tutorial covered:

- Setting up a Lit TypeScript project
- Creating a grid-based container with `polyfea-context` elements
- Building a reusable tile component with reactive properties
- Proper TypeScript type declarations for custom elements

Your components are now ready to be packaged and deployed as Polyfea microfrontends!