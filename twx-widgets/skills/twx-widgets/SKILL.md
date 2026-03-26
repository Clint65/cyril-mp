---
name: twx-widgets
description: >
  Develop, modify, and debug custom ThingWorx widgets using TypeScript/SASS or React+TypeScript.
  This skill handles CLIENT-SIDE widget code (TypeScript/React running in the browser) — for
  server-side ThingWorx services, ThingShapes XML, dashboards, or mashup XML generation, use
  twx-databox instead.
  Use this skill whenever the user wants to create a new widget project, scaffold a widget,
  add a widget to an existing multi-widget project, modify widget source code, understand the
  widget lifecycle, debug a widget in the browser, convert a JS widget to TypeScript, or
  build/package a widget extension.
  Triggers on: "create widget", "new widget", "widget property", "widget lifecycle", "debug widget",
  "CombinedExtensions", "mashup widget", "TWRuntimeWidget", "TWComposerWidget",
  "typescriptwebpacksupport", "widget extension", "afterRender", "renderHtml", "beforeDestroy",
  "widget runtime", "widget ide", or any custom widget development task for the ThingWorx platform.
  Also triggers when the user mentions files in ~/Documents/01_Sources/01_Databox/01_Widgets/.
---

# ThingWorx Widget Development Skill

This skill assists with developing custom ThingWorx widgets using a **Webpack-based build pipeline** (not the standard Ant packaging). Widgets can be written in:

- **TypeScript + SASS** (preferred) — using `typescriptwebpacksupport` decorators
- **React + TypeScript** — with MUI and vendor splitting for shared libraries
- **JavaScript + CSS** — legacy approach, supported for reading/modifying existing widgets but not recommended for new ones. No dedicated template — use the TS template as reference and adapt. For converting JS widgets to TypeScript, see `workflows/migrate-js-to-ts.md`

## First reflex: read before writing

Before generating any widget code, read at least one existing example project to match the user's conventions. The example projects in `~/Documents/01_Sources/01_Databox/01_Widgets/` are the ground truth — they contain the actual patterns, dependency versions, and webpack configurations that work in production. Generating code from memory alone risks subtle incompatibilities (wrong import paths, missing webpack config details, outdated decorator syntax). Read the closest example first, then adapt.

## Example projects

| Project | Type | Description |
|---------|------|-------------|
| `databox-twx-displayprops` | TypeScript | Simple widget, good starting template |
| `databox-twx-echarts` | TypeScript | Complex widget with lazy loading, themes, events |
| `databox-twx-grid` | TypeScript | Data grid with services, selection handling |
| `databox-twx-react` | React+TS | 18 sub-widgets, vendor splitting, MUI |
| `databox-twx-gridstack` | TypeScript | Layout container widget |
| `databox-twx-llmchat` | TypeScript | Chat widget with markdown rendering |
| `databox-twx-synoptic` | TypeScript | Canvas-based widget (fabric.js) |
| `databox-twx-explodeinfotable` | JavaScript | Simple JS widget |
| `databox-twx-systemdesign` | JavaScript | Web Components (11 basic UI widgets) |

## Key principles

### 1. CombinedExtensions.js — keep it small

ThingWorx concatenates all `*.runtime.js` files into a single `CombinedExtensions.js` loaded on **every mashup**, even if the widget isn't used. This means:

- The main runtime bundle must stay minimal
- External libraries MUST be loaded via dynamic `import()` so they become separate webpack chunks
- The `publicPath` in webpack output (`/Thingworx/Common/extensions/{package}/ui/{widget}/`) enables chunk loading at runtime
- Read `references/combined-extensions.md` for the full mechanism and patterns

### 2. Property binding — order is not guaranteed

- Static values (set in Composer) are available in `afterRender()` via `this.getProperty()`
- Bound values arrive via property setters (decorated with `@property set`)
- A property can have BOTH a default value AND be bound
- `afterRender()` and property setters can fire in **any order**
- The widget must handle partial state: render only when all required data is present
- Read `references/property-binding.md` for patterns

