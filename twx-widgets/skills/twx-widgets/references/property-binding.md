# Property Binding Reference

Understanding how ThingWorx handles property values is critical for writing correct widgets. Properties can receive values from two sources: static configuration in the Composer, and dynamic bindings from services or other widgets.

## How Properties Work

### Static Values (set in Composer)
When a user sets a property value directly in the Mashup Builder's property panel, that value is stored in the mashup definition file. At runtime, ThingWorx loads these values **before** calling `renderHtml()`. They are available via `this.getProperty('Name')` or directly as class members (with `@property` decorator).

### Bound Values (from services or other widgets)
When a property is bound to a data source (service output, another widget's property), the value arrives asynchronously at runtime. With the `@property` decorator, the setter is called automatically. Without decorators, `updateProperty()` is called.

### Combined: Default + Binding
A property can have a default/static value AND be bound. The static value is loaded first, then the binding update overwrites it when data arrives.

## The Order Problem

**There is no guaranteed order** between `afterRender()` and property setter calls:

```
Possible order A:            Possible order B:
1. renderHtml()              1. renderHtml()
2. afterRender()             2. @property set Data(value)
3. @property set Data(value) 3. afterRender()
4. @property set Config(value) 4. @property set Config(value)

Multiple bound properties can also arrive in any order relative to each other.
```

## Defensive Rendering Pattern

The correct approach: **always check that all required data is present before rendering**.

```typescript
@TWWidgetDefinition
class MyWidget extends TWRuntimeWidget {
  _data: any = undefined;
  _config: any = undefined;

  @property Label: string;   // Static — always available in afterRender

  @property set Data(value: any) {
    this._data = value;
    this.render();            // Try to render after each update
  }

  @property set Config(value: any) {
    this._config = typeof value === "string" ? JSON.parse(value) : value;
    this.render();
  }

  afterRender(): void {
    // Read static values and any already-set bound values
    this._data = this._data || this.Data;
    this._config = this._config || this.Config;
    this.render();
  }

  render(): void {
    // Guard: don't render until all required data is present
    if (!this._data) return;
    if (!this._config) return;

    // Now safe to render
    const container = document.getElementById(this._widgetSelector);
    if (!container) return;
    // ... build UI
  }
}
```

## JSON Property Gotcha

JSON properties coming from the Composer are often strings, while bound values are objects. Always handle both:

```typescript
@property set Options(value: any) {
  this._options = typeof value === "string" ? JSON.parse(value) : value;
  this.render();
}
```

## INFOTABLE Structure

INFOTABLE values have this structure:
```typescript
{
  rows: [
    { field1: "value1", field2: 42 },
    { field1: "value2", field2: 17 }
  ],
  dataShape: {
    fieldDefinitions: {
      field1: { name: "field1", baseType: "STRING" },
      field2: { name: "field2", baseType: "NUMBER" }
    }
  }
}
```

Access data rows via `value.rows[0].fieldName`. The `dataShape` is useful for dynamic rendering (building columns from field definitions).

## updateProperty() Info Object

When not using decorators, `updateProperty()` receives:
```typescript
updateProperty(info: {
  TargetProperty: string;           // Which property is being updated
  RawSinglePropertyValue: any;      // Raw value
  SinglePropertyValue: any;         // Processed value
  ActualDataRows: any[];            // Row data for INFOTABLEs
  DataShape: any;                   // Data shape definition
  SelectedRowIndices: number[];     // Currently selected rows
  IsBoundToSelectedRows: boolean;   // Whether bound to selection
}): void { }
```

## Emitting Data — bindingSource

To emit data back to other widgets or services:

```typescript
// Update the property value
this.setProperty("SelectedId", "123");

// Trigger the property change event
this.jqElement.triggerHandler("SelectedId");
```

With the `@property` decorator as `bindingSource`, simply assigning the value works:
```typescript
this.SelectedId = "123";
// Then trigger the event if other widgets need to react:
this.jqElement.triggerHandler("SelectedId");
```

## Selection Pattern

For widgets that support row selection on an INFOTABLE:

```typescript
// Update selection
this.updateSelection("Data", [selectedRowIndex]);

// Handle incoming selection changes
handleSelectionUpdate(propertyName: string, selectedRows: any[], selectedRowIndices: number[]): void {
  if (propertyName === "Data") {
    this.highlightRows(selectedRowIndices);
  }
}
```
