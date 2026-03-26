# TypeScript Widget Template

Templates for a standard TypeScript widget. Replace `{{WidgetName}}` with the actual class name (PascalCase) and `{{widgetname}}` with the lowercase folder/file name.

## IDE File: `{{widgetname}}.ide.ts`

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
  selectOptions,
  service,
  event,
} from "typescriptwebpacksupport/widgetIDESupport";
import widgetIconUrl from "./images/icon.png";

@description("{{Widget description}}")
@TWWidgetDefinition("{{Widget Display Name}}", autoResizable)
class {{WidgetName}} extends TWComposerWidget {
  @property("NUMBER", defaultValue(600)) Width: number;
  @property("NUMBER", defaultValue(400)) Height: number;

  // Add properties here
  // @property("STRING", defaultValue("")) Label: string;
  // @property("INFOTABLE", bindingTarget) Data;
  // @property("STRING", bindingSource, nonEditable) SelectedValue: string;

  // Add events here
  // @event Clicked;

  // Add services here
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

## Runtime File: `{{widgetname}}.runtime.ts`

```typescript
import "./{{widgetname}}.runtime.scss";
import {
  TWWidgetDefinition,
  property,
  TWEvent,
  event,
  service,
} from "typescriptwebpacksupport/widgetRuntimeSupport";
import widgetHtml from "./{{widgetname}}.html";

@TWWidgetDefinition
class {{WidgetName}} extends TWRuntimeWidget {
  _widgetSelector: string;

  // Internal state — use _ prefix for internal variables
  // _data: any = undefined;

  // Simple properties (read static value in afterRender)
  // @property Label: string;

  // Binding source properties
  // @property SelectedValue: string;

  // Events
  // @event Clicked: TWEvent;

  // Bound properties with setters
  // @property set Data(value: any) {
  //   this._data = value;
  //   this.render();
  // }

  // Services
  // @service Refresh(): void {
  //   this.render();
  // }

  afterRender(): void {
    // Read static property values here
    // Initialize the widget
    this.render();
  }

  render(): void {
    // Guard: check all required data is present
    // if (!this._data) return;

    const container = document.getElementById(this._widgetSelector);
    if (!container) return;

    // Build UI here
  }

  beforeDestroy(domOnly?: boolean): void {
    // Clean up event listeners, third-party libs, DOM references
    // domOnly === true: widget is inside a lazy-loaded container (Tab, Accordion)
    //   → destroy DOM only, keep state for re-render when container becomes visible again
    // domOnly === false/undefined: full cleanup, widget is being permanently removed
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
  <div class="container" id="{{widgetname}}-placeholder"></div>
</div>
```

## IDE Stylesheet: `{{widgetname}}.ide.scss`

```scss
.widget-{{widgetname}} {
  background-color: var(--primary-color, #2196F3);
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-family: sans-serif;
}
```

## Runtime Stylesheet: `{{widgetname}}.runtime.scss`

```scss
.widget-{{widgetname}}-main {
  width: 100%;
  height: 100%;
  overflow: auto;

  .container {
    width: 100%;
    height: 100%;
  }
}
```

## Widget Icon

Place a PNG icon (ideally 48x48 or 64x64) at `{{widgetname}}/images/icon.png`. This icon appears in the Mashup Builder toolbar.

## With External Library (Lazy Loading)

When the widget needs an external library, lazy-load it in `afterRender()`:

```typescript
@TWWidgetDefinition
class {{WidgetName}} extends TWRuntimeWidget {
  _lib: any = null;

  async afterRender(): Promise<void> {
    // Library loaded as a separate chunk — not in CombinedExtensions.js
    this._lib = await import(/* webpackChunkName: "{{libname}}" */ "{{libname}}");
    this.render();
  }

  beforeDestroy(): void {
    // Clean up library resources
    this._lib = null;
  }
}
```
