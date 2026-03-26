# TypeScript Patterns Reference

Widgets are written as TypeScript classes using decorators from the `typescriptwebpacksupport` package. Each widget has two files: an IDE class (for the Composer) and a Runtime class.

## Imports

### IDE file
```typescript
import {
  TWWidgetDefinition, autoResizable, description,
  property, defaultValue, bindingTarget, bindingSource,
  nonEditable, hidden, selectOptions, sourcePropertyName,
  willSet, didSet, container,
  service, event
} from "typescriptwebpacksupport/widgetIDESupport";
```

### Runtime file
```typescript
import {
  TWWidgetDefinition, property, canBind,
  TWEvent, event, service
} from "typescriptwebpacksupport/widgetRuntimeSupport";
```

## Base Classes

- **`TWComposerWidget`** — IDE class. Provides: `jqElement`, `getProperty()`, `setProperty()`, `renderHtml()`, `afterRender()`, `beforeDestroy()`, `widgetProperties()`, `widgetEvents()`, `widgetServices()`, `widgetIconUrl()`, `beforeSetProperty()`, `afterSetProperty()`
- **`TWRuntimeWidget`** — Runtime class. Provides: `jqElement`, `jqElementId`, `getProperty()`, `setProperty()`, `renderHtml()`, `afterRender()`, `beforeDestroy()`, `updateProperty()`, `serviceInvoked()`, `handleSelectionUpdate()`, `resize()`

These classes are globally available (declared in type definitions) — no import needed.

## Class-Level Decorators

### `@TWWidgetDefinition`
Registers the class as a ThingWorx widget.

**IDE form** (with display name and aspects):
```typescript
@description("My widget description")
@TWWidgetDefinition("My Widget Display Name", autoResizable)
class MyWidget extends TWComposerWidget { ... }
```

**Runtime form** (bare):
```typescript
@TWWidgetDefinition
class MyWidget extends TWRuntimeWidget { ... }
```

### Widget Aspects (IDE only)
- `autoResizable` — Widget supports auto-resize
- `container` — Widget can contain other widgets
- `borderWidth(px)` — Specifies border width

## Property Decorators

### IDE Properties
```typescript
// Simple property with default
@property("STRING", defaultValue("Hello")) Label: string;

// Binding target (receives data)
@property("INFOTABLE", bindingTarget) Data;

// Binding source (emits data)
@property("STRING", bindingSource) SelectedValue: string;

// Both target and source
@property("JSON", bindingTarget, bindingSource, defaultValue("{}")) Config;

// Non-editable (hidden from property panel but still bindable)
@property("STRING", bindingSource, nonEditable) Output: string;

// Completely hidden (not visible in property panel at all)
@property("STRING", hidden) InternalState: string;

// Dropdown selection
@property("STRING", selectOptions([
  { value: "small", text: "Small" },
  { value: "medium", text: "Medium" },
  { value: "large", text: "Large" }
]), defaultValue("medium")) Size: string;

// Field name linked to an infotable property
@property("FIELDNAME", sourcePropertyName("Data")) IdField;

// Validation callback before property change (IDE)
@property("NUMBER", willSet("validateWidth"), defaultValue(600)) Width: number;

// Callback after property change (IDE)
@property("STRING", didSet("onLabelChange"), defaultValue("")) Label: string;
```

### Runtime Properties

In the runtime, use `@property` with setters to react to binding updates:

```typescript
// Simple read-only property
@property Label: string;

// Property with setter — called when binding data arrives
@property set Data(value: any) {
  this._data = value;
  this.render();
}

// Property with binding validation
@property(canBind("validateData")) set Data(value: any) {
  this._data = value;
  this.render();
}

// Property mapped to a different IDE name
@property("clickedAmount") timesClicked: number;
```

