# Vendor Splitting Pattern Reference

When a widget extension project contains multiple widgets that share large frameworks (React, Vue.js, MUI, etc.), the **vendor splitting pattern** ensures that shared libraries are loaded only once and shared across all widgets.

## The Problem

Without vendor splitting, each widget bundles its own copy of React + MUI + dependencies:

```
widget1.runtime.js  →  React (150KB) + MUI (200KB) + widget code (5KB)
widget2.runtime.js  →  React (150KB) + MUI (200KB) + widget code (3KB)
widget3.runtime.js  →  React (150KB) + MUI (200KB) + widget code (4KB)
Total: 1065KB loaded, 1050KB duplicated
```

With vendor splitting:

```
vendor-core.chunk.js  →  React + MUI (350KB) — loaded ONCE, shared
widget1.runtime.js    →  widget code (5KB) + vendor loader
widget2.runtime.js    →  widget code (3KB) + vendor loader
widget3.runtime.js    →  widget code (4KB) + vendor loader
Total: 362KB loaded, 0 duplicated
```

## Architecture

The pattern has three components:

### 1. Vendor Module (`src/common/vendor-core.ts`)

A module that imports and re-exports all shared dependencies:

```typescript
/**
 * Vendor Core — shared dependencies for ALL widgets in this extension.
 * Loaded as a single webpack chunk, cached globally.
 */

// React core
import React from "react";
import { createRoot } from "react-dom/client";

// MUI
import { ThemeProvider, createTheme } from "@mui/material/styles";
import Button from "@mui/material/Button";
import TextField from "@mui/material/TextField";
// ... import only what's actually used across widgets

// Local theme
import { theme } from "./theme";

export {
  React, createRoot,
  ThemeProvider, createTheme,
  Button, TextField,
  theme,
};
```

### 2. Vendor Loader (`src/common/vendor-loader.ts`)

A caching layer that ensures the vendor chunk is loaded only once, even when multiple widgets request it simultaneously:

```typescript
export type VendorCoreModule = typeof import("./vendor-core");

let vendorCorePromise: Promise<VendorCoreModule> | null = null;
let vendorCoreModule: VendorCoreModule | null = null;

/**
 * Load vendor-core module. Returns cached instance if already loaded.
 * Multiple concurrent calls share the same Promise.
 */
export async function loadVendorCore(): Promise<VendorCoreModule> {
  if (vendorCoreModule) return vendorCoreModule;
  if (!vendorCorePromise) {
    vendorCorePromise = import(
      /* webpackChunkName: "vendor-core" */ "./vendor-core"
    );
  }
  vendorCoreModule = await vendorCorePromise;
  return vendorCoreModule;
}

/**
 * Get vendor-core synchronously. Only call AFTER loadVendorCore() has completed.
 * Useful in React components rendered after the initial async load.
 */
export function getVendorCoreSync(): VendorCoreModule {
  if (!vendorCoreModule) {
    throw new Error("VendorCore not loaded yet! Call loadVendorCore() first.");
  }
  return vendorCoreModule;
}

export function isVendorCoreLoaded(): boolean {
  return vendorCoreModule !== null;
}
```

### 3. Widget Usage

Each widget loads the vendor asynchronously in `afterRender()`:

```typescript
import { loadVendorCore, type VendorCoreModule } from "../common/vendor-loader";

@TWWidgetDefinition
class MyReactWidget extends TWRuntimeWidget {
  _root: any = null;
  _vendor: VendorCoreModule | null = null;

  async afterRender(): Promise<void> {
    this._vendor = await loadVendorCore();
    this._root = this._vendor.createRoot(this.jqElement[0]);
    this.renderComponent();
  }

  renderComponent(): void {
    if (!this._vendor || !this._root) return;
    const { React, ThemeProvider, Button, theme } = this._vendor;
    this._root.render(
      <ThemeProvider theme={theme}>
        <Button onClick={() => this.jqElement.triggerHandler("Clicked")}>
          {this.Label}
        </Button>
      </ThemeProvider>
    );
  }

  beforeDestroy(): void {
    if (this._root) {
      this._root.unmount();
      this._root = null;
    }
  }
}
```

## Multiple Vendor Chunks

For projects where some widgets need additional heavy libraries (e.g., diagram widgets needing ReactFlow), split vendors into tiers:

```
src/common/
├── vendor-core.ts        ← React + MUI (all widgets)
├── vendor-diagram.ts     ← ReactFlow + Dagre (diagram widgets only)
├── vendor-loader.ts      ← Caching loaders for both
└── theme.ts              ← Shared theme configuration
```

```typescript
// vendor-loader.ts — add a second loader
let vendorDiagramPromise: Promise<VendorDiagramModule> | null = null;
let vendorDiagramModule: VendorDiagramModule | null = null;

export async function loadVendorDiagram(): Promise<VendorDiagramModule> {
  if (vendorDiagramModule) return vendorDiagramModule;
  if (!vendorDiagramPromise) {
    vendorDiagramPromise = import(
      /* webpackChunkName: "vendor-diagram" */ "./vendor-diagram"
    );
  }
  vendorDiagramModule = await vendorDiagramPromise;
  return vendorDiagramModule;
}

// Load both in parallel
export async function loadAllVendors() {
  const [core, diagram] = await Promise.all([
    loadVendorCore(),
    loadVendorDiagram(),
  ]);
  return { core, diagram };
}
```

## Applying to Other Frameworks

This pattern works for any framework, not just React:

**Vue.js example:**
```typescript
// vendor-vue.ts
import { createApp, ref, computed, watch, onMounted } from "vue";
import ElementPlus from "element-plus";
export { createApp, ref, computed, watch, onMounted, ElementPlus };
```

**Any large library:**
```typescript
// vendor-chart.ts
import * as echarts from "echarts";
import "echarts-gl";
export { echarts };
```

## Webpack Configuration for Vendor Splitting

No special webpack configuration is needed. Webpack automatically creates separate chunks for dynamic `import()` calls. The `webpackChunkName` magic comment controls the chunk filename:

```typescript
import(/* webpackChunkName: "vendor-core" */ "./vendor-core")
// → produces: vendor-core.chunk.js
```

Ensure `chunkFilename` in webpack output supports named chunks:
```javascript
output: {
  chunkFilename: "[name].chunk.js",  // or "[id].chunk.js"
}
```