### 3. Widget lifecycle

Two distinct lifecycles exist — IDE (Composer) and Runtime. Understanding both is essential for writing correct widgets. Read `references/widget-lifecycle.md` for the complete reference.

### 4. Vendor splitting for frameworks

When using React, Vue.js, or any large framework across multiple widgets in the same project, use the **vendor splitting pattern** to share libraries via a single lazy-loaded chunk. Read `references/vendor-splitting.md`.

## Routing — determine what the user needs

Analyze the user's request and route to the appropriate workflow:

### Create a new widget project from scratch
The user wants to start a new widget extension project.
→ Read `workflows/create-project.md`

### Add a widget to an existing project
The user has an existing multi-widget project and wants to add another widget.
→ Read `workflows/add-widget.md`

### Modify an existing widget
The user wants to change behavior, add properties, fix bugs in an existing widget.
→ Read `workflows/modify-widget.md`

### Debug a widget in the browser
The user wants to debug a running widget in Chrome.
→ Read `workflows/debug-widget.md`

### Build and package a widget
The user wants to compile, package, or verify a widget extension builds correctly.
→ Read `workflows/build-package.md`

### Migrate a JavaScript widget to TypeScript
The user has an existing JS widget and wants to convert it to TypeScript with decorators.
→ Read `workflows/migrate-js-to-ts.md`

## Reference files

Read these on demand — they provide detailed technical documentation:

| File | When to read |
|------|-------------|
| `references/widget-lifecycle.md` | Understanding IDE/Runtime lifecycle methods |
| `references/typescript-patterns.md` | Writing widgets with decorators and base classes |
| `references/property-binding.md` | Handling static values, bindings, and data flow |
| `references/webpack-build.md` | Webpack configuration details |
| `references/combined-extensions.md` | CombinedExtensions.js mechanism and lazy loading |
| `references/vendor-splitting.md` | Sharing frameworks across widgets (React, Vue, etc.) |

## Template files

Templates contain code scaffolds ready to use:

| File | Content |
|------|---------|
| `templates/project-scaffold.md` | Full project structure (package.json, tsconfig, .gitignore, @types) |
| `templates/ts-widget.md` | TypeScript widget template (IDE + Runtime + HTML + SCSS) |
| `templates/react-widget.md` | React widget template with vendor loading + theme.ts |
| `templates/webpack-common.md` | Main webpack build configuration (multi-widget, loaders, plugins) |
| `templates/webpack-plugins.md` | Custom webpack plugins (metadata generator, source URL updater, upload) |

## Common pitfalls

These are the most frequent bugs in ThingWorx widgets — check for them during code review:

1. **Assuming property order**: Writing code in `afterRender()` that relies on bound data being already set. The data may arrive after `afterRender()` — always use guard checks in `render()`.

2. **JSON string vs object**: Properties set in the Composer arrive as strings, but bound values arrive as objects. Always handle both: `typeof value === "string" ? JSON.parse(value) : value`.

3. **Missing `beforeDestroy()` cleanup**: Every event listener, third-party library instance, or DOM modification done in `afterRender()` must be undone in `beforeDestroy()`. Memory leaks accumulate across mashup navigation.

4. **Bundling large libraries into CombinedExtensions.js**: Any `import` at the top of the runtime file is bundled into the concatenated file loaded everywhere. Large libraries must use dynamic `import()` in `afterRender()`.

5. **Non-unique DOM IDs**: Multiple instances of the same widget can coexist on a mashup. Always use `this.jqElementId` to generate unique element IDs.

6. **Missing `widget-content` CSS class**: The root element returned by `renderHtml()` must include the `widget-content` class or ThingWorx won't manage the widget correctly.

## Chrome DevTools MCP for debugging

This skill supports browser debugging via the Chrome DevTools MCP server. See `workflows/debug-widget.md` for setup and usage.
