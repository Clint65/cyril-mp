# React Widget Template

Templates for a React-based widget using the vendor splitting pattern. Replace `{{WidgetName}}` with PascalCase class name and `{{widgetname}}` with lowercase folder/file name.

## Prerequisites

The project must have the vendor splitting infrastructure in `src/common/`:
- `vendor-core.ts` — Shared React + MUI imports
- `vendor-loader.ts` — Caching lazy loader
- `theme.ts` — Shared MUI theme (optional)

See `references/vendor-splitting.md` for `vendor-core.ts` and `vendor-loader.ts`. The `theme.ts` template is provided below.

### theme.ts — MUI Theme (src/common/theme.ts)

```typescript
import { createTheme } from "@mui/material/styles";

export const theme = createTheme({
  palette: {
    primary: {
      main: "#1976d2",
    },
    secondary: {
      main: "#dc004e",
    },
  },
  typography: {
    fontFamily: '"Roboto", "Helvetica", "Arial", sans-serif',
    fontSize: 14,
  },
  components: {
    MuiButton: {
      styleOverrides: {
        root: {
          textTransform: "none",
        },
      },
    },
  },
});
```

Customize colors and components to match the project's design system.

## IDE File: `{{widgetname}}.ide.ts`

The IDE file is identical to a standard TypeScript widget — React is only used at runtime:

```typescript
import "./{{widgetname}}.ide.scss";
import {
  TWWidgetDefinition,
  autoResizable,
  description,
  property,
  defaultValue,
  bindingTarget,
  bindingSource,
  nonEditable,
  service,
  event,
} from "typescriptwebpacksupport/widgetIDESupport";
import widgetIconUrl from "./images/icon.png";

@description("{{Widget description}}")
@TWWidgetDefinition("{{Widget Display Name}}", autoResizable)
class {{WidgetName}} extends TWComposerWidget {
  @property("NUMBER", defaultValue(600)) Width: number;
  @property("NUMBER", defaultValue(400)) Height: number;

  // Properties
  @property("STRING", defaultValue("Label")) Label: string;
  // @property("INFOTABLE", bindingTarget) Data;
  // @property("BOOLEAN", defaultValue(false), bindingTarget) Disabled: boolean;

  // Events
  @event Clicked;

  // Services
  // @service Refresh;

  widgetIconUrl(): string {
    return widgetIconUrl;
  }

  renderHtml(): string {
    return '<div class="widget-content widget-{{widgetname}}">' +
      '<table height="100%" width="100%"><tr><td valign="middle" align="center">' +
      '<span style="color:white">{{Widget Display Name}}</span>' +
      '</td></tr></table></div>';
  }

  afterRender(): void {}
  beforeDestroy(): void {}
}
```

## Runtime File: `{{widgetname}}.runtime.tsx`

Note the `.tsx` extension for JSX support:

```typescript
import "./{{widgetname}}.runtime.css";
import {
  TWWidgetDefinition,
  property,
  TWEvent,
  event,
  service,
} from "typescriptwebpacksupport/widgetRuntimeSupport";
import widgetHtml from "./{{widgetname}}.html";
import { loadVendorCore, type VendorCoreModule } from "../common/vendor-loader";

@TWWidgetDefinition
class {{WidgetName}} extends TWRuntimeWidget {
  _widgetSelector: string = "";
  _root: any = null;
  _vendorReady: boolean = false;
  _vendor: VendorCoreModule | null = null;

  // Properties
  @property Label: string;
  @event Clicked: TWEvent;

  // Bound property with re-render
  // @property set Disabled(value: boolean) {
  //   this._disabled = value;
  //   if (this._vendorReady) this.renderReactComponent();
  // }

  async afterRender(): Promise<void> {
    // Load vendor chunk (React + MUI) — shared across all widgets
    this._vendor = await loadVendorCore();
    this._vendorReady = true;

    // Create React root
    this._root = this._vendor.createRoot(this.jqElement[0]);
    this.renderReactComponent();
  }

  renderReactComponent(): void {
    if (!this._vendor || !this._root) return;

    const { React, ThemeProvider, Button, theme } = this._vendor;

    this._root.render(
      <ThemeProvider theme={theme}>
        <Button
          variant="contained"
          onClick={() => this.jqElement.triggerHandler("Clicked")}
        >
          {this.Label}
        </Button>
      </ThemeProvider>
    );
  }

  beforeDestroy(domOnly?: boolean): void {
    // domOnly === true: widget is inside a lazy-loaded container (Tab, Accordion)
    //   → unmount React root but keep vendor reference for re-render
    // domOnly === false/undefined: full cleanup
    if (this._root) {
      this._root.unmount();
      this._root = null;
    }
    if (!domOnly) {
      this._vendor = null;
      this._vendorReady = false;
    }
  }

  renderHtml(): string {
    this._widgetSelector = "{{widgetname}}-" + this.jqElementId;
    let html = widgetHtml;
    html = html.replace(/{{widgetname}}-placeholder/g, this._widgetSelector);
    return html;
  }
}
```

## HTML Template: `{{widgetname}}.html`

```html
<div class="widget-content widget-{{widgetname}}-main">
  <div id="{{widgetname}}-placeholder"></div>
</div>
```

## Stylesheet: `{{widgetname}}.runtime.css`

```css
.widget-{{widgetname}}-main {
  width: 100%;
  height: 100%;
}
```

## Pattern: React Component in Separate File

For complex widgets, extract the React component into its own file:

```typescript
// {{widgetname}}-component.tsx
import { getVendorCoreSync } from "../common/vendor-loader";

interface {{WidgetName}}Props {
  label: string;
  disabled?: boolean;
  onClicked?: () => void;
}

export function {{WidgetName}}Component({ label, disabled, onClicked }: {{WidgetName}}Props) {
  const { React, ThemeProvider, Button, theme } = getVendorCoreSync();

  return (
    <ThemeProvider theme={theme}>
      <Button
        variant="contained"
        disabled={disabled}
        onClick={onClicked}
      >
        {label}
      </Button>
    </ThemeProvider>
  );
}
```

```typescript
// {{widgetname}}.runtime.tsx — simplified
import { {{WidgetName}}Component } from "./{{widgetname}}-component";

// In renderReactComponent():
this._root.render(
  <{{WidgetName}}Component
    label={this.Label}
    disabled={this._disabled}
    onClicked={() => this.jqElement.triggerHandler("Clicked")}
  />
);
```

## Pattern: Widget with Diagram Libraries

For widgets needing ReactFlow or other diagram libraries in addition to the core:

```typescript
import { loadAllVendors } from "../common/vendor-loader";
import type { VendorCoreModule } from "../common/vendor-loader";
import type { VendorDiagramModule } from "../common/vendor-loader";

@TWWidgetDefinition
class MyDiagramWidget extends TWRuntimeWidget {
  _vendor: VendorCoreModule | null = null;
  _diagramVendor: VendorDiagramModule | null = null;

  async afterRender(): Promise<void> {
    const { core, diagram } = await loadAllVendors();
    this._vendor = core;
    this._diagramVendor = diagram;
    // ...
  }
}
```