### Supported Base Types
`STRING`, `NUMBER`, `INTEGER`, `BOOLEAN`, `DATETIME`, `TIMESPAN`, `INFOTABLE`, `JSON`, `XML`, `QUERY`, `HYPERLINK`, `IMAGELINK`, `HTML`, `TEXT`, `TAGS`, `SCHEDULE`, `FIELDNAME`, `THINGNAME`, `THINGSHAPENAME`, `THINGTEMPLATENAME`, `DATASHAPENAME`, `MASHUPNAME`, `MENUNAME`, `BASETYPENAME`, `STYLEDEFINITIONNAME`, `STATEFORMATNAME`, `VEC2`, `VEC3`, `VEC4`, `NOTHING`

## Events

### IDE
```typescript
@event Clicked;
@event DataChanged;
```

### Runtime
```typescript
@event Clicked: TWEvent;

// Trigger the event:
this.Clicked();  // or: this.jqElement.triggerHandler("Clicked");
```

## Services

### IDE
```typescript
@service Refresh;
@service ExportCSV;
@service ShowLoading;
```

### Runtime

Services are methods triggered by bindings from other widgets (e.g., a button's "Clicked" event bound to this widget's "Refresh" service). The method body runs when the service is invoked.

```typescript
@service Refresh(): void {
  // Re-render with current data — useful when external state changes
  this.render();
}

@service ExportCSV(): void {
  if (!this._data) return;
  const csv = this._data.rows
    .map(row => Object.values(row).join(","))
    .join("\n");
  const blob = new Blob([csv], { type: "text/csv" });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = "export.csv";
  a.click();
  URL.revokeObjectURL(url);
}

@service ShowLoading(): void {
  const container = document.getElementById(this._widgetSelector);
  if (container) container.innerHTML = '<div class="loading-spinner"></div>';
}
```

## Complete IDE Example

```typescript
import "./mywidget.ide.scss";
import {
  TWWidgetDefinition, autoResizable, description,
  property, defaultValue, bindingTarget, bindingSource,
  nonEditable, selectOptions, service, event
} from "typescriptwebpacksupport/widgetIDESupport";
import widgetIconUrl from "./images/icon.png";

@description("My Custom Widget")
@TWWidgetDefinition("My Widget", autoResizable)
class MyWidget extends TWComposerWidget {
  @property("NUMBER", defaultValue(600)) Width: number;
  @property("NUMBER", defaultValue(400)) Height: number;

  @property("STRING", defaultValue("Title")) Label: string;
  @property("INFOTABLE", bindingTarget) Data;
  @property("STRING", bindingSource, nonEditable) SelectedId: string;

  @event Clicked;
  @service Refresh;

  widgetIconUrl(): string {
    return widgetIconUrl;
  }

  renderHtml(): string {
    return '<div class="widget-content widget-mywidget">' +
      '<table height="100%" width="100%"><tr><td valign="middle" align="center">' +
      '<span>My Widget</span></td></tr></table></div>';
  }

  afterRender(): void {}
  beforeDestroy(): void {}
}
```

## Complete Runtime Example

```typescript
import "./mywidget.runtime.scss";
import { TWWidgetDefinition, property, TWEvent, event, service } from "typescriptwebpacksupport/widgetRuntimeSupport";
import mywidgetHtml from "./mywidget.html";

@TWWidgetDefinition
class MyWidget extends TWRuntimeWidget {
  _widgetSelector: string;
  _data: any = undefined;

  @property Label: string;
  @property SelectedId: string;
  @event Clicked: TWEvent;

  @property set Data(value: any) {
    this._data = value;
    this.render();
  }

  @service Refresh(): void {
    this.render();
  }

  afterRender(): void {
    // Static properties are available here
    this.render();
  }

  render(): void {
    if (!this._data) return;
    const container = document.getElementById(this._widgetSelector);
    if (!container) return;
    // Build the UI using this._data
    container.innerHTML = `<h2>${this.Label}</h2>`;
  }

  beforeDestroy(): void {
    // Clean up event listeners, third-party libs
  }

  renderHtml(): string {
    this._widgetSelector = "mywidget-" + this.jqElementId;
    let html = mywidgetHtml;
    html = html.replace(/mywidget-placeholder/g, this._widgetSelector);
    return html;
  }
}
```
