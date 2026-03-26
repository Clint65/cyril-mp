# Workflow: Migrate a JavaScript Widget to TypeScript

Convert an existing JavaScript widget to TypeScript with decorator syntax.

## Step 1: Understand the existing JS widget

Read the JavaScript widget files:
1. **IDE file** (`*.ide.js`) — Look for the `TW.IDE.Widgets.widgetName = function() { ... }` pattern
2. **Runtime file** (`*.runtime.js`) — Look for `TW.Runtime.Widgets.widgetName = function() { ... }`
3. **HTML, CSS files** — These can often stay as-is

Map the JS patterns to their TypeScript equivalents:

| JavaScript Pattern | TypeScript Equivalent |
|---|---|
| `TW.IDE.Widgets.name = function() { ... }` | `@TWWidgetDefinition class Name extends TWComposerWidget` |
| `this.widgetProperties = function() { return { name: "...", properties: { ... } } }` | `@property("TYPE", aspects) PropName: type;` |
| `this.widgetEvents = function() { return [{ name: "Clicked" }] }` | `@event Clicked;` |
| `this.widgetServices = function() { return [{ name: "Refresh" }] }` | `@service Refresh;` |
| `this.renderHtml = function() { ... }` | `renderHtml(): string { ... }` |
| `this.afterRender = function() { ... }` | `afterRender(): void { ... }` |
| `this.updateProperty = function(info) { ... }` | `@property set PropName(value) { ... }` |
| `this.beforeDestroy = function() { ... }` | `beforeDestroy(): void { ... }` |

## Step 2: Extract properties from widgetProperties()

The JS `widgetProperties()` function returns an object like:
```javascript
this.widgetProperties = function () {
  return {
    name: "MyWidget",
    description: "My widget",
    properties: {
      Label: { baseType: "STRING", defaultValue: "Hello", isBindingTarget: true },
      Data: { baseType: "INFOTABLE", isBindingTarget: true },
      Output: { baseType: "STRING", isBindingSource: true, isEditable: false }
    }
  };
};
```

Convert each property to a decorator:
```typescript
@property("STRING", defaultValue("Hello"), bindingTarget) Label: string;
@property("INFOTABLE", bindingTarget) Data;
@property("STRING", bindingSource, nonEditable) Output: string;
```

Property aspect mapping:
| JS Property | TypeScript Aspect |
|---|---|
| `isBindingTarget: true` | `bindingTarget` |
| `isBindingSource: true` | `bindingSource` |
| `isEditable: false` | `nonEditable` |
| `defaultValue: X` | `defaultValue(X)` |
| `isVisible: false` | `hidden` |
| `selectOptions: [...]` | `selectOptions([...])` |

## Step 3: Convert updateProperty to typed setters

The JS `updateProperty()` function typically has a switch/if on `TargetProperty`:
```javascript
this.updateProperty = function (updatePropertyInfo) {
  if (updatePropertyInfo.TargetProperty === "Data") {
    this._data = updatePropertyInfo.ActualDataRows;
    this.render();
  }
  if (updatePropertyInfo.TargetProperty === "Config") {
    this._config = JSON.parse(updatePropertyInfo.SinglePropertyValue);
    this.render();
  }
};
```

Convert each case to a typed property setter:
```typescript
@property set Data(value: any) {
  this._data = value;
  this.render();
}

@property set Config(value: any) {
  this._config = typeof value === "string" ? JSON.parse(value) : value;
  this.render();
}
```

## Step 4: Set up TypeScript infrastructure

If the project doesn't already have TypeScript support:
1. Add `tsconfig.json` (see `templates/project-scaffold.md`)
2. Add `ts-loader`, `typescript`, `typescriptwebpacksupport` to `devDependencies`
3. Create `src/@types/assets/index.d.ts` for HTML/image imports
4. Update `webpack/webpack.common.js` to handle `.ts` files (add ts-loader rule)

## Step 5: Rename and convert files

1. Rename `*.ide.js` → `*.ide.ts`
2. Rename `*.runtime.js` → `*.runtime.ts`
3. Rename `*.css` → `*.scss` (optional but recommended)
4. Rewrite each file using the decorator patterns from `references/typescript-patterns.md`
5. Add proper imports at the top of each file

## Step 6: Verify and fix

```bash
npm run testbuild
```

Common migration issues:
- **`this` context**: In JS, `this` inside nested functions may lose widget context. In TS class methods, `this` works correctly. Check arrow functions vs regular functions.
- **jQuery usage**: Keep using `this.jqElement` — jQuery is still available as an external.
- **Global variables**: JS widgets sometimes store state on `this` directly. In TS, declare class members explicitly.
- **Missing types**: Start with `any` types and refine later — getting the build working first is more important than perfect typing.
