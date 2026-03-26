# CombinedExtensions.js and Lazy Loading Reference

## The CombinedExtensions.js Mechanism

When a widget extension is imported into ThingWorx, the platform **concatenates** all widget JavaScript files into combined files:

- All `*.ide.js` files (marked `isDevelopment="true"`) → `CombinedExtensions.js` for the Composer
- All `*.runtime.js` files (marked `isRuntime="true"`) → `CombinedExtensions.js` for the Runtime

This combined file is loaded on **every mashup page load**, regardless of whether the widget is actually used on that mashup.

### Consequences

1. **Every byte in your `*.runtime.js` is loaded everywhere**. If you bundle a 500KB library directly into the runtime file, every mashup page becomes 500KB heavier, even if the widget isn't used.

2. **Syntax errors break everything**. A syntax error in any widget's code corrupts the entire combined file, potentially breaking all widgets on the platform.

3. **Global namespace pollution**. All widget code runs in the same scope. Use unique class names and avoid polluting `window`.

## The Solution: Dynamic Imports (Lazy Loading)

Webpack's dynamic `import()` splits code into separate **chunk files** that are loaded on demand, not bundled into the main runtime file:

```typescript
@TWWidgetDefinition
class MyChartWidget extends TWRuntimeWidget {
  _echarts: any;

  async afterRender(): Promise<void> {
    // echarts is NOT in CombinedExtensions.js
    // It loads as a separate chunk only when this widget renders
    const echarts = await import("echarts");
    this._echarts = echarts;
    this.initChart();
  }
}
```

### How It Works

1. Webpack sees `import("echarts")` and creates a separate chunk file (e.g., `123.chunk.js`)
2. The main `mywidget.runtime.js` stays small — it only contains the widget's own code plus a reference to load the chunk
3. At runtime, when `afterRender()` calls `import("echarts")`, webpack fetches the chunk from the `publicPath` URL
4. The chunk is loaded via a script tag and cached by the browser

### publicPath is Critical

The `publicPath` in `webpack.common.js` tells webpack where to find chunks at runtime:

```javascript
publicPath: `/Thingworx/Common/extensions/${packageName}/ui/${widget}/`
```

This maps to where ThingWorx physically serves extension files. If this path is wrong, chunk loading fails silently.

### What Goes Into CombinedExtensions.js

**YES** (unavoidable — keep these minimal):
- Widget class definition and decorator registration
- Property/event/service declarations
- `renderHtml()` and basic lifecycle methods
- Import statements for CSS/SCSS (extracted by MiniCssExtractPlugin)
- Small utility functions

**NO** (must be lazy loaded):
- Large external libraries (echarts, fabric.js, marked, highlight.js, etc.)
- Framework code (React, Vue, MUI, etc.)
- Heavy computation modules

## Lazy Loading Patterns

### Pattern 1: Simple Dynamic Import
For a single external library:
```typescript
async afterRender(): Promise<void> {
  const { default: echarts } = await import("echarts");
  this._chart = echarts.init(this.jqElement[0]);
}
```

### Pattern 2: Named Chunk
Use webpack magic comments to control chunk names:
```typescript
const { default: echarts } = await import(/* webpackChunkName: "echarts" */ "echarts");
```

### Pattern 3: Internal Module Split
Split your own heavy code into a separate file:
```typescript
// heavy-logic.ts — contains complex calculations
export function processData(data: any) { /* heavy code */ }

// widget.runtime.ts
async afterRender(): Promise<void> {
  const { processData } = await import("./heavy-logic");
  this._processData = processData;
}
```

### Pattern 4: Vendor Splitting (for multi-widget projects)
When multiple widgets in the same extension share large libraries (React, MUI, etc.), use the vendor splitting pattern described in `references/vendor-splitting.md`.

## beforeDestroy with Lazy-Loaded Containers

ThingWorx containers (Tabs, Accordion, etc.) can lazily load/unload child widgets as panels become visible/hidden. When a container hides a panel:

1. It calls `beforeDestroy(true)` — `domOnly = true`
2. The widget should destroy its DOM but keep its state
3. When the panel becomes visible again, `renderHtml()` + `afterRender()` are called again

```typescript
beforeDestroy(domOnly?: boolean): void {
  if (domOnly) {
    // Only destroy DOM — keep state for re-render
    if (this._chart) {
      this._chart.dispose();
      this._chart = null;
    }
  } else {
    // Full cleanup — widget is being removed permanently
    if (this._chart) {
      this._chart.dispose();
      this._chart = null;
    }
    this._data = null;
    this._config = null;
  }
}
```

## Debugging Chunk Loading Issues

If chunks fail to load at runtime:

1. **Check `publicPath`** — Open browser DevTools Network tab, look for 404s on chunk files
2. **Verify the chunk exists** in `dist/ui/{widgetName}/`
3. **Check `chunkLoadingGlobal`** — Must be unique per extension to avoid conflicts
4. **Check ThingWorx extension import** — The zip must include all chunk files
