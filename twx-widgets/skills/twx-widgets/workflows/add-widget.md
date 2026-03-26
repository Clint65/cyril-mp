# Workflow: Add a Widget to an Existing Project

Add a new widget to an existing multi-widget extension project.

## Step 1: Understand the existing project

Read the project's structure:
1. `package.json` — Get package name, dependencies, widget type (React or TS)
2. List directories in `src/` — See existing widgets
3. Read one existing widget's source files — Match the coding style and patterns
4. Check if `src/common/` exists — Vendor splitting may already be set up

## Step 2: Gather requirements

Ask the user:
1. **Widget name** — Lowercase name for the folder and files (e.g., `fcadmynewwidget`)
2. **Widget display name** — Shown in the Mashup Builder toolbar
3. **Widget purpose** — What does it do?
4. **Properties needed** — Input data (binding targets), output data (binding sources)
5. **Events** — What user interactions trigger events?
6. **Services** — What actions can be triggered on the widget?
7. **External libraries** — Any new libraries not already in the project?

## Step 3: Create widget files

Create the new widget directory `src/{{widgetname}}/` with:

### For TypeScript widgets:
- `{{widgetname}}.ide.ts` — Based on `templates/ts-widget.md`, customized with the required properties, events, services
- `{{widgetname}}.runtime.ts` — With property setters and the defensive rendering pattern
- `{{widgetname}}.html` — HTML template with unique placeholder ID
- `{{widgetname}}.ide.scss` — Composer appearance
- `{{widgetname}}.runtime.scss` — Runtime styles
- `images/icon.png` — Copy from another widget in the project or use a placeholder

### For React widgets:
- `{{widgetname}}.ide.ts` — Same as TypeScript (no React in IDE)
- `{{widgetname}}.runtime.tsx` — Using `loadVendorCore()` from `src/common/vendor-loader.ts`
- `{{widgetname}}.html` — HTML template
- `{{widgetname}}.runtime.css` — Runtime styles
- `images/icon.png`

## Step 4: Match existing conventions

Read the existing widgets and match:
- Property naming conventions (PascalCase for public properties)
- Internal variable naming (`_` prefix for private state)
- HTML class naming patterns (e.g., `widget-{{widgetname}}-main`)
- Render pattern (guard checks, container ID pattern)
- Event triggering pattern (`this.jqElement.triggerHandler()` vs decorator)

## Step 5: Add new dependencies if needed

If the widget requires new external libraries:
1. Add to `package.json` dependencies
2. If the library is large, implement lazy loading (see `references/combined-extensions.md`)
3. For React projects with shared libraries, consider adding to `vendor-core.ts`

**When adding a React widget to a TypeScript-only project:**
The project needs React-specific configuration changes:
- Add `react`, `react-dom`, `@mui/material`, `@emotion/react`, `@emotion/styled` to `dependencies`
- Add `@babel/preset-react` to `devDependencies`
- Add `"jsx": "react-jsx"` to `tsconfig.json` → `compilerOptions`
- Add `"@babel/preset-react"` to the Babel presets in `webpack/webpack.common.js`
- Create `src/common/` with `vendor-core.ts`, `vendor-loader.ts`, and optionally `theme.ts` (see `references/vendor-splitting.md`)

## Step 6: Verify build

```bash
cd {{projectPath}} && npm run testbuild
```

Check that:
- The new widget compiles without errors
- Existing widgets still compile
- Output files exist in `dist/ui/{{widgetname}}/`

Then package:
```bash
npm run buildDev
```

Check that `dist/metadata.xml` includes the new widget entry.

## Important Notes

- The webpack configuration auto-discovers widgets by scanning `src/` directories — no config changes needed
- The `widgetMetadataGeneratorPlugin.js` auto-generates metadata for all widgets
- Each widget directory is built independently — adding a widget doesn't affect others
- If the project uses vendor splitting, the new widget should import from `src/common/vendor-loader.ts` rather than importing React/MUI directly
