# Widget Lifecycle Reference

ThingWorx widgets have two distinct lifecycles: one for the **IDE** (Mashup Builder / Composer) and one for the **Runtime** (live mashup viewer).

**Why two separate files?** The IDE and Runtime environments have fundamentally different purposes. The IDE file runs inside the Mashup Builder where developers design mashups — it defines which properties appear in the property panel, what the widget looks like as a placeholder, and how it validates configuration. The Runtime file runs in the browser when end users view a mashup — it handles real data, user interactions, and third-party libraries. Keeping them separate means the IDE doesn't load heavy runtime libraries (echarts, React, etc.), and the runtime doesn't carry design-time validation code. ThingWorx enforces this by loading them from different combined files (`isDevelopment` vs `isRuntime` in metadata.xml). Some code can be shared by placing it in a separate module imported by both files.

## IDE Lifecycle (TWComposerWidget)

The IDE lifecycle governs the widget inside the Mashup Builder where users configure it visually.

### 1. Discovery
When ThingWorx loads the Composer, it discovers all installed widget extensions and adds them to the toolbar palette.

- `widgetProperties()` — Returns widget configuration: name, description, properties with their types, defaults, and binding capabilities
- `widgetEvents()` — Returns event definitions (e.g., `Clicked`, `DataChanged`)
- `widgetServices()` — Returns service definitions (e.g., `Refresh`, `ExportCSV`)
- `widgetIconUrl()` — Returns the URL to the widget's icon displayed in the toolbar

### 2. Creation
When the user drags the widget onto the mashup canvas:

- `afterCreate()` — Called once when the widget is first placed. Use this to set initial property values programmatically
- `afterLoad()` — Called after saved properties are restored from the mashup file (before rendering)

### 3. Rendering
- `renderHtml()` — Must return an HTML string. The root element **must include** the CSS class `widget-content`. This HTML becomes the widget's visual representation in the Composer
- `afterRender()` — Called after the HTML is inserted into the DOM. `this.jqElement` is now available for jQuery operations

### 4. Property Updates
When the user changes a property in the property panel:

- `beforeSetProperty(name, value)` — Validation hook. Return a string to reject the change (the string is shown as an error message). Return nothing to accept
- `afterSetProperty(name, value)` — Called after the property is updated. Return `true` to trigger a full redraw (calls `renderHtml()` + `afterRender()` again)

### 5. Destruction
When the user deletes the widget from the mashup:

- `beforeDestroy()` — Clean up resources: remove event handlers, destroy third-party plugin instances, release DOM references

## Runtime Lifecycle (TWRuntimeWidget)

The Runtime lifecycle governs the widget when a mashup is viewed by an end user.

### 1. Configuration
- `runtimeProperties()` — Returns runtime-specific configuration. Key properties:
  - `needsDataLoadingAndError: true` — Shows loading spinner during data fetch
  - `supportsAutoResize: true` — Enables `resize()` callback on container resize
  - `needsErrorRecovery: true` — Widget can recover from errors

### 2. Property Loading
ThingWorx loads saved property values from the mashup definition. Static (non-bound) values are set directly on the widget object. No callback is fired at this stage — the values are simply available via `this.getProperty('Name')`.

### 3. Rendering
- `renderHtml()` — Returns HTML string. Root element must include `widget-content` class. Use a unique ID pattern: `"mywidget-" + this.jqElementId`
- `afterRender()` — DOM is ready. `this.jqElement` and `this.jqElementId` are available. This is where you:
  - Initialize third-party libraries
  - Set up event listeners
  - Read static property values and perform initial render
  - Start async loading of external libraries via `import()`

### 4. Data Binding Updates
When bound data arrives (from services, other widgets, etc.):

- **With decorators**: The `@property set PropertyName(value)` setter is called automatically
- **Without decorators**: `updateProperty(updatePropertyInfo)` is called with:
  - `updatePropertyInfo.TargetProperty` — Name of the property being updated
  - `updatePropertyInfo.SinglePropertyValue` — Scalar value
  - `updatePropertyInfo.ActualDataRows` — Row data for INFOTABLE types
  - `updatePropertyInfo.DataShape` — The data shape definition
  - `updatePropertyInfo.SelectedRowIndices` — Selected row indices

### 5. Service Invocation
- `serviceInvoked(serviceName)` — Called when a bound service is triggered (e.g., the user clicks a button elsewhere that triggers this widget's `Refresh` service)

### 6. Selection Changes
- `handleSelectionUpdate(propertyName, selectedRows, selectedRowIndices)` — Called when selection changes on a bound infotable property

### 7. Resize
- `resize(width, height)` — Called when the widget's container is resized (only if `supportsAutoResize: true` in `runtimeProperties()`)

### 8. Destruction
- `beforeDestroy(domOnly?)` — Clean up everything:
  - Remove event listeners
  - Destroy third-party library instances
  - Unmount React roots
  - Release references
  - **Important for lazy-loaded containers**: When `domOnly === true`, only destroy DOM elements (the widget might be re-rendered when the container becomes visible again). When `domOnly === false` or undefined, do a full cleanup

## Lifecycle Diagram

```
IDE:     Discovery → Creation → Render → [Property Updates] → Destruction
           │                      │              │
     widgetProperties()    renderHtml()    beforeSetProperty()
     widgetEvents()        afterRender()   afterSetProperty()
     widgetServices()
     widgetIconUrl()

Runtime: Config → Props Load → Render → [Binding Updates] → Destruction
           │                     │              │
    runtimeProperties()   renderHtml()    @property set X()
                          afterRender()   updateProperty()
                                          serviceInvoked()
                                          handleSelectionUpdate()
                                          resize()
```

## Critical Rules

1. **Never assume property order**: `afterRender()` and property setters can fire in any order. Always check that all required data is present before rendering.

2. **Always clean up**: `beforeDestroy()` must undo everything `afterRender()` set up. Memory leaks accumulate across mashup navigation.

3. **Use `jqElementId` for unique IDs**: Multiple instances of the same widget can exist on a mashup. Always use `this.jqElementId` to create unique DOM element IDs.

4. **`renderHtml()` is synchronous**: It must return an HTML string immediately. Async operations (library loading, data fetching) go in `afterRender()`.
