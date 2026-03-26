# Workflow: Modify an Existing Widget

Modify the behavior, properties, appearance, or logic of an existing widget.

## Step 1: Understand the current widget

Read these files in order:
1. **IDE file** (`*.ide.ts`) — Understand the property definitions, events, services
2. **Runtime file** (`*.runtime.ts` or `*.runtime.tsx`) — Understand the rendering logic, data flow, lifecycle usage
3. **HTML template** (`*.html`) — Understand the DOM structure
4. **Stylesheets** (`*.scss` or `*.css`) — Understand the styling

Take note of:
- Which properties are binding targets (receive data) vs binding sources (emit data)
- How the render pattern works (guard checks, what triggers re-render)
- Whether the widget uses lazy loading for external libraries
- Whether it's part of a multi-widget project with vendor splitting

## Step 2: Understand the modification request

Common modification types:

### Adding a new property

1. Add `@property` declaration in the IDE file with appropriate type and aspects
2. Add `@property` (or `@property set`) in the runtime file
3. If the property is a binding target, add a setter that triggers re-render
4. Update the `render()` method to use the new property

**Concrete example — adding a `Color` property (STRING, bindingTarget) to an existing widget:**

IDE file — add among the other `@property` declarations:
```typescript
@property("STRING", selectOptions([
  { value: "primary", text: "Primary" },
  { value: "success", text: "Success" },
  { value: "warning", text: "Warning" },
  { value: "danger", text: "Danger" }
]), defaultValue("primary"), bindingTarget) Color: string;
```

Runtime file — add a property setter that triggers re-render:
```typescript
@property set Color(value: string) {
  this._color = value;
  this.render();
}
```

And in `afterRender()`, read the initial value:
```typescript
afterRender(): void {
  this._color = this.Color || "primary";
  // ... existing init code
  this.render();
}
```

Then use `this._color` in the `render()` method to apply the color.

### Adding a new event
1. Add `@event EventName;` in the IDE file
2. Add `@event EventName: TWEvent;` in the runtime file
3. Trigger with `this.EventName()` or `this.jqElement.triggerHandler("EventName")`

### Adding a new service
1. Add `@service ServiceName;` in the IDE file
2. Add `@service ServiceName(): void { ... }` in the runtime file

### Changing rendering logic
1. Modify the `render()` method in the runtime file
2. If the DOM structure changes, update the HTML template
3. Update stylesheets if needed

### Adding an external library
1. Add dependency to `package.json`
2. Import with dynamic `import()` in `afterRender()` — read `references/combined-extensions.md`
3. Store reference for cleanup in `beforeDestroy()`

### Fixing a bug
1. Read the widget lifecycle reference (`references/widget-lifecycle.md`)
2. Check for common issues:
   - Property order assumption (afterRender vs setter order)
   - Missing null checks in render
   - Missing cleanup in beforeDestroy
   - JSON parsing (string vs object)

## Step 3: Make the changes

Apply modifications following these rules:

1. **Keep IDE and Runtime in sync** — Every property, event, and service must be declared in both files
2. **Maintain the defensive rendering pattern** — Check all required data before rendering
3. **Handle JSON properties defensively** — Always parse: `typeof value === "string" ? JSON.parse(value) : value`
4. **Use unique DOM IDs** — Always include `this.jqElementId` in generated element IDs
5. **Clean up in beforeDestroy** — Undo everything afterRender set up

## Step 4: Verify the changes

Run the build to check for compilation errors:
```bash
cd {{projectPath}} && npm run testbuild
```

If it compiles, verify packaging:
```bash
npm run buildDev
```

## Step 5: Guide the user for testing

Tell the user:
1. Upload the extension to ThingWorx (manual upload or `npm run upload`)
2. Open a mashup containing the widget in the Mashup Builder
3. Verify the new properties appear in the property panel
4. Run the mashup to test runtime behavior
5. If debugging is needed, use the debug workflow (`workflows/debug-widget.md`)
